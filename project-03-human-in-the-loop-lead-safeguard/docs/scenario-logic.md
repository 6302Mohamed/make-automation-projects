# Scenario Logic

## Overview

The AI Lead Review & Follow-Up System is implemented with two Make.com scenarios:

1. **Scenario 1 — Lead Intake + AI Draft Generation**
2. **Scenario 2 — Human Review + Follow-Up Processing**

Scenario 1 prepares the lead for human review.
Scenario 2 processes the reviewer’s decision and completes the follow-up workflow.

The two scenarios are connected asynchronously via an event-driven pattern through Airtable. Scenario 1 acts as the intake pipeline, staging the relational records. Scenario 2 acts as the processing engine, watching a filtered Airtable view and triggering immediately when a human state-change occurs

---

# Scenario 1 — Lead Intake + AI Draft Generation

## Purpose

Scenario 1 captures a new lead submission, validates the input, stores the lead in Airtable, asks Groq to classify and draft a reply, then prepares the record for human review.

This scenario does not send any customer-facing message. Its job is to prepare a structured AI draft and hand it to a human reviewer.

---

## Trigger

Scenario 1 starts when a new lead is submitted through the Tally form.

The form captures lead information such as:

* full name,
* email,
* phone,
* company,
* service interest,
* budget range,
* timeline,
* preferred contact method,
* original message.

---

## Main Flow

```text id="18ffvp"
Tally form submission
→ Validate required fields
→ Create Lead record in Airtable
→ Create JSON payload
→ Send HTTP request to Groq
→ Parse JSON response
→ Create AI Draft record
→ Create Review & Approval record
→ Update Lead status to Pending Review
→ Send Slack notification to reviewer
```

---

## Step-by-Step Logic

## 1. Receive Lead Submission

The scenario receives lead data from Tally.

This creates the raw input for the rest of the workflow.

---

## 2. Validate Required Fields

The workflow checks whether the required form fields are present.

Required fields include the key information needed to create a usable lead record and generate an AI draft.

Examples:

* full name,
* contact information,
* service interest,
* original message.

### Valid Submission

If the submission is valid, the workflow continues to AI drafting.

### Invalid Submission

If the submission is incomplete or invalid:

```text id="x43tko"
Create Lead record
→ Set Lead Status = Error
→ Store invalid submission note
→ Stop before AI drafting
```

Invalid submissions are still saved in Airtable instead of being ignored. This gives the business visibility into incomplete form submissions and prevents silent data loss.

---

## 3. Create Lead Record

For valid submissions, Make creates a record in the **Leads** table.

The new lead record stores:

* submitted contact details,
* service interest,
* preferred contact method,
* original message,
* initial status,
* notes if needed.

At this stage, the lead is captured but not yet reviewed.

---

## 4. Create JSON Payload

Before calling Groq, Make prepares a structured JSON payload.

This payload includes the lead information needed by the AI model to:

* classify the lead,
* estimate priority,
* summarize the request,
* generate a professional draft reply,
* provide confidence notes.

Using a JSON payload makes the AI request more structured and easier to parse later.

---

## 5. Send HTTP Request to Groq

Make sends the structured request to Groq through an HTTP module.

Groq is asked to return a valid JSON response containing fields such as:

```text id="um72cl"
lead_category
priority
summary
draft_reply
confidence_notes
```

The prompt instructs the AI to avoid unsafe promises, avoid claiming the project is accepted, and produce a reply that still requires human review.

---

## 6. Parse JSON Response

Make parses the Groq response so each field can be mapped into Airtable.

The parsed AI fields are used to populate:

* the **AI Drafts** table,
* the **Review & Approval** table,
* lead category and priority fields in the **Leads** table.

If the AI response cannot be parsed, the workflow routes into an error handler.

---

## 7. Create AI Draft Record

Make creates a new record in the **AI Drafts** table.

The draft record stores:

* linked lead,
* AI lead category,
* AI priority,
* AI summary,
* draft reply,
* AI confidence notes,
* prompt version,
* draft version.

The first draft generated from the intake scenario is stored as `v1`.

