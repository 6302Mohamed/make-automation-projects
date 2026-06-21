# Error Handling

## Overview

The **AI Lead Review & Follow-Up System** includes error handling to make failed automation steps visible, traceable, and recoverable.

This document explains how the system handles invalid input, AI/API failures, JSON parsing failures, missing reviewer notes, Gmail delivery failures, Slack notification failures, and manual fallback cases.

The goal is not only to automate the happy path, but to make the workflow safer for real business use.

---

## Error Handling Goals

The workflow is designed around the following safety goals:

* **Do not silently lose failed records.**
* **Do not send unapproved AI-generated messages.**
* **Do not continue automation when required data is missing.**
* **Do not treat failed sends as successful sends.**
* **Keep failed or manual cases visible in Airtable.**
* **Notify the team when human attention is required.**
* **Use Message Log records to preserve the outcome of follow-up attempts.**

In short, failure paths should create visibility instead of confusion.

---

## Error Handling Strategy

The system uses four main recovery patterns.

| Pattern                             | Purpose                                                                              |
| ----------------------------------- | ------------------------------------------------------------------------------------ |
| **Mark the Lead as Error**          | Used when the automation cannot safely continue.                                     |
| **Keep the Lead in Pending Review** | Used when the review task still exists but notification or revision needs attention. |
| **Queue Manual Follow-Up**          | Used when automation should not send the message automatically.                      |
| **Create or Update Message Log**    | Used to record sent, failed, queued, or cancelled outcomes.                          |

These patterns help keep the workflow understandable and auditable.

---

# Scenario 1 Error Handling

Scenario 1 handles lead intake, validation, AI drafting, review record creation, and reviewer notification.

The main failure points are:

1. Invalid form submission
2. Groq API failure
3. JSON parsing failure
4. Slack notification failure

---

## 1. Invalid Form Submission

## Failure Point

A lead submits the form, but required information is missing or incomplete.

Examples:

* missing name,
* missing contact information,
* missing service interest,
* missing original message,
* incomplete required form fields.

## Why This Matters

The system should not send incomplete data to the AI model. Bad input can create weak summaries, poor classification, or unusable draft replies.

At the same time, the submission should not disappear. A partially completed lead may still be useful.

## Handling

```text id="jptmwe"
Invalid Tally submission
→ Create Lead record
→ Set Lead Status = Error
→ Add note explaining missing/incomplete submission
→ Stop before AI drafting
```

## Airtable Outcome

| Table                 | Outcome                                     |
| --------------------- | ------------------------------------------- |
| **Leads**             | Record is created with Lead Status `Error`. |
| **AI Drafts**         | No draft is created.                        |
| **Review & Approval** | No review record is created.                |
| **Message Log**       | No message log is created.                  |

## Business Result

The invalid lead is still visible in Airtable for inspection, but it does not continue into AI drafting or review.

---

## 2. Groq API Failure During Intake

## Failure Point

The scenario successfully creates a lead record, but the Groq API request fails.

Possible causes:

* API timeout,
* authentication or connection issue,
* rate limit,
* service unavailable,
* malformed request payload.

## Why This Matters

If the AI call fails, the system cannot safely create an AI draft or review record. Continuing would create incomplete downstream data.

## Handling

```text id="1d4lz3"
Groq API request fails
→ Update Lead record
→ Set Lead Status = Error
→ Add API failure note
→ Send internal/admin alert
→ Stop downstream draft/review creation
```

## Airtable Outcome

| Table                 | Outcome                                          |
| --------------------- | ------------------------------------------------ |
| **Leads**             | Existing lead is updated to Lead Status `Error`. |
| **AI Drafts**         | No AI draft is created.                          |
| **Review & Approval** | No review record is created.                     |
| **Message Log**       | No customer message log is created.              |

## Business Result

The team can see that the lead exists, but AI drafting failed and requires attention.

---

## 3. JSON Parsing Failure During Intake

## Failure Point

Groq returns a response, but Make cannot parse the response as valid JSON.

Possible causes:

* malformed AI response,
* missing expected fields,
* extra non-JSON text,
* invalid structure,
* unexpected field names.

## Why This Matters

Scenario 1 depends on structured AI fields such as:

```text id="b4y9fz"
lead_category
priority
summary
draft_reply
confidence_notes
```

If parsing fails, the automation cannot reliably map the AI output into Airtable.

## Handling

