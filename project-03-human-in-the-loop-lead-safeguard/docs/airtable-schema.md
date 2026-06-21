# Airtable Schema

## Overview

Airtable is used as the operational database for the **AI Lead Review & Follow-Up System**.

The base is designed around four connected tables:

1. **Leads**
2. **AI Drafts**
3. **Review & Approval**
4. **Message Log**

Together, these tables support the full lifecycle of a lead:

```text
Lead captured
→ AI draft created
→ Human review decision
→ Follow-up sent, queued, revised, cancelled, or failed
→ Outcome logged
```

The schema is intentionally designed for a human-in-the-loop workflow. AI can classify leads, summarize requests, draft replies, rewrite drafts, and support rejection routing, but Airtable stores the decision state and keeps humans in control of final customer-facing actions.

---

## Table Relationship Overview

The workflow uses linked records so every lead has a clear audit trail.

```text
Leads
  ├── AI Drafts
  ├── Review & Approval
  └── Message Log

AI Drafts
  └── Linked Lead

Review & Approval
  ├── Linked Lead
  ├── Linked AI Draft
  └── Message Log

Message Log
  ├── Linked Lead
  └── Linked Review
```

This relationship model makes it possible to inspect one lead and understand:

* the original submitted request,
* the AI-generated draft,
* the human review decision,
* any revised draft versions,
* the final follow-up channel,
* whether the message was sent, queued, cancelled, or failed.

---

# 1. Leads Table

## Purpose

The **Leads** table stores the original lead submission and the current operational state of the lead.

Scenario 1 creates records in this table. Scenario 2 updates records in this table after the human review decision is processed.

The Leads table acts as the main source of truth for each lead.

---

## Leads Table Columns

| Field                        | Field Type                                   | Purpose                                                                                                                                     |
| ---------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Lead ID**                  | Autonumber / identifier                      | Human-readable lead identifier used in Slack alerts, logs, and review records.                                                              |
| **Full Name**                | Text                                         | Lead name submitted through the Tally form.                                                                                                 |
| **Email**                    | Email / text                                 | Lead email address. Used when an approved email follow-up can be sent.                                                                      |
| **Phone**                    | Phone / text                                 | Lead phone number. Used for manual phone or WhatsApp follow-up.                                                                             |
| **Company**                  | Text                                         | Lead company, organization, or business name.                                                                                               |
| **Lead Source**              | Text or single select                        | Source of the lead, such as portfolio, referral, LinkedIn, GitHub, or another source.                                                       |
| **Service Interest**         | Text or single select                        | Service the lead is interested in.                                                                                                          |
| **Budget Range**             | Text or single select                        | Budget range selected by the lead.                                                                                                          |
| **Timeline**                 | Text or single select                        | Desired project timeline selected by the lead.                                                                                              |
| **Preferred Contact Method** | Single select / form value                   | Preferred follow-up channel selected by the lead.                                                                                           |
| **Original Message**         | Long text                                    | Lead’s original message or project description.                                                                                             |
| **Lead Category**            | Single select / AI output                    | AI-generated category for the lead.                                                                                                         |
| **Priority**                 | Single select / AI output                    | AI-generated priority level.                                                                                                                |
| **Lead Status**              | Single select                                | Current operational state of the lead.                                                                                                      |
| **Notes**                    | Long text                                    | System notes, invalid submission notes, error details, or manual follow-up comments.                                                        |
| **AI Drafts 2 (System Link)**                | Linked record                                | Links the lead to one or more AI draft records.                                                                                             |
<!-- | **AI Drafts 2**              | Linked record / duplicate relationship field | Additional linked-record relationship created by Airtable/automation during table linking. Kept only if required by Airtable relationships. | -->
| **Review & Approval**        | Linked record                                | Links the lead to one or more human review records.                                                                                         |
| **Message Log**              | Linked record                                | Links the lead to follow-up outcome records.                                                                                                |

---

## Dropdown / Single Select Fields in Leads

### Preferred Contact Method

Used to decide whether Scenario 2 should send an approved email automatically or queue manual follow-up.

