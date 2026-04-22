# Project 01: Agency Intake Routing

## Overview

This project is a Make.com workflow for processing agency requests submitted through a Google Form connected to Google Sheets.

It validates incoming requests, flags invalid submissions, detects duplicates, routes requests based on business rules, and updates records for audit visibility.

## Business problem

Agencies can receive mixed inbound requests such as leads, support issues, content requests, and partnerships. Without structure, requests can be missed, duplicated, or sent to the wrong destination.

This workflow introduces validation, duplicate detection, priority handling, and category-based routing.

## Tools used

- Make.com
- Google Forms
- Google Sheets
- Email modules

## Workflow summary

1. A new row is detected from the form response sheet.
2. The request is checked for required fields.
3. Invalid rows are marked and handled separately.
4. Valid rows are checked for possible duplicates.
5. Duplicate rows are marked in the source sheet.
6. Non-duplicate rows continue through normal routing.
7. Requests are routed by priority and category.
8. Audit fields such as status, destination, and notification details are updated.

## Main features

- Required-field validation
- Invalid request handling
- Duplicate detection with a time window
- Priority-based routing
- Category-based email routing
- Audit-friendly status updates

## Example categories

- Lead
- Support
- Content
- Partnership
- Not sure

## Files in this folder

- `README.md` — project explanation
- `agency-intake-routing-duplicate-detection.json` — exported Make blueprint
- `workflow-notes.md` — assumptions and workflow notes

## Notes

This project uses placeholder values for public sharing. Replace emails and environment-specific values before production use.
