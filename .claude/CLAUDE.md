# Societiza Skills — Hub de Orquestração

Repositório de conhecimento e orquestração do ecossistema Societiza.
Este repo não contém código de aplicação. Ele produz specs de features,
specs e mapas de impacto que os repos de execução consomem.

---

## Papel no ecossistema

```
skills/              <- este repo (orquestrador)
  recebe intenção de feature, produz specs, mapeia impacto

api.societiza/       <- execução backend
  lê specs e mapas de impacto, implementa backend .NET

web.societiza/       <- execução frontend
  lê specs e mapas de impacto, implementa frontend Next.js
```

A interação humana acontece aqui — aprovação da spec antes de qualquer
implementação começar nos repos de execução.

---

## Estrutura do repositório

```
.claude/
  CLAUDE.md            <- este arquivo
  settings.json        <- permissões
  rules/               <- restrições aplicadas a cada sessão
  skills/              <- conhecimento especializado carregado sob demanda
  commands/            <- workflows reutilizáveis
  agents/product-thinking.md  <- agente cérebro de produto

docs/
  product/             <- visão, valores, prioridades
  context/             <- societário, escritório, auth
  planning/            <- dashboard, features futuras, mapa de features
  integration/         <- template de integração backend-to-frontend

features/
  specs/               <- especificações de features aprovadas

  business-context.md  <- índice de contexto do produto
```

---

## Módulos de documentação existentes

| Caminho | Conteúdo |
|---------|----------|
| `docs/product/product-vision.md` | Identidade, ICP, valores, diretrizes de features |
| `docs/product/product-priorities.md` | Semântica de prioridades P1-P5 |
| `docs/context/societario-context.md` | Workflow, template, board, campos |
| `docs/context/escritorio-context.md` | Escritório como unidade organizacional multi-tenant. Substitui accountancy. |
| `docs/context/accountancy-context.md` | DEPRECIADO — ver escritorio-context.md |
| `features/specs/escritorio-e-planos.md` | Spec aprovada do Escritório e Planos (P0 ativo) |
| `docs/context/authentication-context.md` | Estado e lacunas de autenticação |
| `docs/planning/dashboard.md` | Próxima feature obrigatória |
| `docs/planning/future-features.md` | Todas as features futuras planejadas |
| `docs/planning/feature-map.md` | Dependências e acoplamento entre features |

---

## Onde encontrar o quê

| Tarefa | Leia primeiro |
|--------|---------------|
| Iniciar qualquer sessão de produto | `.claude/agents/product-thinking.md` (agente product-thinking) |
| Planejar uma nova feature | `.claude/skills/feature-design/SKILL.md` |
| Entender contexto do produto | `docs/business-context.md` (índice) |
| Verificar dependências entre features | `docs/planning/feature-map.md` |
| Mapear impacto cross-repo | `.claude/commands/impact-map.md` |
| Workflow completo de feature para engenharia | `.claude/commands/to-engineering.md` |

Regras ativas a cada sessão: `.claude/rules/`

---

## Notas de migração

`societiza-feature-plan/` na raiz do repo está pendente de migração para os
repos de execução. Veja `societiza-feature-plan/MIGRATING.md` para detalhes.

`societiza-product-plan/` foi removido — seu conteúdo agora vive em
`.claude/skills/feature-design/`.