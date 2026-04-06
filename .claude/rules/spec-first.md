# Rule — Spec Before Implementation

Scope: orchestration

## Never do

- Start implementation in any repo without an approved spec
- Skip human approval of the spec
- Treat a conversation summary as a spec

## Always do

- Generate a spec using the feature-design skill
- Save the spec to `features/specs/[feature-name].md`
- Obtain explicit human approval before generating contracts
- After approval, generate the contract and map impact

## Reason

The spec is the quality control point for product decisions. Without it,
execution repos build features that may not fit the product or may conflict
with existing features.
