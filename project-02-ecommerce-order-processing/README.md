
# Project 02 — E-commerce Order Processing, Invoice Logging & Team Notification

A Make.com workflow that receives paid e-commerce orders through a webhook, validates incoming payloads, prevents duplicate processing, logs order headers and individual line items, creates invoice records in Airtable, sends buyer follow-up emails, and notifies internal teams through Slack with Telegram fallback support.

## Overview

This project was built as a practical business automation workflow for handling paid e-commerce orders in a more structured and transparent way.

Instead of treating an order as a single flat record, the workflow separates order-level data from line-item-level data, checks for duplicates before processing, logs invalid payloads for auditability, creates an invoice record, and sends internal as well as customer-facing communication.

It is designed to reflect a realistic internal operations workflow rather than a simple demo automation.

## Business Problem

Manual order handling can create several operational problems:

- duplicate order processing
- poor visibility into individual line items
- invalid payloads being ignored or lost
- weak internal communication when orders arrive
- no structured invoice record for downstream finance handling
- silent notification failures

This workflow was designed to reduce those problems by creating a more reliable order intake and processing pipeline.

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

## Tools Used

- Make.com
- Custom Webhook
- Google Sheets
- Airtable
- Slack
- Telegram Bot
- Email module
- Postman

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

## Workflow Logic

1. A custom webhook receives a new order payload.
2. The workflow validates the required fields: `order_id`, `order_timestamp`, and `line_items`.
3. If validation fails, the payload is logged to `Invalid_Orders` and the workflow stops.
4. If validation passes, existing order records are searched using `order_id`.
5. If the duplicate check fails, the workflow alerts the team and stops to avoid unsafe processing.
6. If the `order_id` already exists, the order is logged to `Duplicates` and the workflow ends.
7. If the order is new, the order header is logged in the `Orders` sheet.
8. An invoice record is created in Airtable.
9. A buyer confirmation email is sent.
10. The `line_items` array is iterated so each product can be handled individually.
11. Each line item is logged in the `Order_Items` sheet.
12. The item bundles are regrouped using an Array Aggregator.
13. A Slack summary notification is sent to the team.
14. If Slack fails, Telegram sends the fallback alert.
15. The main order record is updated with final processing and notification statuses.

## Notification Design

### Buyer Email
The workflow sends a customer-facing confirmation email after invoice record creation.

### Slack
Slack is used as the primary internal team notification channel.

### Telegram
Telegram is used as the fallback notification channel when Slack delivery fails.

## Edge Cases Handled

- missing required fields
- duplicate `order_id`
- duplicate-check lookup failure
- Slack notification failure
- invalid payload logging
- multi-line-item orders

## Sample Use Cases

- internal order operations logging
- structured finance/invoice tracking
- team alerts for new confirmed orders
- buyer confirmation workflows
- future extensions for fulfillment or accounting systems

## Why This Project Matters

This project demonstrates that Make.com can be used for more than basic form automation. It shows practical workflow design across validation, duplicate prevention, array handling, relational-style logging, invoice record generation, customer communication, and fallback-based team notifications.

## Future Improvements

- real invoice PDF generation
- accounting platform integration
- fulfillment handoff flow
- item-level validation rules
- source-system tagging for WooCommerce / Shopify / other platforms

## Files Included

This project folder is intended to contain:

- `README.md`
- scenario blueprint export
- screenshots of the scenario
- workflow notes

## Author

Mohamed Yousuf Hussein  
AI Agent Engineer | Automation Builder | Workflow Designer
