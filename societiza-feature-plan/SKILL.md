---
name: societiza-feature-plan
description: Code implementation planning skill for Societiza. Use when you know what feature to build and need to plan the implementation. Asks which feature, looks up its business rules, asks frontend or backend, reads ALL markdown files in the corresponding code-rules folder if the user answered frontend, search for frontend-code-rules and vice-versa, then plans the implementation. Hard boundary: backend blocks frontend entirely; frontend blocks backend by default with explicit-permission exception.
user-invocable: true
argument-hint: "[feature name]"
---

# Societiza Feature Plan

This skill plans code implementation for Societiza features.

It is not a product skill. It assumes the feature's business rules are already
defined — either in `docs/business-context.md`, `docs/feature-map.md`, or
`docs/features/product/`. If the feature is not defined yet, this skill stops
and redirects to `societiza-product-plan` first.

---

## Step 1 — Identify the feature

If the feature name was not provided as an argument, ask the user:

> "Qual feature você quer implementar?"

Once a feature name is provided:

1. Search `docs/business-context.md` for mentions of that feature.
2. Search `docs/feature-map.md` for that feature's dependencies and integrations.
3. Search `docs/features/product/` for a product spec document for that feature.

**If the feature is found in any of these documents:**
Load its business context. Summarize what the feature is, what problem it solves,
and how it connects to the rest of the product before moving to Step 2.

**If the feature is NOT found in any of these documents:**
Stop. Respond with:

> "Não encontrei a regra de negócio para essa feature nos documentos de produto.
> Antes de planejar a implementação, a feature precisa estar definida.
>
> Você pode:
> 1. Rodar `/product-plan [nome da feature]` para criar a especificação de produto primeiro.
> 2. Me fornecer manualmente a regra de negócio dessa feature aqui mesmo, e eu uso isso como base para o planejamento.
>
> Qual prefere?"

If the user provides the business rules manually, accept them and proceed.
Do not proceed without any business context.

---

## Step 2 — Ask: frontend or backend?

After confirming the feature context, ask:

> "Você vai trabalhar no **frontend** ou no **backend** dessa feature?"

Wait for the explicit answer before proceeding. Do not assume.

Once the answer is received, activate the boundary rules for that context
immediately. These rules are non-negotiable.

---

## Step 3 — Load the code rules

Read dynamically. Do not hardcode filenames. Scan the entire folder and read
every `.md` file present.

**If frontend was selected:**

Read ALL `.md` files inside:
```
.agents/skills/societiza-feature-plan/references/frontend-code-rules/
```

If this folder is empty or does not exist, stop and respond:

> "A pasta `references/frontend-code-rules/` está vazia ou não existe ainda.
> As regras de código do frontend precisam ser adicionadas antes de continuar.
> Adicione os arquivos `.md` de regras nessa pasta e tente novamente."

Additionally, if this feature involves integrating something already built on
the backend, check:
```
docs/integration/[feature-name].md
```
If it exists, read it fully. This is the backend contract the frontend must
follow. If it does not exist, note this as a gap and warn that the integration
contract is missing.

**If backend was selected:**

Read ALL `.md` files inside:
```
.agents/skills/societiza-feature-plan/references/backend-code-rules/
```

If this folder is empty or does not exist, stop and respond:

> "A pasta `references/backend-code-rules/` está vazia ou não existe ainda.
> As regras de código do backend precisam ser adicionadas antes de continuar.
> Adicione os arquivos `.md` de regras nessa pasta e tente novamente."

Additionally, check:
```
docs/integration/[feature-name].md
```
If a contract exists for this feature, read it. It defines what the backend
must publish for the frontend to integrate later.

---

## Step 4 — Enforce boundary rules

These rules activate immediately when the user answers Step 2. They are hard.

### Backend selected — Frontend is BLOCKED

If the user selected backend:

- Do not write, suggest, or reference any frontend code.
- Do not propose frontend components, hooks, schemas, or UI patterns.
- Do not reference any frontend file or path.
- Frontend does not exist in this conversation. Zero. No exceptions.

Any mention of frontend code means a boundary violation. Stay in backend territory.

### Frontend selected — Backend is BLOCKED by default

If the user selected frontend:

- Do not write, suggest, or implement any backend code.
- Do not propose endpoint changes, database changes, or domain model changes.
- Backend code is blocked until explicit user permission is granted.

**The only exception:**
During the planning process, if you identify that a specific backend improvement
would meaningfully enhance the user experience (not just complete the feature
technically), you may ask the user ONCE:

> "Identifiquei que [descrição precisa da melhoria de backend] poderia melhorar
> a experiência do usuário em [situação específica]. Quer que eu inclua isso
> no plano?"

You may only ask this if:
1. The backend change is genuinely about user experience improvement, not just
   technical completeness.
2. You have not already asked for backend permission in this session.
3. The frontend work can be completed without this backend change.

If the user says no: backend remains blocked. Accept it.
If the user says yes: you may propose the backend change, clearly marked as a
separate section of the plan with its own boundary.

---

## Step 5 — Plan the implementation

Based on the feature context (Step 1) and the code rules loaded (Step 3):

### Technical integration map

Identify existing assets in the codebase that are relevant to this feature:
- For frontend: existing schemas, query hooks, mutation hooks, HTTP services,
  reusable components, path aliases
- For backend: existing modules, aggregates, commands, queries, domain events,
  repositories, services

Do not create new structures when existing ones can be extended.

### Files to create or modify

List every file with its responsibility. For each file:
- Path
- What it contains
- Why it is necessary

### Implementation order

List steps in dependency order. For each step:
- What is created or modified
- Why this order (which dependency makes this the right sequence)
- What validates that this step is complete before moving to the next

### End-to-end verification

How to confirm the feature works correctly after implementation:
- User flow from start to completion
- For frontend: mutations that must invalidate queries, expected success and
  error toasts, build validation (`npm run build` passing)
- For backend: commands that produce the expected domain events, queries that
  return correct projections, endpoint contracts that match what was documented

---

## Planning guardrails

- Do not invent business rules. Only use what was found in Step 1.
- If a dependency does not exist yet (e.g., a notifications module that was not
  built), name it explicitly and propose the correct build order.
- If there is a gap between the product spec and what is technically possible
  today, name the gap instead of hiding it.
- For frontend: types must come from `z.infer<typeof schema>`. Never declared
  separately. Mutations always invalidate queries and always show both success
  and error toasts.
- Commits follow Conventional Commits with scope matching the feature name:
  `feat(feature-name):`, `fix(feature-name):`, `refactor(feature-name):`.
