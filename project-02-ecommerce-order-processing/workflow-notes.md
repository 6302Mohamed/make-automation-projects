# Workflow Notes — Project 02

## Purpose

This project was built to simulate a realistic paid e-commerce order processing workflow using Make.com.

The goal was to design a workflow that does more than simply capture orders. It separates valid, duplicate, and invalid cases, logs order headers and line items in structured storage, creates invoice records, and sends internal and customer-facing communication.

## Key Design Decisions

### 1. Paid orders only
This workflow is scoped for paid orders only. Unpaid or incomplete payment cases were intentionally excluded to keep the project focused and avoid mixing order confirmation with payment recovery workflows.

### 2. Duplicate rule
Duplicate detection is based on `order_id`. This was chosen because it is the most reliable identifier for confirmed order events in this scenario.

### 3. Structured storage
Google Sheets is used for operational logging:
- Orders
- Order_Items
- Duplicates
- Invalid_Orders

Airtable is used separately for invoice records to make the project feel closer to a real-world business stack.

### 4. Notification design
Slack is the primary internal alert channel.
Telegram is the fallback notification channel.
A buyer email is also sent after invoice record creation.

### 5. Array handling
The `line_items` array is iterated so that each item can be logged separately. After item processing, an Array Aggregator is used to regroup the bundles before the final order-level notification and status update.

## Important Lessons From Building

- required field validation should happen before duplicate logic
- duplicate checking should happen before any new-order logging
- order-level actions and item-level actions should be separated clearly
- anything after an Iterator runs once per item unless the flow is regrouped
- Array Aggregator is useful when you want a single order-level step after item-level processing
- fallback notifications should serve a real purpose, not just add complexity

## Implementation Notes

- The project uses a custom webhook and Postman for test payload simulation.
- The order header is logged before line item processing.
- Invoice creation is implemented as an invoice record, not as a PDF document.
- Buyer communication is intentionally lightweight and confirmation-focused.
- Slack summary messaging is sent only after regrouping to avoid duplicate alerts per line item.

## Future Improvement Notes

Possible future versions could add:
- invoice PDF generation
- fulfillment routing
- accounting integration
- source tagging for real store platforms
- more detailed validation reasons
- item-level validation
