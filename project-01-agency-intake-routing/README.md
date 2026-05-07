
# Project 01 | Agency Intake Routing

A Make.com automation workflow for processing agency requests submitted through a Google Form connected to Google Sheets.

This project demonstrates how a service-based business can organize incoming requests, validate submissions, detect possible duplicates, route inquiries based on priority and category, and keep audit-friendly records for operational visibility.

> **Template availability:** A sanitized reference blueprint is included for portfolio purposes. Environment-specific emails, sheet references, connection details, and configuration values were removed or replaced. Manual reconnection and remapping may be required before reuse.

---

## Overview

This project was built as a practical intake automation workflow for agencies or service-based businesses that receive different types of requests through forms.

Instead of allowing every form submission to become an unstructured row in a spreadsheet, the workflow adds a business process around the intake:

- request capture
- required-field validation
- invalid request handling
- duplicate detection
- priority-based routing
- category-based routing
- email notification
- audit-friendly status updates

The result is a cleaner intake process where requests are easier to review, route, and follow up on.

---

## Business Problem

Agencies often receive mixed inbound requests from different people and for different reasons.

A single intake form may receive:

- new client leads
- support requests
- content requests
- partnership inquiries
- unclear or incomplete submissions

Without structure, this creates operational problems:

- important leads may be missed
- duplicate submissions may create confusion
- incomplete requests may waste staff time
- urgent requests may not be prioritized
- requests may be sent to the wrong person
- managers may not know what happened to each submission
- follow-up depends too much on manual checking

This workflow introduces structure so incoming requests can be handled more consistently.

---

## Solution Overview

The automation watches for new form responses in Google Sheets and processes each request through a controlled routing workflow.

At a high level, the workflow:

1. Detects a new form submission.
2. Checks whether required information is present.
3. Marks incomplete requests as invalid.
4. Checks whether the request may be a duplicate.
5. Marks duplicate requests separately.
6. Routes valid, non-duplicate requests based on priority and category.
7. Sends the request to the appropriate destination.
8. Updates audit fields in the source sheet.

This creates a simple but useful intake system for small teams that need more structure without building a full CRM.

---

## Architecture

