# Societiza Skills — Hub de Orquestração

Este repo não contém código. Ele recebe intenções de produto, produz specs
aprovadas por humano e mapas de impacto que os repos de execução consomem.

Repos de execução: `api.societiza` (.NET 9) · `web.societiza` (Next.js)

---

## Estrutura

```
.claude/
  agents/product-thinking.md   <- agente principal
  skills/feature-design/       <- skill de planejamento de produto
  commands/                    <- to-engineering, impact-map
  rules/                       <- spec-first, cross-repo, integration

docs/
  business-context.md          <- índice de contexto (leitura obrigatória)
  product/                     <- visão, ICP, valores, prioridades
  context/                     <- societário, accountancy, identity
  planning/                    <- dashboard, features futuras, feature-map

specs/                         <- specs aprovadas por domínio
  [domínio]-[conceito]/        <- ex: societario-rastreamento/
    spec.md                    <- spec de produto
    impact-map.md              <- mapa de impacto cross-repo

templates/                     <- modelos canônicos
  specification.md             <- template de spec de produto
  impact-map.md                <- template de mapa de impacto
  integration.md               <- template de contrato backend → frontend
```

---

## Onde encontrar o quê

| Tarefa | Arquivo |
|--------|---------|
| Iniciar sessão de produto | `.claude/agents/product-thinking.md` |
| Planejar nova feature | `.claude/skills/feature-design/SKILL.md` |
| Contexto geral do produto | `docs/business-context.md` |
| Visão, ICP, valores | `docs/product/product-vision.md` |
| Semântica de prioridades | `docs/product/product-priorities.md` |
| Workflow societário | `docs/context/societario-context.md` |
| Accountancy (unidade multi-tenant) | `docs/context/accountancy-context.md` |
| Identity (auth + convites) | `docs/context/identity-context.md` |
| Próxima feature obrigatória | `docs/planning/dashboard.md` |
| Features futuras | `docs/planning/future-features.md` |
| Dependências entre features | `docs/planning/feature-map.md` |
| Mapear impacto cross-repo | `.claude/commands/impact-map.md` |
| Workflow feature → engenharia | `.claude/commands/to-engineering.md` |
| Specs aprovadas | `specs/[domínio]-[conceito]/spec.md` |
| Mapas de impacto aprovados | `specs/[domínio]-[conceito]/impact-map.md` |
| Templates de documentação | `templates/` |

---

## Regras ativas

Carregadas automaticamente de `.claude/rules/`:

- **spec-first** — nenhuma feature implementada sem spec aprovada por humano
- **cross-repo** — este repo produz specs e mapas de impacto, não código
- **integration** — guardrails para evitar invenção de features ou mascaramento de lacunas