| Choice              | Meaning                                                                                                  |
| ------------------- | -------------------------------------------------------------------------------------------------------- |
| **Email**           | Customer prefers email. If an approved reply exists, Gmail can send the follow-up.                       |
| **WhatsApp**        | Customer prefers WhatsApp. The system queues manual follow-up instead of sending automatically.          |
| **Phone call**      | Customer prefers a phone call. The system queues manual follow-up.                                       |
| **No preference**   | The system can use email if a usable email address exists.                                               |
| **Empty / missing** | Treated as no stored preference; email may be used if available, otherwise manual follow-up is required. |

### Lead Category

Generated by AI during Scenario 1 and stored for reviewer context.

| Choice                          | Meaning                                                                          |
| ------------------------------- | -------------------------------------------------------------------------------- |
| **Sales Inquiry**               | Lead is asking about a service or potential project.                             |
| **Pricing Question**            | Lead is mainly asking about cost, budget, or pricing.                            |
| **Support Request**             | Lead appears to need support, troubleshooting, or help with an existing process. |
| **Partnership / Collaboration** | Lead is asking about collaboration, partnership, or business cooperation.        |
| **General Inquiry**             | Lead does not clearly fit another category.                                      |

### Priority

Generated by AI during Scenario 1.

| Choice     | Meaning                                                       |
| ---------- | ------------------------------------------------------------- |
| **High**   | Lead appears urgent, high-value, or time-sensitive.           |
| **Medium** | Lead is relevant but not urgent.                              |
| **Low**    | Lead is low urgency, unclear, or less immediately actionable. |

### Lead Status

Lead Status describes the operational state of the lead itself.

| Status                        | Meaning                                                                                     |
| ----------------------------- | ------------------------------------------------------------------------------------------- |
| **New**                       | Lead was created but has not yet completed the intake workflow.                             |
| **Pending Review**            | AI draft exists and the lead is waiting for human review.                                   |
| **Sent**                      | Approved email follow-up was successfully sent.                                             |
| **Manual Follow-Up Required** | A human must follow up manually, usually by WhatsApp, phone, or custom handling.            |
| **Rejected**                  | Lead was rejected and cancelled; no customer follow-up should be sent.                      |
| **Error**                     | Automation encountered invalid input, an API failure, parsing failure, or delivery failure. |

---

## Why the Leads Table Matters

The Leads table answers the main operational questions:

* Who submitted the lead?
* What service are they asking about?
* How should they be contacted?
* What did AI classify the lead as?
* What is the current workflow status?
* Did the process complete, fail, or require manual action?

---

# 2. AI Drafts Table

## Purpose

The **AI Drafts** table stores AI-generated draft replies and related AI metadata.

Drafts are stored separately from the lead record so the system can support versioning, reviewer edits, prompt tracking, and auditability.

---

## AI Drafts Table Columns

| Field                   | Field Type                | Purpose                                                         |
| ----------------------- | ------------------------- | --------------------------------------------------------------- |
| **Draft ID**            | Autonumber / identifier   | Human-readable draft identifier.                                |
| **Linked Lead**         | Linked record             | Connects the draft to the original lead.                        |
| **Full Name**           | Lookup                    | Displays the lead’s name from the linked lead.                  |
| **AI Lead Category**    | Single select / AI output | AI-generated lead category.                                     |
| **AI Priority**         | Single select / AI output | AI-generated priority level.                                    |
| **AI Summary**          | Long text                 | AI-generated summary of the lead request.                       |
| **Draft Reply**         | Long text                 | AI-generated reply prepared for human review.                   |
| **AI Confidence Notes** | Long text                 | AI notes about uncertainty, assumptions, or limitations.        |
| **Prompt Version**      | Text or single select     | Identifies which prompt version generated the draft.            |
| **Draft Version**       | Single select             | Identifies whether the draft is an original or revised version. |
| **Revision Notes**      | Long text                 | Reviewer instructions used when generating a revised draft.     |

---

## Dropdown / Single Select Fields in AI Drafts

### AI Lead Category

This mirrors the `Lead Category` stored on the Leads table.