```text id="4fpb0z"
Groq response received
→ JSON parsing fails
→ Update Lead record
→ Set Lead Status = Error
→ Add parsing failure note
→ Send internal/admin alert
→ Stop draft/review creation
```

## Airtable Outcome

| Table                 | Outcome                                          |
| --------------------- | ------------------------------------------------ |
| **Leads**             | Existing lead is updated to Lead Status `Error`. |
| **AI Drafts**         | No reliable AI draft is created.                 |
| **Review & Approval** | No review record is created.                     |
| **Message Log**       | No customer message log is created.              |

## Business Result

The system avoids storing unreliable AI output as if it were valid.

---

## 4. Slack Notification Failure During Intake

## Failure Point

The lead, AI draft, and review record are created successfully, but the Slack notification fails.

Possible causes:

* Slack connection issue,
* missing channel permission,
* unavailable workspace,
* invalid Slack module configuration.

## Why This Matters

At this point, the review task already exists in Airtable. The failure is not in the core business record; it is in the reviewer notification.

The system should not mark the lead as failed if the review record exists and is still actionable.

## Handling

```text id="2g1aoe"
Slack notification fails
→ Keep Lead Status = Pending Review
→ Keep AI Draft record
→ Keep Review & Approval record
→ Send fallback admin alert where available
```

## Airtable Outcome

| Table                 | Outcome                                                         |
| --------------------- | --------------------------------------------------------------- |
| **Leads**             | Lead remains `Pending Review`.                                  |
| **AI Drafts**         | Draft remains available.                                        |
| **Review & Approval** | Review record remains available.                                |
| **Message Log**       | No message log is created because no customer message was sent. |

## Business Result

The reviewer can still find the task in Airtable even if the Slack notification fails.

---

# Scenario 2 Error Handling

Scenario 2 handles human review decisions and follow-up processing.

The main failure points are:

1. Missing reviewer notes
2. Groq rewrite failure
3. Groq rejection-routing failure
4. JSON parsing failure
5. Gmail send failure
6. Slack notification failure

---

## 1. Missing Reviewer Notes

## Failure Point

The reviewer selects `Needs Edit`, but does not provide revision notes.

## Why This Matters

The AI rewrite step needs clear instructions. Without reviewer notes, the AI would have to guess what to change, which could produce a worse or irrelevant revision.

## Handling

```text id="8dhegv"
Approval Status = Needs Edit
→ Reviewer Notes is empty
→ Notify reviewer to add notes
→ Return or keep review in Pending Review
→ Keep Lead Status = Pending Review
→ Do not call Groq
→ Do not create AI Draft v2
```

## Airtable Outcome

| Table                 | Outcome                                        |
| --------------------- | ---------------------------------------------- |
| **Leads**             | Lead remains or returns to `Pending Review`.   |
| **AI Drafts**         | No new draft is created.                       |
| **Review & Approval** | Review remains or returns to `Pending Review`. |
| **Message Log**       | No message log is created.                     |

## Business Result

The system prevents unnecessary AI calls and keeps the reviewer in control.

---

## 2. Groq Failure During Needs Edit Rewrite

## Failure Point

The reviewer provides notes, but the Groq rewrite request fails.

Possible causes:

* API timeout,
* rate limit,
* authentication issue,
* malformed request,
* service unavailable.

## Why This Matters

If the rewrite fails, the system cannot create a reliable `v2` draft.

## Handling

```text id="zmd6og"
Needs Edit + Reviewer Notes provided
→ Create JSON payload
→ Groq rewrite request fails
→ Update Lead Status = Error
→ Keep review decision visible
→ Send internal/admin alert
→ Do not create AI Draft v2
```

## Airtable Outcome

| Table                 | Outcome                                                    |
| --------------------- | ---------------------------------------------------------- |
| **Leads**             | Lead is updated to `Error`.                                |
| **AI Drafts**         | No `v2` draft is created.                                  |
| **Review & Approval** | Existing review remains available for inspection or retry. |
| **Message Log**       | No customer message log is created.                        |

## Business Result

The failed rewrite is visible, and no incomplete revised draft is created.

---

## 3. JSON Parsing Failure During Needs Edit Rewrite

## Failure Point

Groq returns a rewrite response, but the response cannot be parsed as valid JSON.

Possible causes:

* invalid JSON,
* missing revised reply field,
* unexpected response structure,
* extra prose outside the JSON object.

## Why This Matters

The revised draft must be mapped into Airtable correctly. If parsing fails, the workflow cannot safely create a `v2` draft.

## Handling

