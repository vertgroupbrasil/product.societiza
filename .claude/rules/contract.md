# Rule — Contract as Interface

Scope: orchestration

## Never do

- Define API contracts outside of `contracts/api/`
- Let frontend or backend assume a contract that is not published here
- Modify a published contract without versioning

## Always do

- Version contracts (v1/, v2/, etc.)
- Validate contracts against the real backend before publishing
- Include request/response shapes, error envelopes, and auth requirements

## Reason

Contracts in `contracts/api/` are the single interface between repos. Both
execution repos consume them independently. Unversioned or informal contracts
cause integration drift.
