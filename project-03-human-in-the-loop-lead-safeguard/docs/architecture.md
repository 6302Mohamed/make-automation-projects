
# System Architecture

## Overview

The **AI Lead Review & Follow-Up System** is a human-in-the-loop automation workflow designed to help a business capture incoming leads, generate AI-assisted draft replies, route those drafts for human review, and process the final follow-up decision safely.

The system is built around two Make.com scenarios:

1. **Scenario 1 — Lead Intake + AI Draft Generation**

   * Captures a new lead from a Tally form.
   * Validates required form data.
   * Stores the lead in Airtable.
   * Sends structured lead context to Groq for classification and draft generation.
   * Stores the AI draft in Airtable.
   * Creates a review record for human approval.
   * Notifies the reviewer in Slack.

2. **Scenario 2 — Human Review + Follow-Up Processing**

   * Watches Airtable review decisions.
   * Handles approved replies, requested edits, and rejected leads.
   * Sends approved email replies only after human approval.
   * Queues manual follow-up for WhatsApp, phone calls, or missing contact data.
   * Generates revised AI drafts when reviewer notes are provided.
   * Routes rejected leads into cancellation or manual intervention.
   * Updates lead statuses and writes message log records for auditability.

The architecture is intentionally designed so that AI assists with classification and drafting, but does not independently send customer-facing messages without human approval.

---

## Architecture Goals

The system was designed with the following goals:

* **Human control:** AI-generated replies must be reviewed before any customer follow-up is sent.
* **Traceability:** Lead status changes, draft versions, follow-up outcomes, and failed sends are recorded in Airtable.
* **Safe automation:** WhatsApp and phone follow-ups are queued for manual handling rather than sent automatically.
* **Structured AI output:** AI responses are requested and parsed as JSON so downstream automation can reliably use category, priority, summary, and draft fields.
* **Error visibility:** API, JSON parsing, Gmail, and Slack failures are routed into explicit error or fallback paths.
* **Portfolio-ready implementation:** The project demonstrates practical Make.com orchestration, Airtable data modeling, AI API integration, human approval workflows, and production-style error handling.

---

## High-Level System Components

| Component    | Role                                                                                                                 |
| ------------ | -------------------------------------------------------------------------------------------------------------------- |
| **Tally**    | Captures lead submissions through a public lead intake form.                                                         |
| **Make.com** | Orchestrates validation, AI calls, routing logic, record creation, status updates, and notifications.                |
| **Airtable** | Serves as the operational database for leads, AI drafts, review decisions, and message logs.                         |
| **Groq API** | Provides LLM-powered classification, summarization, draft generation, rewrite support, and rejection-intent routing. |
| **Gmail**    | Sends approved email follow-ups after human approval.                                                                |
| **Slack**    | Notifies reviewers and admins about review tasks, manual follow-ups, cancellations, and failures.                    |

---

## Scenario 1: Lead Intake + AI Draft Generation

Scenario 1 is responsible for converting a raw lead submission into a structured review-ready record.

![Scenario 1 Lead Intake and AI Draft Diagram](scenario-1-lead-intake-ai-draft-diagram.png)

### Main Flow

1. A lead submits a form through Tally.
2. Make checks whether the submission contains the required information.
3. Valid leads are stored in the Airtable `Leads` table.
4. Make builds a structured JSON request for the LLM.
5. Groq classifies the lead and generates a draft reply.
6. Make parses the AI response as JSON.
7. The AI output is stored in the `AI Drafts` table.
8. A related record is created in the `Review & Approval` table.
9. Slack notifies the reviewer that a new lead is ready.
10. The original lead is updated to `Pending Review` with category and priority metadata.

### Invalid Submission Path

If required lead data is missing or incomplete:

* The lead is still stored in Airtable.
* The lead status is set to `Error`.
* The invalid submission is logged for later inspection.
* No AI drafting is attempted.

This preserves visibility into failed or incomplete submissions without allowing bad input to continue through the AI workflow.

### Scenario 1 Error Handling

Scenario 1 includes specific error-handling paths for:

* Groq API request failure.
* JSON parsing failure.
* Slack notification failure.

When AI or parsing fails, the lead is marked as `Error` and an admin alert is created. When Slack notification fails after the Airtable records are already created, the lead remains in `Pending Review` and a fallback admin notification is sent.

---

## Scenario 2: Human Review + Follow-Up Processing

Scenario 2 begins after a reviewer makes a decision in Airtable.

![Scenario 2 Human Review and Follow-Up Diagram](scenario-2-human-review-followup-diagram.png)

### Main Decision Paths

Scenario 2 handles three main review outcomes:

1. **Approved**
2. **Needs Edit**
3. **Rejected**