| Choice                          | Meaning                                                                          |
| ------------------------------- | -------------------------------------------------------------------------------- |
| **Sales Inquiry**               | Lead is asking about a service or potential project.                             |
| **Pricing Question**            | Lead is mainly asking about cost, budget, or pricing.                            |
| **Support Request**             | Lead appears to need support, troubleshooting, or help with an existing process. |
| **Partnership / Collaboration** | Lead is asking about collaboration, partnership, or business cooperation.        |
| **General Inquiry**             | Lead does not clearly fit another category.                                      |

### AI Priority

This mirrors the `Priority` stored on the Leads table.

| Choice     | Meaning                                                       |
| ---------- | ------------------------------------------------------------- |
| **High**   | Lead appears urgent, high-value, or time-sensitive.           |
| **Medium** | Lead is relevant but not urgent.                              |
| **Low**    | Lead is low urgency, unclear, or less immediately actionable. |

### Draft Version

| Choice | Meaning                                                                            |
| ------ | ---------------------------------------------------------------------------------- |
| **v1** | Initial AI draft created during Scenario 1.                                        |
| **v2** | Revised AI draft created after a reviewer selects `Needs Edit` and provides notes. |

### Prompt Version

Prompt Version may be stored as text or a controlled value depending on the Airtable setup.

| Example                   | Meaning                                               |
| ------------------------- | ----------------------------------------------------- |
| **intake-v1.0**           | Initial classification and draft-generation prompt.   |
| **rewrite-v1.0**          | Rewrite prompt used when reviewer notes are provided. |
| **rejection-router-v1.0** | Prompt used to classify rejection intent.             |

---

## Linked and Lookup Fields in AI Drafts

| Field           | Relationship Type | Explanation                                                        |
| --------------- | ----------------- | ------------------------------------------------------------------ |
| **Linked Lead** | Linked record     | Connects each AI draft to the original lead.                       |
| **Full Name**   | Lookup            | Pulls the lead’s full name from the linked lead for easier review. |

---

## Why the AI Drafts Table Matters

The AI Drafts table demonstrates that AI output is not treated as a final customer message immediately.

Instead, AI output is:

* stored,
* linked to the lead,
* versioned,
* reviewed by a human,
* revised when needed,
* traced back to prompt versions and reviewer notes.

This is more robust than storing a single AI response directly inside the lead record.

---

# 3. Review & Approval Table

## Purpose

The **Review & Approval** table is the human decision layer.

This table is where the reviewer decides what should happen next:

* approve the reply,
* request edits,
* reject the lead,
* provide reviewer notes,
* update the approved reply.

Scenario 2 watches this table and processes the decision.

---

## Review & Approval Table Columns

| Field                        | Field Type               | Purpose                                                                                                               |
| ---------------------------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| **Review ID**                | Autonumber / identifier  | Human-readable review identifier.                                                                                     |
| **Linked Lead**              | Linked record            | Connects the review record to the original lead.                                                                      |
| **Linked AI Draft**          | Linked record            | Connects the review record to the current AI draft.                                                                   |
| **Approval Status**          | Single select            | Human review decision that controls Scenario 2 routing.                                                               |
| **Approved Reply**           | Long text                | Final reply text that may be sent after approval. Initially populated from the AI draft and editable by the reviewer. |
| **Reviewer Notes**           | Long text                | Reviewer instructions for edits, rejection reasons, or manual handling.                                               |
| **Reviewed By**              | Collaborator / text      | Person who reviewed the lead.                                                                                         |
| **Reviewed At**              | Date/time                | Timestamp of the review action.                                                                                       |
| **Message Log**              | Linked record            | Links the review decision to the resulting message/audit log.                                                         |
| **Lead Email**               | Lookup                   | Pulls the lead’s email from the linked lead.                                                                          |
| **Lead Company**             | Lookup                   | Pulls the company name from the linked lead.                                                                          |
| **Lead Message**             | Lookup                   | Pulls the original lead message from the linked lead.                                                                 |
| **Service Interest**         | Lookup                   | Pulls the requested service from the linked lead.                                                                     |
| **AI Priority**              | Lookup                   | Pulls AI priority from the linked AI draft.                                                                           |
| **AI Category**              | Lookup                   | Pulls AI category from the linked AI draft.                                                                           |
| **AI Summary**               | Lookup                   | Pulls AI summary from the linked AI draft.                                                                            |
| **Preferred Contact Method** | Lookup                   | Pulls preferred contact method from the linked lead.                                                                  |
| **Decision Processed**       | Checkbox / control field | Indicates whether the decision has already been processed by Scenario 2. Used to prevent duplicate processing.        |