---

## 8. Create Review & Approval Record

Make creates a new record in the **Review & Approval** table.

This record becomes the human review task.

It links to:

* the original lead,
* the AI draft,
* the proposed reply.

The initial approval status is set to:

```text id="wwh4og"
Pending Review
```

The reviewer can then approve, request edits, or reject the draft.

---

## 9. Update Lead Status

After the draft and review records are created, Make updates the original lead.

The lead is updated with:

* AI-generated category,
* AI-generated priority,
* Lead Status = `Pending Review`.

This makes the lead visible in the review queue.

---

## 10. Send Slack Notification

Make sends a Slack notification to alert the reviewer that a new lead is ready for review.

The Slack message includes useful context such as:

* lead name,
* company,
* category,
* priority,
* review record reference,
* summary or next action.

---

## Scenario 1 Error Handling

Scenario 1 includes error handling for the main failure points.

| Failure Point                  | Handling                                                     |
| ------------------------------ | ------------------------------------------------------------ |
| **Invalid form submission**    | Lead is saved with status `Error`; AI drafting is skipped.   |
| **Groq API failure**           | Lead is updated to `Error`; internal/admin alert is sent.    |
| **JSON parsing failure**       | Lead is updated to `Error`; internal/admin alert is sent.    |
| **Slack notification failure** | Lead remains `Pending Review`; fallback admin alert is sent. |

Scenario 1 is designed so that failed or invalid records are visible instead of silently disappearing.

---

# Scenario 2 — Human Review + Follow-Up Processing

## Purpose

Scenario 2 processes the human reviewer’s decision.

It watches the **Review & Approval** table and routes the workflow based on the selected approval status.

Scenario 2 is responsible for:

* sending approved email replies,
* queueing manual follow-ups,
* rewriting drafts when edits are requested,
* processing rejected leads,
* updating lead statuses,
* creating message logs,
* notifying the team in Slack.

---

## Trigger

Scenario 2 watches the **Decision Processing** view in the **Review & Approval** table.

A record becomes actionable when the reviewer changes `Approval Status` to one of:

```text id="8b1hj4"
Approved
Needs Edit
Rejected
```

In a production-style setup, a `Decision Processed` checkbox or similar control field can be used to prevent the same decision from being processed multiple times.

---

## Main Routing Logic

```text id="vxg3uu"
Watch Review & Approval record
→ Check Approval Status
→ Route to Approved, Needs Edit, or Rejected path
```

Each branch has different behavior.

---

# Approved Path

## Purpose

The approved path handles records where a human reviewer has approved the final reply.

The system then decides whether the reply can be sent automatically by email or should be queued for manual follow-up.

---

## Approved Path Flow

```text id="hm4t78"
Approval Status = Approved
→ Check preferred contact method and usable email
→ Send Gmail OR queue manual follow-up
→ Update Lead status
→ Create Message Log
→ Send Slack notification
```

---

## Email Send Logic

If the lead has a usable email and the preferred contact method allows email follow-up, the system sends the approved reply through Gmail.

This applies when:

```text id="b44w8b"
Preferred Contact Method = Email
OR Preferred Contact Method = No preference and email exists
OR Preferred Contact Method is empty/missing and email exists
```

After sending the email:

```text id="2kwxp0"
Update Lead Status = Sent
Create Message Log:
  Channel = Email
  Sent Status = Sent
  Final Message = Approved Reply
  Sent At = timestamp
Send Slack delivery confirmation
```

---

## Manual Follow-Up Logic

If the lead should not receive an automated email, the system queues manual follow-up.

This applies when:

```text id="4e01oi"
Preferred Contact Method = WhatsApp
OR Preferred Contact Method = Phone call
OR no usable email exists
```

After queueing manual follow-up:

```text id="rss0mi"
Update Lead Status = Manual Follow-Up Required
Create Message Log:
  Channel = WhatsApp / Phone / Manual
  Sent Status = Queued
  Final Message = Approved Reply or follow-up note
Send Slack manual action alert
```

This protects the business from sending messages through channels that still require human handling.

