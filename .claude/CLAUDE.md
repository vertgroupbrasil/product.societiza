# Societiza Skills — Orchestration Hub

Knowledge and orchestration repository for the Societiza ecosystem.
This repo does not contain application code. It produces feature specs,
contracts, and impact maps that the execution repos consume.

---

## Role in the ecosystem

```
skills/              <- this repo (orchestrator)
  receives feature intent, produces specs, generates contracts

api.societiza/       <- backend execution
  reads specs and contracts, implements .NET backend

web.societiza/       <- frontend execution
  reads contracts, implements Next.js frontend
```

Human interaction happens here — spec approval before any implementation
starts in execution repos.

---

## Repository structure

```
.claude/
  CLAUDE.md            <- this file
  settings.json        <- permissions
  rules/               <- constraints applied every session
  skills/              <- specialized knowledge loaded on demand
  commands/            <- reusable workflows
  agents/              <- reserved for future sub-agents

docs/
  product/             <- vision, values, priorities
  context/             <- societario, accountancy, auth
  planning/            <- dashboard, future features, feature map
  integration/         <- backend-to-frontend integration template

contracts/
  api/                 <- versioned OpenAPI contracts (consumed by both repos)

features/
  specs/               <- approved feature specifications

  business-context.md  <- product context index
```

---

## Existing documentation modules

| Path | Content |
|------|---------|
| `docs/product/product-vision.md` | Identity, ICP, values, feature guidelines |
| `docs/product/product-priorities.md` | P1-P5 priority semantics |
| `docs/context/societario-context.md` | Workflow, template, board, fields |
| `docs/context/accountancy-context.md` | Multi-tenant discriminant |
| `docs/context/authentication-context.md` | Auth state and gaps |
| `docs/planning/dashboard.md` | Next mandatory feature |
| `docs/planning/future-features.md` | All planned future features |
| `docs/planning/feature-map.md` | Feature dependencies and coupling |

---

## Where to find what

| Task | Read first |
|------|-----------|
| Plan a new feature | `.claude/skills/feature-design/SKILL.md` |
| Understand product context | `docs/business-context.md` (index) |
| Check feature dependencies | `docs/planning/feature-map.md` |
| Map cross-repo impact | `.claude/commands/impact-map.md` |
| Full new-feature workflow | `.claude/commands/new-feature.md` |

Rules active every session: `.claude/rules/`

---

## Migration notes

`societiza-feature-plan/` at repo root is pending migration to execution repos.
See `societiza-feature-plan/MIGRATING.md` for details.