---

## Dropdown / Single Select Fields in Review & Approval

### Approval Status

Approval Status describes the human review decision.

| Status             | Meaning                                        |
| ------------------ | ---------------------------------------------- |
| **Pending Review** | Waiting for a human reviewer.                  |
| **Approved**       | Approved reply can be processed by Scenario 2. |
| **Needs Edit**     | Reviewer wants the AI draft revised.           |
| **Rejected**       | Reviewer rejected the lead or draft.           |

---

## Linked and Lookup Fields in Review & Approval

| Field                        | Relationship Type | Explanation                                                                 |
| ---------------------------- | ----------------- | --------------------------------------------------------------------------- |
| **Linked Lead**              | Linked record     | Connects the review decision to the original lead.                          |
| **Linked AI Draft**          | Linked record     | Connects the review decision to the current AI draft version.               |
| **Message Log**              | Linked record     | Connects the review decision to the final communication outcome.            |
| **Lead Email**               | Lookup            | Used by Scenario 2 when deciding whether Gmail can send the approved reply. |
| **Lead Company**             | Lookup            | Used in reviewer context and Slack notifications.                           |
| **Lead Message**             | Lookup            | Used in reviewer context and AI rewrite/rejection prompts.                  |
| **Service Interest**         | Lookup            | Used in reviewer context and AI prompt context.                             |
| **Preferred Contact Method** | Lookup            | Used to route approved leads to email or manual follow-up.                  |
| **AI Priority**              | Lookup            | Shows reviewer the AI priority estimate.                                    |
| **AI Category**              | Lookup            | Shows reviewer the AI classification.                                       |
| **AI Summary**               | Lookup            | Shows reviewer the AI-generated summary.                                    |

---

## Why the Review & Approval Table Matters

The Review & Approval table is the core human-in-the-loop control point.

AI can prepare a reply, but the automation does not send the reply until the reviewer changes the status to `Approved`.

This table also controls the rewrite loop:

```text
Needs Edit + Reviewer Notes
→ Groq rewrite
→ AI Draft v2
→ Review reset to Pending Review
```

And it controls the rejection flow:

```text
Rejected + Reviewer Notes
→ AI interprets rejection intent
→ Cancel Completely or Manual Human Intervention
```

This keeps the reviewer in control while still allowing AI to support decision processing.

---

# 4. Message Log Table

## Purpose

The **Message Log** table records the outcome after the review decision is processed.

It is used for auditability and operational visibility.

---

## Message Log Table Columns

| Field             | Field Type              | Purpose                                                                             |
| ----------------- | ----------------------- | ----------------------------------------------------------------------------------- |
| **Log ID**        | Autonumber / identifier | Human-readable log identifier.                                                      |
| **Linked Lead**   | Linked record           | Connects the log to the original lead.                                              |
| **Linked Review** | Linked record           | Connects the log to the review decision.                                            |
| **Channel**       | Single select           | Follow-up channel or outcome channel.                                               |
| **Final Message** | Long text               | Final approved reply, manual follow-up note, cancellation note, or failure context. |
| **Sent Status**   | Single select           | Outcome of the follow-up attempt or queue.                                          |
| **Sent At**       | Date/time               | Timestamp of successful email send.                                                 |
| **Error Message** | Long text               | Failure reason, cancellation reason, or manual follow-up explanation.               |

---

## Dropdown / Single Select Fields in Message Log

### Channel

Channel describes how the follow-up was handled or intended to be handled.

| Channel      | Meaning                                                                 |
| ------------ | ----------------------------------------------------------------------- |
| **Email**    | Follow-up was sent or attempted through Gmail.                          |
| **WhatsApp** | Follow-up is queued for manual WhatsApp handling.                       |
| **Phone**    | Follow-up is queued for manual phone handling.                          |
| **Manual**   | Human follow-up is required, but no specific automated channel is used. |
| **None**     | No customer follow-up should be sent, usually after cancellation.       |

### Sent Status