---

# Needs Edit Path

## Purpose

The needs-edit path handles cases where the reviewer wants the AI draft revised.

The reviewer must provide notes so the system can generate a useful revised draft.

---

## Needs Edit Flow

```text id="cxsr8i"
Approval Status = Needs Edit
→ Check Reviewer Notes
→ If notes missing: ask reviewer to add notes
→ If notes provided: generate revised draft with Groq
→ Save AI Draft v2
→ Reset review to Pending Review
→ Notify reviewer
```

---

## Reviewer Notes Missing

If the reviewer selects `Needs Edit` but does not provide notes:

```text id="go905w"
Notify reviewer to add notes
Update / keep Review Status = Pending Review
Update Lead Status = Pending Review
Do not create new AI draft
Do not create Message Log
```

No customer-facing action happens in this path.

---

## Reviewer Notes Provided

If reviewer notes are present:

```text id="rcc4rm"
Create JSON payload
→ Send HTTP request to Groq
→ Parse revised draft JSON
→ Create AI Draft v2
→ Update Review & Approval record
→ Reset Approval Status to Pending Review
→ Send Slack notification
```

The revised draft is stored as `v2` in the **AI Drafts** table.

The **Review & Approval** record is updated with the revised approved reply and linked to the new draft.

The reviewer must review the revised version again before anything is sent.

---

## Why the Needs Edit Path Matters

This path demonstrates a controlled AI revision loop.

The AI does not decide what to change by itself. It rewrites the draft only after the reviewer gives explicit notes.

---

# Rejected Path

## Purpose

The rejected path handles cases where the reviewer rejects the lead or draft.

Instead of treating every rejection the same way, the workflow uses AI to classify the reviewer’s rejection intent.

This makes the rejected path more flexible and realistic.

---

## Rejected Path Flow

```text id="juq6uv"
Approval Status = Rejected
→ Create JSON payload
→ Send reviewer notes and lead context to Groq
→ Parse rejection-intent JSON
→ Route to Cancel Completely or Manual Human Intervention
```

---

## Rejection Intent Classification

When a reviewer rejects a record, the reviewer notes may mean different things.

For example:

```text id="8wpxe6"
"Do not follow up. Not a fit."
```

This should likely become:

```text id="psbmyc"
Cancel Completely
```

But:

```text id="5l2vsb"
"Needs a custom call. Too sensitive for email."
```

This should become:

```text id="axdymw"
Manual Human Intervention
```

The system uses Groq to classify the rejection intent into one of two outcomes.

---

## Rejection Outcomes

## 1. Cancel Completely

If the reviewer clearly indicates that no follow-up should be sent:

```text id="kooa6z"
Update Lead Status = Rejected
Create Message Log:
  Channel = None
  Sent Status = Cancelled
  Final Message = cancellation note
Send Slack notification:
  no customer follow-up sent
```

This is used when the lead should be closed without customer communication.

---

## 2. Manual Human Intervention

If the reviewer indicates that the case needs custom handling:

```text id="vrez0u"
Update Lead Status = Manual Follow-Up Required
Create Message Log:
  Channel = Manual
  Sent Status = Queued
  Final Message = reviewer/manual handoff note
Send Slack notification:
  human intervention required
```

This is used when the lead should not be handled automatically, but should not be fully cancelled either.

---

## Why the Rejected Path Matters

The rejected path demonstrates AI-assisted routing without giving AI final authority.

AI interprets reviewer notes, but the reviewer has already made the rejection decision. The AI only helps decide the operational follow-up path:

```text id="1dl69i"
Cancel completely
OR
manual human intervention
```

This keeps the workflow safe while still using AI to reduce manual routing effort.

---

# Scenario 2 Error Handling

Scenario 2 includes error handling for important operational failures.

