# Rule — Cross-Repo Boundary

Scope: orchestration

## Never do

- Write application code (.cs, .tsx, .ts, .csproj, package.json)
- Create components, hooks, services, or endpoints
- Alter database migrations or infrastructure config
- Make implementation decisions that belong to execution repos

## Always do

- Produce feature specs in `features/specs/`
- Reference execution repos by name (api.societiza, web.societiza)
- Map cross-repo impact before distributing work

## Reason

This repo orchestrates product decisions and distributes them. It does not
execute. Implementation happens in api.societiza and web.societiza.