```txt
Google Form Submission
        │
        ▼
Google Sheets Response Row
        │
        ▼
Make.com Scenario
        │
        ├── Validate Required Fields
        │       └── Invalid Request → Mark in Google Sheets
        │
        ├── Duplicate Detection
        │       └── Duplicate Request → Mark in Google Sheets
        │
        └── Valid New Request
                ├── Priority-Based Routing
                ├── Category-Based Routing
                ├── Email Notification
                └── Audit Field Updates
````

---

## Tools Used

<p>
  <img src="https://img.shields.io/badge/Make.com-Scenario%20Builder-6D00CC?style=flat-square&logo=make&logoColor=white" />
  <img src="https://img.shields.io/badge/Google%20Forms-Request%20Intake-7248B9?style=flat-square&logo=googleforms&logoColor=white" />
  <img src="https://img.shields.io/badge/Google%20Sheets-Request%20Tracking-34A853?style=flat-square&logo=googlesheets&logoColor=white" />
  <img src="https://img.shields.io/badge/Email-Team%20Routing-EA4335?style=flat-square&logo=gmail&logoColor=white" />
</p>

* **Make.com** — workflow orchestration
* **Google Forms** — request intake
* **Google Sheets** — request tracking and audit fields
* **Email modules** — category-based team routing and notifications

---

## What the Workflow Does

* detects new requests from a Google Forms response sheet
* checks whether required fields are present
* marks incomplete submissions as invalid
* checks for possible duplicate requests within a time window
* marks duplicate submissions in the source sheet
* routes valid requests based on priority and category
* sends email notifications to the relevant destination
* updates audit fields such as status, destination, notification result, and processing notes

---

## Workflow Logic

1. A new row is detected from the Google Forms response sheet.
2. The workflow checks whether the required fields are complete.
3. If required fields are missing, the request is marked as invalid and processing stops.
4. If the request is valid, the workflow checks for possible duplicates.
5. Duplicate detection uses a time-window approach to reduce repeated processing from similar submissions.
6. If a duplicate is found, the row is marked as duplicate in the source sheet.
7. If the request is valid and not a duplicate, it continues to routing.
8. The workflow evaluates request priority.
9. The workflow evaluates request category.
10. The request is routed to the correct email destination.
11. Audit fields are updated to show how the request was handled.

---

## Example Request Categories

The workflow supports common agency intake categories such as:

* Lead
* Support
* Content
* Partnership
* Not sure

These categories can be adapted for different business types, such as consulting firms, marketing agencies, freelancers, clinics, service providers, or internal operations teams.

---

## Routing Design

The workflow uses both **priority** and **category** to decide how a request should be handled.

### Priority-based routing

Priority helps the team identify which requests need faster attention.

Examples:

* high-priority client issues
* urgent support cases
* important new business leads
* time-sensitive partnership inquiries

### Category-based routing

Category helps send the request to the right person or team.

Examples:

* lead requests can go to sales
* support issues can go to support/admin
* content requests can go to the content team
* partnership inquiries can go to business development
* unclear submissions can go to manual review

This design keeps the workflow simple while still reflecting how real teams separate responsibilities.

---

## Data & Audit Fields

The source Google Sheet can include request data and automation tracking fields.

Example request fields:

| Field       | Purpose                     |
| ----------- | --------------------------- |
| `timestamp` | Time the form was submitted |
| `name`      | Requester name              |
| `email`     | Requester email             |
| `category`  | Type of request             |
| `priority`  | Request urgency             |
| `message`   | Request details             |

Example audit fields:

| Field                  | Purpose                                                          |
| ---------------------- | ---------------------------------------------------------------- |
| `processing_status`    | Whether the request was processed, invalid, duplicate, or routed |
| `assigned_destination` | Team/person/email destination                                    |
| `notification_status`  | Whether notification was sent                                    |
| `processed_at`         | Time the automation handled the request                          |
| `processing_notes`     | Short explanation of what happened                               |

Audit fields make the workflow more transparent because the team can see what happened to each request directly inside the sheet.

---

## Key Skills Demonstrated

* Make.com scenario design
* Google Forms to Google Sheets automation
* required-field validation
* invalid submission handling
* duplicate detection
* time-window based comparison logic
* router and filter usage
* priority-based routing
* category-based routing
* email notification design
* audit-friendly status updates
* lightweight operations workflow design
* sanitized documentation for public portfolio use

---

## Business Outcome

This automation improves request handling in four practical ways.

### 1. Fewer missed requests

Incoming requests are automatically checked and routed instead of depending only on manual review.

### 2. Cleaner intake records

Invalid and duplicate submissions are marked clearly, which keeps the source sheet easier to understand.

### 3. Faster internal routing

Requests can be sent to the right destination based on category and priority.

### 4. Better operational visibility

Audit fields show whether a request was processed, routed, marked invalid, or marked as duplicate.

---

## Expected ROI / Business Value

This project is a portfolio workflow, so it does **not** claim measured production ROI.

However, the business value is realistic for agencies and service-based teams.

A workflow like this can help reduce:

* manual form checking
* missed inquiry risk
* duplicate request confusion
* time spent deciding who should handle each request
* delays in responding to important leads or support issues
* messy spreadsheet tracking

### Grounded impact examples

Depending on request volume, this workflow could create value by:

* saving admin time on request review
* helping urgent requests reach the right person faster
* keeping lead and support intake cleaner
* reducing duplicate follow-up
* making form submissions easier to audit
* giving small teams a lightweight intake system without needing a full CRM

The exact ROI depends on:

* number of weekly submissions
* team size
* current manual review process
* number of categories handled
* importance of response speed
* cost of missed or delayed follow-up

The claim is simple: this workflow targets common intake problems that waste time and reduce follow-up quality.

---

## Trade-Offs & Assumptions

This project makes a few deliberate trade-offs to stay practical.

### Trade-off 1: Google Sheets as the main tracking system

Google Sheets is used as the operational tracker because it is simple, familiar, and easy for small teams to review.

For larger teams, this could later be extended to a CRM such as HubSpot, Pipedrive, Zoho CRM, or Airtable.

### Trade-off 2: Email routing instead of full task management

The portfolio version uses email notifications for routing.

A production version could create tasks in tools like Trello, Asana, ClickUp, Monday.com, or Notion.

### Trade-off 3: Simple intake categories

The category list is intentionally simple.

A real business could expand it with more detailed departments, service types, lead sources, or SLA rules.

### Trade-off 4: Time-window duplicate detection

Duplicate detection is based on a practical time-window approach.

This is useful for catching repeated submissions, but a production system may need more advanced matching rules depending on the business.

### Trade-off 5: Sanitized public blueprint

The shared blueprint is intentionally sanitized for safety.

This makes it safe for GitHub, but it may require reconnection, remapping, and testing before reuse.

---

## Edge Cases Handled

* missing required fields
* incomplete requester information
* duplicate submissions
* unclear request categories
* priority-based routing differences
* category-based routing differences
* audit status updates after processing

---

## Example Business Use Cases

This workflow pattern can be adapted for:

* agency client intake
* freelancer inquiry management
* consulting request forms
* support request routing
* partnership inquiry handling
* content request management
* internal operations requests
* service booking requests
* lead routing for small businesses
* lightweight CRM-style request tracking

---

## Security & Sanitization

The public version of this project is sanitized.

Sensitive values were removed or replaced, including:

* private emails
* sheet IDs
* connection references
* organization-specific destinations
* internal routing details
* environment-specific configuration values

Before using this workflow in a real environment, the blueprint should be reviewed and reconnected with secure production credentials.

---

## Files Included

This project folder includes:

* `README.md`
* `workflow-notes.md`
* sanitized Make.com blueprint reference

Example structure:

```txt
project-01-agency-intake-routing/
│
├── README.md
├── workflow-notes.md
└── agency-intake-routing-duplicate-detection.json
```

Screenshots may be added later when available.

---

## Future Improvements

Possible future improvements include:

* screenshots of the full Make.com scenario
* CRM integration with HubSpot, Pipedrive, Zoho CRM, or Airtable
* task creation in Trello, Asana, ClickUp, Monday.com, or Notion
* SLA-based routing for urgent requests
* auto-reply emails to requesters
* lead scoring based on request content
* AI-assisted request classification
* Slack or Telegram notifications for urgent requests
* dashboard summary for weekly intake volume
* richer duplicate matching using email, name, message similarity, and timestamp

---

## Why This Project Matters

This project shows how Make.com can turn a basic form submission process into a lightweight business intake system.

It demonstrates workflow thinking across:

* request capture
* validation
* duplicate detection
* routing
* notification
* audit visibility
* operational documentation
* public-safe blueprint sharing

For recruiters or clients, the point is not just that the scenario runs.

The point is that the workflow was designed with real intake problems in mind: missed requests, duplicate submissions, unclear routing, and lack of visibility.

---

## Author

**Mohamed Yousuf Hussein**
**AI Engineer | Workflow Automation**

<p>
  <a href="https://github.com/6302Mohamed">
    <img src="https://img.shields.io/badge/GitHub-6302Mohamed-181717?style=flat-square&logo=github&logoColor=white" />
  </a>
</p>
```