```text id="0p58dc"
Groq rewrite response received
→ Parse JSON fails
→ Update Lead Status = Error
→ Send internal/admin alert
→ Do not create AI Draft v2
```

## Airtable Outcome

| Table                 | Outcome                                           |
| --------------------- | ------------------------------------------------- |
| **Leads**             | Lead is updated to `Error`.                       |
| **AI Drafts**         | No reliable revised draft is created.             |
| **Review & Approval** | Review remains available for inspection or retry. |
| **Message Log**       | No customer message log is created.               |

## Business Result

The system avoids storing a broken or incomplete revised reply.

---

## 4. Groq Failure During Rejection Routing

## Failure Point

The reviewer selects `Rejected`, but the Groq rejection-intent classification request fails.

## Why This Matters

The rejection path uses AI only to decide the operational handling of the rejection:

```text id="62yxtp"
Cancel Completely
OR
Manual Human Intervention
```

If the AI cannot classify the reviewer’s intent, the system should not guess.

## Handling

```text id="9myamw"
Approval Status = Rejected
→ Create JSON payload
→ Groq rejection-routing request fails
→ Update Lead Status = Error
→ Send internal/admin alert
→ Do not send customer message
```

## Airtable Outcome

| Table                 | Outcome                                                                         |
| --------------------- | ------------------------------------------------------------------------------- |
| **Leads**             | Lead is updated to `Error`.                                                     |
| **AI Drafts**         | No new draft is created.                                                        |
| **Review & Approval** | Review remains available for inspection.                                        |
| **Message Log**       | No customer message log is created unless a failure log is intentionally added. |

## Business Result

The system avoids incorrectly cancelling or queueing a lead when the reviewer’s rejection notes could not be interpreted.

---

## 5. JSON Parsing Failure During Rejection Routing

## Failure Point

Groq returns a rejection-routing response, but Make cannot parse the response as valid JSON.

Expected output should include a structured rejection outcome such as:

```text id="idxjdw"
Cancel Completely
Manual Human Intervention
```

## Why This Matters

The rejected path updates lead status and message logs based on the parsed outcome. If parsing fails, the workflow cannot safely decide which operational path to follow.

## Handling

```text id="7nwx37"
Groq rejection response received
→ Parse JSON fails
→ Update Lead Status = Error
→ Send internal/admin alert
→ Do not send customer message
```

## Airtable Outcome

| Table                 | Outcome                                                                         |
| --------------------- | ------------------------------------------------------------------------------- |
| **Leads**             | Lead is updated to `Error`.                                                     |
| **AI Drafts**         | No new draft is created.                                                        |
| **Review & Approval** | Review remains available for inspection.                                        |
| **Message Log**       | No customer message log is created unless a failure log is intentionally added. |

## Business Result

The system prevents an unsafe rejection outcome from being applied.

---

## 6. Gmail Send Failure

## Failure Point

The reviewer approves a reply and the workflow attempts to send it by Gmail, but the send fails.

Possible causes:

* Gmail connection issue,
* invalid email address,
* mailbox permission issue,
* temporary provider failure,
* module configuration error.

## Why This Matters

A failed Gmail send must not be treated as a successful follow-up.

## Handling

```text id="mhxrjz"
Approval Status = Approved
→ Gmail send attempted
→ Gmail send fails
→ Update Lead Status = Error
→ Create or update Message Log:
    Channel = Email
    Sent Status = Failed
    Error Message = failure reason
→ Send internal/admin alert
```

## Airtable Outcome

| Table                 | Outcome                                             |
| --------------------- | --------------------------------------------------- |
| **Leads**             | Lead is updated to `Error`.                         |
| **AI Drafts**         | No new draft is created.                            |
| **Review & Approval** | Approved decision remains available for inspection. |
| **Message Log**       | Email outcome is recorded as `Failed`.              |

## Business Result

The team can see that the approved reply was not delivered and can retry or follow up manually.

---

## 7. Slack Notification Failure During Scenario 2

## Failure Point

A Slack notification fails after Scenario 2 processes a decision.

This can happen after:

* an approved email is sent,
* a manual follow-up is queued,
* a revised draft is created,
* a lead is cancelled,
* a manual intervention path is created.

## Why This Matters

Slack is a visibility layer. In most paths, Airtable remains the source of truth. A Slack failure should not undo already completed Airtable updates.

## Handling

```text id="fyg52t"
Slack notification fails
→ Keep completed Airtable updates
→ Preserve Lead Status
→ Preserve Message Log where applicable
→ Send fallback admin alert where available
```

