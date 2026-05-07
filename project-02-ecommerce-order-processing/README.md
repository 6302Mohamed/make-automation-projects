# Project 02 — E-commerce Order Processing, Invoice Logging & Team Notification

A Make.com workflow that receives paid e-commerce orders through a webhook, validates incoming payloads, prevents duplicate processing, logs order headers and individual line items, creates invoice records in Airtable, sends buyer follow-up emails, and notifies internal teams through Slack with Telegram fallback support.

> **Template availability:** A sanitized reference blueprint is included for portfolio purposes. Environment-specific connections, IDs, and configuration details were removed or replaced, so manual reconnection and remapping may be required before reuse.

---

## Overview

This project was built as a practical business automation workflow for handling paid e-commerce orders in a more structured, traceable, and operations-friendly way.

Instead of treating an order as a single flat record, the workflow separates:

- **valid orders**
- **invalid payloads**
- **duplicate events**
- **order-level records**
- **line-item-level records**
- **invoice records**
- **team notifications**
- **buyer communication**

The result is a workflow that feels closer to a real internal operations process than a simple demo automation.

---

## Business Problem

Manual order handling often creates avoidable operational friction:

- duplicate orders may be processed twice
- invalid payloads may be ignored without visibility
- line items may not be stored in a usable structure
- finance may not have a clear invoice record
- teams may learn about new orders too late
- notification failures may go unnoticed unless there is escalation

In a real business setting, even small failures here can create:

- order confusion
- reporting inaccuracies
- extra admin time
- delayed internal response
- weak auditability

This workflow was designed to reduce those risks and make order handling more consistent.

---

## Business Outcome

This automation improves the workflow in four practical ways:

### 1. Better operational visibility
Orders are no longer just “received.” They are classified, logged, and tracked with status fields.

### 2. Better data structure
Order headers and line items are separated properly, which makes downstream reporting and operations easier.

### 3. Better exception handling
Invalid payloads, duplicate events, and duplicate-check failures are not silently ignored. They are surfaced for review or escalated to humans.

### 4. Better communication
Both internal teams and the buyer receive the right message at the right stage of the workflow.

---

## Expected ROI / Value

This project is a portfolio workflow, so it does **not** claim measured production ROI. However, the business value is grounded and realistic.

In a real environment, a workflow like this can reasonably help reduce:

- **manual order logging time**
- **duplicate-processing mistakes**
- **time spent investigating missing or malformed orders**
- **handoff delays between operations and finance**
- **notification gaps when the main channel fails**

### Grounded impact examples
Depending on order volume, even a modest implementation can create value such as:

- saving a few minutes per order that would otherwise be spent on manual logging
- reducing duplicate-handling mistakes from repeated intake events
- making invalid submissions reviewable instead of invisible
- giving finance a cleaner starting point through invoice records
- reducing time-to-awareness for internal teams through automated notifications

The exact ROI depends on:
- order volume
- current manual process maturity
- team size
- frequency of duplicate or malformed payloads
- internal cost of errors and delays

So the claim here is **not hype**. The claim is that this workflow targets real sources of operational waste.

---

## What the Workflow Does

- receives a paid order through a custom webhook
- validates required fields before processing
- logs invalid payloads into a dedicated sheet
- checks existing records for duplicate `order_id` values
- logs duplicates separately instead of reprocessing them
- stores the order header in Google Sheets
- creates an invoice record in Airtable
- sends a buyer confirmation email
- iterates through all line items and stores them individually
- sends an internal Slack notification
- falls back to Telegram if Slack fails
- updates final processing and notification statuses

---

## Key Skills Demonstrated

- webhook-based workflow intake
- validation and branching logic
- duplicate prevention
- multi-step business process design
- structured logging across systems
- Google Sheets operational storage
- Airtable invoice record creation
- buyer-facing email automation
- internal team notification design
- fallback alerting
- Iterator and Array Aggregator usage
- workflow state/status updates
- human escalation for failure scenarios

---

## Tools Used

- **Make.com**
- **Custom Webhook**
- **Google Sheets**
- **Airtable**
- **Slack**
- **Telegram Bot**
- **Email module**
- **Postman**

---

## Data Structure

### Google Sheets — Orders
- `order_id`
- `order_timestamp`
- `customer_name`
- `customer_email`
- `payment_status`
- `total_amount`
- `item_count`
- `processed_at`
- `processing_status`
- `notification_status`

### Google Sheets — Order_Items
- `order_id`
- `item_id`
- `product_name`
- `sku`
- `quantity`
- `unit_price`
- `line_total`
- `logged_at`

