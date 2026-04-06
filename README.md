# Societiza Skills — Hub de Produto e Orquestração

Este repositório é o hub central de conhecimento de produto e orquestração do
ecossistema Societiza. Ele não contém código de aplicação. Seu papel é:

- Definir o que cada feature é e por que ela existe
- Produzir specs de produto aprovadas pelo humano
- Gerar contratos de API que os repos de execução consomem
- Mapear impacto cross-repo antes de implementar

---

## Ecossistema

```
skills/              <- este repo (orquestrador)
api.societiza/       <- backend (.NET 9)
web.societiza/       <- frontend (Next.js)
```

A interação humana acontece aqui — aprovação de spec antes de qualquer
implementação começar nos repos de execução.

---

## Estrutura

```
.claude/                  <- configuração Claude Code (Profile C: Orquestração)
  rules/                  <- restrições aplicadas em toda sessão
  skills/feature-design/  <- skill de planejamento de produto
  commands/               <- workflows reutilizáveis (new-feature, impact-map)

docs/                     <- documentação de produto decomposta
  product/                <- visão, valores, prioridades
  context/                <- societário, accountancy, auth
  planning/               <- dashboard, features futuras, mapa de features
  integration/            <- template de doc backend -> frontend

contracts/                <- contratos OpenAPI versionados
  api/                    <- consumidos por ambos os repos

features/                 <- specs de features aprovadas
  specs/                  <- geradas pela skill feature-design

  business-context.md    <- índice de contexto de produto
```

---

## Como usar

### Planejar uma feature nova

Descreva a ideia em linguagem natural. A skill `feature-design` será ativada
automaticamente. Ela lê o contexto do produto, aplica o framework de pensamento
de produto e gera uma spec em `features/specs/`.

### Workflow completo

Ver `.claude/commands/new-feature.md`:
1. Contexto -> spec -> aprovação -> contrato -> impacto -> distribuição

### Mapear impacto cross-repo

Ver `.claude/commands/impact-map.md`:
Identifica o que muda em cada repo antes de implementar.

---

## Migração pendente

`societiza-feature-plan/` na raiz está pendente de migração para os repos de
execução. Ver `societiza-feature-plan/MIGRATING.md` para detalhes.

`societiza-product-plan/` na raiz é a versão anterior da skill que agora vive
em `.claude/skills/feature-design/`. Será removida após validação.
