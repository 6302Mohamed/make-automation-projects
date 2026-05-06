<p align="center">
  <h1 align="center">Make Automation Projects</h1>
  <p align="center">
    A growing portfolio of practical business automation workflows built with Make.com.
  </p>
  <p align="center">
    Designed, documented, and maintained by <strong>Mohamed Yousuf Hussein</strong>
  </p>
</p>

---

## Overview

This repository contains practical workflow automation projects built with Make.com around real business use cases.

The purpose of this repository is not just to store exported scenarios, but to document how each workflow is designed, structured, and implemented. Each project is treated as a standalone automation case with its own README, notes, blueprint, and supporting material where relevant.

As the repository grows, it will cover workflow patterns such as:

- intake and request routing
- notifications and escalations
- validation and error handling
- duplicate detection
- invoice and order workflows
- spreadsheet-driven operations
- Airtable-supported record management
- operational business process automation

---

## Repository Structure

Each project is organized in its own folder and may include:

- `README.md` — project overview and workflow explanation
- exported Make scenario blueprint
- workflow notes and assumptions
- screenshots of the scenario
- supporting documentation where needed

---

## Projects

### Project 01 — Agency Intake Routing
A Make.com workflow for processing incoming agency requests from a form connected to Google Sheets.

It includes:
- request validation
- invalid submission handling
- duplicate detection
- routing by priority and category
- audit-friendly status updates

### Project 02 — E-commerce Order Processing, Invoice Logging & Team Notification
A Make.com workflow for processing paid e-commerce orders through a custom webhook and logging them into structured storage.

It includes:
- required field validation
- invalid payload logging
- duplicate order detection
- order header logging
- line-item iteration and logging
- Airtable invoice record creation
- buyer confirmation email
- Slack team notification
- Telegram fallback alerting

More projects will be added over time.

---

## Purpose

This repository serves as:

- a portfolio of Make.com automation work
- a record of hands-on workflow design practice
- a place to document real business automation patterns
- a foundation for more advanced automation projects in the future

---

## Notes

Public project files in this repository use sanitized placeholders for sensitive values such as:

- email addresses
- company names
- private sheet references
- organization-specific details
- webhook URLs
- tokens and connection details

Replace those values with your own environment-specific settings before using any blueprint in production.

---

## Author

**Mohamed Yousuf Hussein**  
AI Agent Engineer | Automation Builder | Workflow Designer