### Google Sheets — Duplicates
- `order_id`
- `order_timestamp`
- `customer_name`
- `customer_email`
- `total_amount`
- `detected_at`
- `duplicate_reason`

### Google Sheets — Invalid_Orders
- `received_at`
- `raw_order_id`
- `customer_name`
- `customer_email`
- `validation_reason`
- `raw_payload_note`

### Airtable — Invoices
- `invoice_id`
- `order_id`
- `customer_name`
- `customer_email`
- `invoice_amount`
- `invoice_status`
- `generated_at`

---

## Workflow Logic

1. A custom webhook receives a new paid-order payload.
2. The workflow validates the required fields: `order_id`, `order_timestamp`, and `line_items`.
3. If validation fails, the payload is logged to `Invalid_Orders` and processing stops.
4. If validation passes, existing order records are searched using `order_id`.
5. If the duplicate check itself fails, the workflow escalates to a human-facing error path and stops to avoid unsafe processing.
6. If the `order_id` already exists, the order is logged to `Duplicates` and the workflow ends.
7. If the order is new, the order header is logged in the `Orders` sheet.
8. An invoice record is created in Airtable.
9. A buyer confirmation email is sent.
10. The `line_items` array is iterated so each product can be handled individually.
11. Each line item is logged in the `Order_Items` sheet.
12. The item bundles are regrouped using an Array Aggregator.
13. A Slack summary notification is sent to the internal team.
14. If Slack fails, Telegram sends the fallback alert.
15. The main order record is updated with final processing and notification statuses.

---

## Human Escalation Design

One of the most important parts of this workflow is that failures are **not treated as invisible technical events**.

### Human escalation happens when:
- the workflow cannot safely validate required input
- duplicate verification cannot be completed reliably
- the primary internal notification channel fails

### Why this matters
A business does not just need automation. It needs **safe automation**.

This workflow is designed so that:
- invalid or risky cases are surfaced
- duplicate uncertainty is escalated instead of guessed
- Slack failure does not leave the team blind
- humans can step in when the workflow reaches a point where automation should not continue blindly

That makes the system more trustworthy for real operations.

---

## Notification Design

### Buyer Email
The workflow sends a customer-facing confirmation email after invoice record creation.

### Slack
Slack is used as the primary internal team notification channel for successful order processing visibility.

### Telegram
Telegram is used as the fallback notification channel when Slack delivery fails.

### Error / Escalation Behavior
If a critical processing step cannot be safely completed, the workflow is designed to escalate rather than continue blindly.

---

## Trade-Offs

This project deliberately makes a few design trade-offs to stay realistic without becoming bloated.

### Trade-off 1: Paid orders only
The workflow focuses on **paid orders only**.
That keeps the scenario clean and avoids mixing confirmed-order handling with abandoned or pending-payment recovery workflows.

### Trade-off 2: Invoice record, not invoice PDF
The workflow creates a structured invoice record in Airtable rather than generating a full PDF invoice.
This is a practical middle ground that supports finance visibility without adding fake complexity.

### Trade-off 3: Multi-system storage
Google Sheets is used for operational logging, while Airtable is used for invoice records.
This is slightly more complex than using one tool only, but it better reflects how real businesses often separate operational tracking from structured records.

### Trade-off 4: Sanitized public blueprint
The shared blueprint is intentionally sanitized for safety.
That means it is useful as a reference, but may require remapping before reuse.

---

## Edge Cases Handled

- missing required fields
- duplicate `order_id`
- duplicate-check lookup failure
- Slack notification failure
- invalid payload logging
- multi-line-item orders

---

## Sample Use Cases

- internal order operations logging
- structured finance/invoice tracking
- team alerts for new confirmed orders
- buyer confirmation workflows
- future extensions for fulfillment or accounting systems

---

## Why This Project Matters

This project shows that Make.com can be used for more than simple form automation.

It demonstrates practical workflow design across:

- validation
- duplicate prevention
- exception visibility
- line-item iteration
- regrouping after item-level processing
- invoice record generation
- customer communication
- fallback-based team notifications
- human escalation when safe automation boundaries are reached

For recruiters or clients, the point is not just that the workflow runs.  
The point is that it was designed with **business consequences** in mind.

---

## Future Improvements

- real invoice PDF generation
- accounting platform integration
- fulfillment handoff flow
- item-level validation rules
- source-system tagging for WooCommerce / Shopify / other platforms
- richer internal alert formatting
- retry/reporting pattern for customer email failures

---

## Files Included

This project folder includes or is intended to include:

- `README.md`
- sanitized reference blueprint
- screenshots of the scenario
- workflow notes

---

## Author

**Mohamed Yousuf Hussein**  
AI Agent Engineer | Automation Builder | Workflow Designer
