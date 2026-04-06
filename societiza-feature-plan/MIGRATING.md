# Migracao Pendente — societiza-feature-plan

Esta skill pertence aos repos de execucao, nao ao orquestrador.

## O que migra para onde

### Frontend (web.societiza)
- Esta skill inteira (`societiza-feature-plan/`)
- As 15 frontend-code-rules em `references/frontend-code-rules/`
- Destino: `web.societiza/.claude/skills/feature-plan/`

### Backend (api.societiza)
- Backend-code-rules ja existem em `api.societiza/.claude/skills/architecture/references/`
- O api.societiza pode criar sua propria variante de feature-plan se necessario

## Quando migrar

A migracao acontece quando cada repo criar sua estrutura `.claude/`:
- web.societiza: criar `.claude/`, migrar `.agents/skills/` -> `.claude/skills/`, absorver esta skill
- api.societiza: ja tem `.claude/`, avaliar se precisa de feature-plan proprio

## Ate la

Esta skill permanece aqui como referencia. Os 15 arquivos de frontend-code-rules
estao intactos e podem ser consultados.

## Apos migracao

Remover `societiza-feature-plan/` e `societiza-product-plan/` deste repo.
A skill de produto ja foi migrada para `.claude/skills/feature-design/`.