## Airtable Outcome

| Table                 | Outcome                                                               |
| --------------------- | --------------------------------------------------------------------- |
| **Leads**             | Keeps the status already applied by the path.                         |
| **AI Drafts**         | Keeps any draft already created.                                      |
| **Review & Approval** | Keeps the decision or reset state already applied.                    |
| **Message Log**       | Keeps the sent, queued, cancelled, or failed outcome already created. |

## Business Result

The workflow does not lose the business outcome just because the Slack alert failed.

---

# Manual Follow-Up Is Not an Error

Some paths intentionally queue human follow-up.

Manual follow-up is not considered an automation failure.

Examples:

* lead prefers WhatsApp,
* lead prefers a phone call,
* no usable email exists,
* reviewer rejects the draft but requests human intervention,
* the case is sensitive or custom.

In these cases, the system should create a normal operational outcome:

```text id="efca7u"
Lead Status = Manual Follow-Up Required
Message Log Sent Status = Queued
Slack alert = human action required
```

This is different from an error because the system completed the expected safe action.

---

# Status Outcomes

## Lead Status Outcomes

| Lead Status                   | When It Is Used                                                                       |
| ----------------------------- | ------------------------------------------------------------------------------------- |
| **New**                       | Lead has been created but not fully processed yet.                                    |
| **Pending Review**            | Lead is waiting for human review or revised review.                                   |
| **Sent**                      | Approved email follow-up was sent successfully.                                       |
| **Manual Follow-Up Required** | Human action is required for WhatsApp, phone, no-email, or manual intervention paths. |
| **Rejected**                  | Lead was rejected and cancelled without customer follow-up.                           |
| **Error**                     | Automation cannot safely continue or delivery failed.                                 |

---

## Message Log Outcomes

| Channel      | Sent Status   | Meaning                                               |
| ------------ | ------------- | ----------------------------------------------------- |
| **Email**    | **Sent**      | Approved reply was successfully sent by Gmail.        |
| **Email**    | **Failed**    | Gmail send failed.                                    |
| **WhatsApp** | **Queued**    | Manual WhatsApp follow-up is required.                |
| **Phone**    | **Queued**    | Manual phone follow-up is required.                   |
| **Manual**   | **Queued**    | Human intervention is required.                       |
| **None**     | **Cancelled** | Lead was rejected and no customer follow-up was sent. |

---

# What Should Never Happen

The workflow is designed to avoid unsafe outcomes.

The system should never:

* send an AI-generated reply without human approval,
* ignore an invalid form submission silently,
* treat a failed Gmail send as successful,
* continue using an AI response that failed JSON parsing,
* create a revised draft when reviewer notes are missing,
* automatically send WhatsApp or phone follow-ups without human action,
* cancel or manually route a rejected lead when the rejection intent cannot be parsed.

---

# Recovery and Review Process

When a record enters an error path, the team can inspect the relevant Airtable table.

## For Lead Intake Errors

Review:

* Leads table,
* Lead Status,
* Notes field,
* original submitted data.

Common recovery actions:

* correct missing data,
* manually re-run the scenario,
* manually create a review task,
* contact the lead manually if needed.

## For AI or JSON Errors

Review:

* Lead Status,
* Notes field,
* AI request context,
* expected JSON fields,
* scenario execution history.

Common recovery actions:

* fix prompt structure,
* re-run failed module,
* manually regenerate the draft,
* adjust JSON parsing rules.

## For Gmail Errors

Review:

* Message Log,
* Sent Status,
* Error Message,
* lead email address,
* Gmail module execution history.

Common recovery actions:

* correct email address,
* retry Gmail send,
* switch to manual follow-up,
* update Message Log after resolution.

## For Slack Errors

Review:

* Airtable records first,
* scenario run history,
* Slack connection or channel permissions.

Common recovery actions:

* notify reviewer manually,
* fix Slack connection,
* re-run notification module if needed.

---

# Summary

The error-handling design makes the automation safer and more realistic.

Instead of assuming every step succeeds, the workflow explicitly handles:

```text id="29v7ry"
invalid submissions
AI/API failures
JSON parsing failures
missing reviewer notes
Gmail delivery failures
Slack notification failures
manual follow-up requirements
cancelled leads
```

This makes the system suitable as a professional portfolio project because it demonstrates more than basic automation. It shows practical thinking about reliability, human approval, operational visibility, and safe AI-assisted workflows.

