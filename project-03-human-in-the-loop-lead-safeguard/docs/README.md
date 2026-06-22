# Project Documentation

This folder contains the technical documentation for the **AI Lead Review & Follow-Up System**.

## Documentation Index

| Document | Purpose |
|---|---|
| [architecture.md](architecture.md) | Explains the overall system architecture and how Make.com, Airtable, Tally, Groq, Gmail, and Slack work together. |
| [airtable-schema.md](airtable-schema.md) | Documents the Airtable tables, fields, dropdown values, views, linked records, and lookup fields. |
| [scenario-logic.md](scenario-logic.md) | Explains the step-by-step logic of Scenario 1 and Scenario 2. |
| [error-handling.md](error-handling.md) | Details failure paths, fallback behavior, status outcomes, and recovery logic. |
| [prompt-engineering-and-safety.pdf](prompt-engineering-and-safety.pdf) | Explains prompt design, JSON contracts, scope control, fallback behavior, and safety decisions. |

## Diagrams

| Diagram | Purpose |
|---|---|
| [scenario-1-lead-intake-ai-draft-diagram.png](scenario-1-lead-intake-ai-draft-diagram.png) | Visual overview of the lead intake, validation, AI draft generation, and review preparation flow. |
| [scenario-2-human-review-followup-diagram.png](scenario-2-human-review-followup-diagram.png) | Visual overview of the human review, approved follow-up, needs-edit loop, and rejection routing flow. |

## Recommended Reading Order

1. Start with [architecture.md](architecture.md).
2. Review [scenario-logic.md](scenario-logic.md).
3. Check [airtable-schema.md](airtable-schema.md).
4. Read [error-handling.md](error-handling.md).
5. Open [prompt-engineering-and-safety.pdf](prompt-engineering-and-safety.pdf) for prompt design details.

These documents are written to make the project easy to review without needing direct access to the live Make.com or Airtable workspace.