| Failure Point                      | Handling                                                                                    |
| ---------------------------------- | ------------------------------------------------------------------------------------------- |
| **Missing reviewer notes**         | Reviewer is asked to add notes; review returns to pending state.                            |
| **Groq rewrite failure**           | Lead is marked `Error`; admin alert is sent.                                                |
| **Groq rejection-routing failure** | Lead is marked `Error`; admin alert is sent.                                                |
| **JSON parsing failure**           | Lead is marked `Error`; admin alert is sent.                                                |
| **Gmail send failure**             | Message Log is created or updated as `Failed`; Lead is marked `Error`; admin alert is sent. |
| **Slack notification failure**     | Fallback admin notification is sent where applicable.                                       |

Not every error branch is shown in the main Scenario 2 diagram because the scenario is longer and would become difficult to read. Full error behavior is documented separately in `error-handling.md`.

---

# Data Updates by Path

## Scenario 1 Data Updates

| Path                   | Leads                                         | AI Drafts                 | Review & Approval              | Message Log        |
| ---------------------- | --------------------------------------------- | ------------------------- | ------------------------------ | ------------------ |
| **Valid lead**         | Create lead, later update to `Pending Review` | Create `v1` draft         | Create `Pending Review` record | No message log yet |
| **Invalid lead**       | Create lead with `Error` status               | Not created               | Not created                    | Not created        |
| **Groq/parse failure** | Update lead to `Error`                        | Not created or incomplete | Not created or incomplete      | Not created        |
| **Slack failure**      | Lead remains `Pending Review`                 | Created                   | Created                        | Not created        |

---

## Scenario 2 Data Updates

| Path                               | Leads                                 | AI Drafts                  | Review & Approval                          | Message Log                                          |
| ---------------------------------- | ------------------------------------- | -------------------------- | ------------------------------------------ | ---------------------------------------------------- |
| **Approved + email**               | Update to `Sent`                      | No new draft               | Process approved decision                  | Create `Email / Sent` log                            |
| **Approved + manual follow-up**    | Update to `Manual Follow-Up Required` | No new draft               | Process approved decision                  | Create `WhatsApp`, `Phone`, or `Manual / Queued` log |
| **Needs Edit + notes missing**     | Keep or update to `Pending Review`    | No new draft               | Reset or keep `Pending Review`             | No message log                                       |
| **Needs Edit + notes provided**    | Keep or update to `Pending Review`    | Create `v2` draft          | Update reply and reset to `Pending Review` | No message log                                       |
| **Rejected + cancel**              | Update to `Rejected`                  | No new draft               | Process rejected decision                  | Create `None / Cancelled` log                        |
| **Rejected + manual intervention** | Update to `Manual Follow-Up Required` | No new draft               | Process rejected decision                  | Create `Manual / Queued` log                         |
| **Gmail failure**                  | Update to `Error`                     | No new draft               | Decision remains reviewable                | Create or update `Email / Failed` log                |
| **AI/API/parse failure**           | Update to `Error`                     | No new draft or incomplete | Decision remains reviewable                | No customer message sent                             |

---

# Human-in-the-Loop Control Points

The workflow has several control points where automation waits for or depends on human judgment.

| Control Point                        | Why It Matters                                                          |
| ------------------------------------ | ----------------------------------------------------------------------- |
| **Review & Approval status**         | Prevents AI drafts from being sent without human approval.              |
| **Approved Reply field**             | Allows the reviewer to edit the final reply before sending.             |
| **Reviewer Notes field**             | Guides AI rewrite and rejection-routing logic.                          |
| **Manual Follow-Up Required status** | Keeps phone, WhatsApp, unclear, or sensitive cases under human control. |
| **Message Log**                      | Creates an audit trail of what happened after review.                   |

---

# Summary

The two-scenario design separates intake and decision processing:

```text id="z3wbqn"
Scenario 1:
Capture lead → validate → generate AI draft → prepare human review

Scenario 2:
Process human decision → send, revise, cancel, queue, or log outcome
```

This separation makes the workflow easier to debug, safer to operate, and clearer to explain.

The system demonstrates practical automation architecture using:

* Make.com scenario orchestration,
* Airtable relational data modeling,
* Groq API integration,
* structured JSON prompting and parsing,
* human-in-the-loop approval,
* Gmail delivery,
* Slack notifications,
* operational error handling,
* audit logging.