Sent Status describes the outcome of the follow-up attempt or queue.

| Status        | Meaning                                       |
| ------------- | --------------------------------------------- |
| **Sent**      | Email was successfully sent.                  |
| **Queued**    | Manual follow-up is required.                 |
| **Failed**    | A delivery attempt or automation step failed. |
| **Cancelled** | Lead was rejected and no follow-up was sent.  |

---

## Linked Fields in Message Log

| Field             | Relationship Type | Explanation                                                     |
| ----------------- | ----------------- | --------------------------------------------------------------- |
| **Linked Lead**   | Linked record     | Connects the outcome log to the original lead.                  |
| **Linked Review** | Linked record     | Connects the outcome log to the review decision that caused it. |

---

## Why the Message Log Table Matters

The Message Log table makes the workflow auditable.

It records:

* successful email follow-ups,
* manual WhatsApp or phone queues,
* manual intervention requirements,
* rejected/cancelled leads,
* failed Gmail sends,
* failure reasons and operational notes.

This helps prevent silent failures and gives the business a clear record of each follow-up outcome.

---

# Dropdown Field Summary

This section summarizes all controlled-choice fields used in the Airtable base.

| Table                 | Field                        | Choices                                                                                        |
| --------------------- | ---------------------------- | ---------------------------------------------------------------------------------------------- |
| **Leads**             | **Preferred Contact Method** | Email, WhatsApp, Phone call, No preference, Empty/missing                                      |
| **Leads**             | **Lead Category**            | Sales Inquiry, Pricing Question, Support Request, Partnership / Collaboration, General Inquiry |
| **Leads**             | **Priority**                 | High, Medium, Low                                                                              |
| **Leads**             | **Lead Status**              | New, Pending Review, Sent, Manual Follow-Up Required, Rejected, Error                          |
| **AI Drafts**         | **AI Lead Category**         | Sales Inquiry, Pricing Question, Support Request, Partnership / Collaboration, General Inquiry |
| **AI Drafts**         | **AI Priority**              | High, Medium, Low                                                                              |
| **AI Drafts**         | **Draft Version**            | v1, v2                                                                                         |
| **Review & Approval** | **Approval Status**          | Pending Review, Approved, Needs Edit, Rejected                                                 |
| **Review & Approval** | **Decision Processed**       | Checkbox/control field: checked or unchecked                                                   |
| **Message Log**       | **Channel**                  | Email, WhatsApp, Phone, Manual, None                                                           |
| **Message Log**       | **Sent Status**              | Sent, Queued, Failed, Cancelled                                                                |

---

# Airtable Views

Airtable views are used for automation routing, reviewer workflow, operations monitoring, and portfolio documentation.

The views below are grouped by table.

---

## Leads Table Views

| View                             | Purpose                                                                                                                 |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **All Leads**                    | Full operational view of every lead record. Useful for debugging and complete inspection.                               |
| **New Leads**                    | Shows leads that were created but not yet moved into review, sent, rejected, or error states.                           |
| **Pending Review**               | Shows leads where an AI draft has been created and a human reviewer still needs to act.                                 |
| **Sent Leads**                   | Shows leads where approved email follow-up was successfully sent.                                                       |
| **Manual Follow-Up Required**    | Shows leads that require human action by WhatsApp, phone, or custom manual handling.                                    |
| **Rejected Leads**               | Shows leads that were rejected and cancelled.                                                                           |
| **Error Leads**                  | Shows invalid submissions, AI/API failures, JSON parsing failures, Gmail failures, or other failed automation outcomes. |
| **Portfolio Screenshot - Leads** | Simplified view used for screenshots, showing only the fields needed to understand the lead record.                     |

### Significance

The Leads views allow the business to monitor the operational pipeline:

```text
New
→ Pending Review
→ Sent / Manual Follow-Up Required / Rejected / Error
```

They also make it easy to identify leads that need attention.

---

## AI Drafts Table Views

