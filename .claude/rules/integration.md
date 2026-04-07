# Rule — Integration and AI Guardrails

Scope: all

## When integrating a feature in the frontend

1. Find the feature's `.md` inside its source folder to understand the
   business rule and how it fits the product.
2. Find the integration doc in `docs/integration/` for the specific feature
   contract (format described in `docs/integration/TEMPLATE.md`).
3. If the correct feature name is unclear, ask before assuming.
4. Integration must always be based on these documents before assuming any
   contract shape.

## AI behavior guardrails

- Do not invent features that do not exist yet.
- Do not downgrade any product information to a superficial sentence.
- When describing an existing feature, explain its role and expected
  experience, not just its name.
- When describing a future feature, make it explicitly clear it is future.
- Preserve Societiza as the central platform for corporate operations.
- Treat the societario as the current product core.
- Treat the dashboard as the next mandatory feature.
- Treat configurable fields in steps and tasks as the current data capture
  mechanism — do not confuse this with a consolidated native form module.
- Make technical gaps explicit instead of masking them with vague language.

## Reason

These rules prevent misalignment between what exists, what is planned, and
what gets communicated to humans or executed by agents.