Each path updates Airtable and creates an appropriate audit trail.

---

## Approved Path

When a review is approved, the workflow checks the lead’s preferred contact method and available contact data.

### Email or No Preference with Email Available

If the lead selected email, or selected no preference and an email address exists:

* Gmail sends the approved reply.
* The lead status is updated to `Sent`.
* A `Message Log` record is created with status `Sent`.
* Slack confirms the delivery.

### WhatsApp, Phone Call, or No Usable Email

If the lead selected WhatsApp, selected phone call, or no usable email is available:

* No customer message is sent automatically.
* The lead is updated to `Manual Follow-Up Required`.
* A `Message Log` record is created with status `Queued`.
* Slack alerts the team that manual follow-up is required.

This keeps customer-facing communication safe while still ensuring the follow-up is tracked.

---

## Needs Edit Path

When the reviewer selects `Needs Edit`, the workflow checks whether reviewer notes are present.

### Reviewer Notes Missing

If the reviewer did not provide notes:

* A notification is sent asking the reviewer to add revision instructions.
* The review remains or returns to `Pending Review`.
* No revised AI draft is generated.

### Reviewer Notes Provided

If reviewer notes are available:

* Make creates a JSON revision request.
* Groq rewrites the draft using the reviewer’s notes.
* Make parses the revised draft response.
* A new `AI Drafts` record is created with draft version `v2`.
* The `Review & Approval` record is updated with the revised reply.
* The review status is reset to `Pending Review`.
* Slack notifies the reviewer that the revised draft is ready.

This creates a controlled AI rewrite loop where the reviewer remains in charge.

---

## Rejected Path

The rejected path includes AI-assisted routing to distinguish between leads that should be cancelled completely and leads that require manual human intervention.

When a review is rejected:

1. Make creates a JSON request containing the reviewer notes and lead context.
2. Groq classifies the reviewer’s rejection intent.
3. Make parses the rejection-intent JSON.
4. The workflow routes the lead into one of two outcomes.

### Cancel Completely

If the reviewer clearly indicates that no follow-up is needed:

* The lead is updated to `Rejected`.
* A `Message Log` record is created with status `Cancelled`.
* Slack confirms that no customer message was sent.

### Manual Human Intervention

If the reviewer notes suggest a sensitive case, unclear decision, custom handling, phone call, WhatsApp follow-up, or manager review:

* The lead is updated to `Manual Follow-Up Required`.
* A `Message Log` record is created with status `Queued`.
* Slack alerts the team that human intervention is required.

This path demonstrates safe AI-assisted decision support without allowing the AI to override the human reviewer.

---

## Data Storage Model

The Airtable base contains four core tables:

1. **Leads**

   * Stores lead details, status, category, priority, contact preference, and notes.

2. **AI Drafts**

   * Stores AI-generated draft replies, summaries, confidence notes, prompt versions, and draft versions.

3. **Review & Approval**

   * Stores human review decisions, approved replies, reviewer notes, review status, and linked lead/draft records.

4. **Message Log**

   * Stores final follow-up messages, channels, sent/queued/cancelled/failed status, timestamps, and error details.

The database is structured to support traceability across the complete workflow: lead intake, AI drafting, human review, follow-up delivery, and audit logging.

---

## Safety and Human-in-the-Loop Design

This system follows a human-in-the-loop design pattern:

* AI can classify leads.
* AI can summarize lead requests.
* AI can draft replies.
* AI can rewrite drafts based on reviewer notes.
* AI can help interpret rejection notes.
* Humans decide whether a reply is approved, edited, rejected, sent, cancelled, or handled manually.

The automation does not blindly send AI-generated customer messages. Customer-facing email delivery only happens after a human-approved reply exists in Airtable.

---

## Error Handling Summary

The workflow includes error-handling logic for important failure points:

* Invalid or incomplete lead submissions.
* Groq API request failures.
* JSON parsing failures.
* Gmail send failures.
* Slack notification failures.
* Missing reviewer notes.
* Rejection-routing failures.

Full details are documented in [`error-handling.md`](error-handling.md).

---

## Blueprint Files

Sanitized Make blueprint files are included in the repository:

* [`../blueprints/scenario-1-lead-intake-ai-draft-sanitized.json`](../blueprints/scenario-1-lead-intake-ai-draft-sanitized.json)
* [`../blueprints/scenario-2-human-review-followup-sanitized.json`](../blueprints/scenario-2-human-review-followup-sanitized.json)

These blueprints are provided for architecture review and portfolio evidence. Sensitive connection IDs, API references, Airtable IDs, Slack channels, and email addresses have been redacted or replaced with placeholders.

Because they are sanitized, importing them into another Make account would require reconnecting services and remapping account-specific fields.