| View                                 | Purpose                                                                                                                           |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| **All AI Drafts**                    | Full view of all generated drafts.                                                                                                |
| **Initial Drafts**                   | Shows `v1` drafts created by Scenario 1.                                                                                          |
| **Revised Drafts**                   | Shows `v2` drafts created after reviewer feedback.                                                                                |
| **High Priority Drafts**             | Shows drafts linked to high-priority leads.                                                                                       |
| **Portfolio Screenshot - AI Drafts** | Simplified screenshot view showing draft category, priority, summary, reply, confidence notes, prompt version, and draft version. |

### Significance

The AI Drafts views make it easy to review AI output quality, compare original and revised drafts, and demonstrate that the system supports versioned AI-generated content.

---

## Review & Approval Table Views

| View                                         | Purpose                                                                                                                                                               |
| -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **All Reviews**                              | Full view of every review record.                                                                                                                                     |
| **Pending Review**                           | Shows reviews waiting for a human decision.                                                                                                                           |
| **Decision Processing**                      | Used by Scenario 2 to watch actionable review decisions.                                                                                                              |
| **Approved Reviews**                         | Shows records approved by a reviewer.                                                                                                                                 |
| **Needs Edit Reviews**                       | Shows records where the reviewer requested changes.                                                                                                                   |
| **Rejected Reviews**                         | Shows records rejected by the reviewer.                                                                                                                               |
| **Processed Decisions**                      | Shows review records already processed by Scenario 2.                                                                                                                 |
| **Portfolio Screenshot - Review & Approval** | Simplified screenshot view showing the review decision, approved reply, reviewer notes, reviewer, timestamp, AI priority, preferred contact method, and lead company. |

### Decision Processing View

The `Decision Processing` view is one of the most important operational views.

Scenario 2 watches this view to process human decisions.

It should include records where:

```text
Approval Status is Approved
OR Approval Status is Needs Edit
OR Approval Status is Rejected
```

And, in a production-style setup, it should exclude records already processed by the scenario:

```text
Decision Processed is unchecked
```

This prevents Scenario 2 from repeatedly processing the same review decision.

### Significance

The Review & Approval views separate human decision states clearly:

```text
Pending Review
→ Approved / Needs Edit / Rejected
→ Processed
```

This makes the review workflow understandable for both human operators and automation reviewers.

---

## Message Log Table Views

| View                                   | Purpose                                                                                               |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **All Message Logs**                   | Full audit view of all communication outcomes.                                                        |
| **Sent Messages**                      | Shows successful email sends.                                                                         |
| **Queued Follow-Ups**                  | Shows manual WhatsApp, phone, or manual handoff tasks.                                                |
| **Failed Messages**                    | Shows failed delivery or failed automation outcomes.                                                  |
| **Cancelled Messages**                 | Shows rejected leads where no customer follow-up was sent.                                            |
| **Email Messages**                     | Shows records where the channel is Email.                                                             |
| **Manual Follow-Up Queue**             | Shows records where the outcome requires human action.                                                |
| **Portfolio Screenshot - Message Log** | Simplified screenshot view showing channel, final message, sent status, timestamp, and error message. |

### Significance

The Message Log views turn the automation into an auditable business process.

They make it easy to answer:

* Which messages were sent?
* Which follow-ups are still queued?
* Which leads were cancelled?
* Which automation paths failed?
* Which records need manual action?

---

# Screenshot-Focused Views

For GitHub and portfolio screenshots, simplified views are used to keep screenshots readable.

These views are not required for automation execution, but they make the project easier to understand visually.

## Leads Screenshot View

```text
Lead ID
Full Name
Company
Service Interest
Lead Status
Timeline
Preferred Contact Method
Original Message
Lead Category
Priority
```

## AI Drafts Screenshot View

```text
Draft ID
Full Name
AI Lead Category
AI Priority
AI Summary
Draft Reply
AI Confidence Notes
Prompt Version
Draft Version
```

## Review & Approval Screenshot View

```text
Review ID
Approval Status
Approved Reply
Reviewer Notes
Reviewed By
Reviewed At
AI Priority
Preferred Contact Method
Lead Company
```

## Message Log Screenshot View

```text
Log ID
Linked Review
Channel
Final Message
Sent Status
Sent At
Error Message
```

---

# Linked Records and Lookups

The system uses linked records to model relationships and lookup fields to expose related data where the automation needs it.

## Linked Record Fields

