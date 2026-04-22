# Workflow Notes

## Version 1 assumptions

- Source trigger is Google Sheets using Google Form responses
- Required fields:
  - Name
  - Email
  - Category
  - Priority
  - Message
- Company is optional
- Invalid submissions are marked and separated
- Duplicate detection uses matching fields plus a short time window
- Non-duplicate requests continue to normal routing
- Emails are routed by priority and category
- Public version uses placeholder emails

## Audit fields

- Status
- Destination Team
- Notification Channel
- Notified At
- Notes

## Learning outcomes

- Using routers and route filters correctly
- Keeping raw input separate from audit updates
- Using search before duplicate filtering
- Designing workflow logic before module logic