| Table                 | Field                 | Links To          | Purpose                                                |
| --------------------- | --------------------- | ----------------- | ------------------------------------------------------ |
| **Leads**             | **AI Drafts**         | AI Drafts         | Shows all drafts created for the lead.                 |
| **Leads**             | **Review & Approval** | Review & Approval | Shows review records connected to the lead.            |
| **Leads**             | **Message Log**       | Message Log       | Shows follow-up outcome records connected to the lead. |
| **AI Drafts**         | **Linked Lead**       | Leads             | Connects each draft to its source lead.                |
| **Review & Approval** | **Linked Lead**       | Leads             | Connects each review to the source lead.               |
| **Review & Approval** | **Linked AI Draft**   | AI Drafts         | Connects each review to the current draft version.     |
| **Review & Approval** | **Message Log**       | Message Log       | Connects the review decision to final outcome logs.    |
| **Message Log**       | **Linked Lead**       | Leads             | Connects each log to the original lead.                |
| **Message Log**       | **Linked Review**     | Review & Approval | Connects each log to the review decision.              |

## Lookup Fields

| Table                 | Field                        | Source    | Purpose                                                        |
| --------------------- | ---------------------------- | --------- | -------------------------------------------------------------- |
| **AI Drafts**         | **Full Name**                | Leads     | Displays the lead name for easier draft review.                |
| **Review & Approval** | **Lead Email**               | Leads     | Used to decide whether Gmail can send the approved reply.      |
| **Review & Approval** | **Lead Company**             | Leads     | Used for reviewer context and Slack notifications.             |
| **Review & Approval** | **Lead Message**             | Leads     | Used in reviewer context and AI prompts.                       |
| **Review & Approval** | **Service Interest**         | Leads     | Used in reviewer context and AI prompts.                       |
| **Review & Approval** | **Preferred Contact Method** | Leads     | Used to route approved decisions to email or manual follow-up. |
| **Review & Approval** | **AI Category**              | AI Drafts | Shows reviewer the AI classification.                          |
| **Review & Approval** | **AI Priority**              | AI Drafts | Shows reviewer the AI priority estimate.                       |
| **Review & Approval** | **AI Summary**               | AI Drafts | Shows reviewer the AI-generated summary.                       |

In Make, Airtable lookup fields are often returned as arrays. The workflow uses the first value from these arrays when mapping lookup data into AI prompts, Slack messages, Gmail messages, and Airtable updates.

---

# Key Design Decisions

## Why separate AI Drafts from Leads?

AI drafts are stored separately so the system can support versioning and auditability.

This allows the workflow to preserve:

* the original lead message,
* the initial AI draft,
* revised AI drafts,
* reviewer notes,
* prompt versions,
* confidence notes.

This is cleaner and more traceable than storing every AI output directly in the lead record.

## Why use Review & Approval as a separate table?

The Review & Approval table creates a clear approval boundary.

It separates:

* what the lead submitted,
* what AI suggested,
* what the human approved or changed,
* what the automation finally sent, queued, cancelled, or logged.

This supports safer automation and easier debugging.

## Why use Message Log?

The Message Log table records business outcomes.

Without it, the business would only know the current lead status. With it, the system can show what happened, when it happened, through which channel, and whether it succeeded, failed, was queued, or was cancelled.

## Why queue WhatsApp and phone instead of sending automatically?

WhatsApp and phone follow-ups are handled manually in this version to keep the system safe and realistic.

The automation prepares the approved message or call note, updates Airtable, creates a message log, and alerts the team in Slack. A human then performs the actual follow-up.

## Why keep invalid submissions in Airtable?

Invalid submissions are still stored instead of being ignored.

This helps the business inspect incomplete form submissions, diagnose form issues, and avoid losing potentially useful leads. These records are marked with Lead Status `Error` so they do not continue through AI drafting.

---

# Summary

The Airtable schema supports a complete human-in-the-loop lead workflow:

```text
Lead captured
→ AI draft created
→ Human review decision
→ Follow-up sent, queued, revised, cancelled, or failed
→ Outcome logged
```

The schema is designed to be understandable, auditable, and safe for business use. It demonstrates practical data modeling for AI-assisted automation rather than a simple one-step form-to-email workflow.
