# Relatório 2 — Relacionamento entre Prioridades, Planejamento, Future Features e Pendências

**Data:** 2026-04-06
**Escopo:** Como os documentos de prioridade, planejamento e features futuras se conectam, onde há coerência e onde há lacunas.

---

## 1. Os documentos envolvidos

| Documento | Função | Granularidade |
|---|---|---|
| `product/product-priorities.md` | Define a semântica de P1-P5 | Apenas conceitual — sem features nomeadas, sem status, sem datas |
| `planning/feature-map.md` | Mapeia features existentes e futuras, dependências e dados compartilhados | Estrutural — mostra o que depende do quê |
| `planning/future-features.md` | Descreve cada feature futura em prosa | Narrativo — explica o "porquê" e "como" de cada feature |
| `planning/dashboard.md` | Detalha a próxima feature mandatória | Específico — visão, dados, dependências, estado |
| `business-context.md` §5 | Tabela resumida de prioridades com status | Operacional — único lugar com status por prioridade |

---

## 2. Como deveriam se conectar (modelo ideal)

```
product-priorities.md          Define o que P1-P5 significam
        │
        ▼
business-context.md §5         Associa cada P a features concretas + status
        │
        ├──▶ feature-map.md    Mostra dependências entre features
        │
        └──▶ future-features.md   Detalha cada feature futura
                │
                └──▶ dashboard.md   Documento dedicado da próxima feature
```

O fluxo esperado: `product-priorities.md` define a escala. `business-context.md` atribui features a cada nível. `feature-map.md` mostra como essas features se relacionam tecnicamente. `future-features.md` explica cada uma em profundidade. Documentos dedicados (como `dashboard.md`) existem para features que precisam de detalhamento extra.

---

## 3. Como realmente se conectam hoje (estado atual)

### 3.1 product-priorities.md é genérico demais

O documento define P1-P5, mas não lista quais features específicas pertencem a cada nível. Exemplo:

- **P1** diz "melhorar societário, workflow-template, consistência..." — mas quais melhorias? Quais já foram feitas? Quais restam?
- **P4** diz "importação, exportação, MCP Server, IA assistiva" — são quatro features distintas agrupadas sem distinção de prioridade entre elas.

**Problema:** sem features nomeadas e com status, o documento funciona como filosofia de priorização, não como roadmap utilizável.

### 3.2 business-context.md §5 é o único lugar com status, mas é superficial

A tabela na §5 é:

| Prioridade | Feature | Status |
|---|---|---|
| P1 | Fortalecer o societário | Em andamento |
| P2 | Dashboard operacional | Não iniciado |
| P3-P5 | (várias) | Futuro |

Isso é insuficiente. "Fortalecer o societário" é um objetivo, não uma feature. "Em andamento" não diz o que foi feito e o que falta. Não há breakdown.

### 3.3 feature-map.md é estruturalmente sólido mas desconectado das prioridades

O `feature-map.md` é o documento mais bem estruturado do conjunto — tem tabela de status, diagrama de dependências, dados compartilhados e pontos de acoplamento. Mas:

- Não referencia P1-P5 em nenhum momento.
- Não indica qual feature é MVP e qual é pós-MVP.
- Não diferencia entre "futuro próximo" e "futuro distante".

**Resultado:** o leitor sabe o que depende do quê, mas não sabe em que ordem as coisas devem ser feitas.

### 3.4 future-features.md é narrativo sem priorização

Cada feature futura tem uma descrição rica e bem contextualizada. Mas:

- Não há indicação de prioridade (P1-P5) em nenhuma feature.
- Não há indicação de dependências (quem depende de quem).
- Não há indicação de complexidade estimada ou pré-requisitos técnicos.
- A ordem das seções no documento não segue a ordem de prioridade.

**Resultado:** o leitor entende o "porquê" de cada feature, mas não sabe por onde começar.

### 3.5 dashboard.md está isolado

O `dashboard.md` é o único documento dedicado a uma feature futura específica. Ele está bem escrito, mas:

- Não referencia `product-priorities.md` (é P2, mas não diz isso).
- Não referencia `future-features.md` (o dashboard não aparece lá).
- Não tem indicação de dependência explícita com P1 (o dashboard depende de P1 estar minimamente consolidado).

---

## 4. Mapa de lacunas entre os documentos

| Lacuna | Onde deveria estar | Impacto |
|---|---|---|
| Features de P1 não estão listadas individualmente | `product-priorities.md` ou `business-context.md` §5 | Impossível rastrear progresso de P1 |
| Nenhum documento associa feature → prioridade P1-P5 | Deveria estar em `feature-map.md` ou `product-priorities.md` | Priorização existe como conceito mas não como atribuição |
| Dashboard não aparece em `future-features.md` | `future-features.md` | Inconsistência — é a feature futura mais importante mas não está no documento de features futuras |
| `future-features.md` não indica dependências | `future-features.md` ou cross-link para `feature-map.md` | Leitor precisa cruzar dois documentos manualmente |
| Nenhum documento marca o que é MVP vs. pós-MVP | Deveria estar em `feature-map.md` ou novo doc | Impossível saber o escopo mínimo de lançamento |
| Freemium não tem documento de planejamento | Deveria existir em `planning/` | Feature que impacta auth, onboarding e modelo comercial sem registro |
| P1 "em andamento" sem breakdown | `business-context.md` §5 ou doc dedicado | Status inutilizável para tracking |

---

## 5. Redundâncias e conflitos

### Redundância: descrição do produto

A descrição "o que é a Societiza" aparece quase idêntica em `business-context.md` §1 e `product-vision.md` §1. Não é um problema grave, mas se um for editado sem o outro, cria divergência.

**Sugestão:** `business-context.md` deveria resumir em 2-3 frases e apontar para `product-vision.md` como fonte canônica, sem repetir os parágrafos completos.

### Conflito potencial: order de features futuras

`feature-map.md` lista as features futuras com dependências técnicas claras (ex: portal-do-cliente depende de notifications). `future-features.md` descreve as mesmas features mas sem indicar essas dependências. Se alguém ler apenas `future-features.md`, pode achar que o portal do cliente pode ser feito a qualquer momento.

---

## 6. Recomendações de reestruturação

### 6.1 Adicionar coluna de prioridade ao feature-map.md

Na tabela "Status das features", adicionar coluna `Prioridade` com P1-P5:

```
| Feature | Status | Prioridade | Descrição curta |
```

Isso conecta o mapa de features ao sistema de prioridades sem duplicar documentação.

### 6.2 Expandir product-priorities.md com features concretas

Cada nível P1-P5 deveria listar as features/melhorias específicas que o compõem, com status individual. Exemplo para P1:

```
## P1 — Fortalecer o núcleo operacional

| Melhoria | Status |
|---|---|
| Campos configuráveis por etapa (StepFields) | Entregue |
| Edição de template ativo com controle de impacto | Não iniciado |
| Proteção de endpoints (auth middleware) | Não iniciado |
| ... | ... |
```

### 6.3 Incluir dashboard em future-features.md

O dashboard é a feature futura mais importante e não aparece no documento de features futuras. Deveria ter uma seção própria lá, mesmo que com ponteiro para `dashboard.md` como documento detalhado.

### 6.4 Adicionar dependências em future-features.md

Cada seção de `future-features.md` deveria ter um bloco padronizado:

```
**Depende de:** workflow, notifications
**Habilita:** portal-do-cliente
**Prioridade:** P3
**Escopo:** MVP | pós-MVP
```

### 6.5 Criar documento de escopo MVP

Hoje nenhum documento responde à pergunta "o que precisa estar pronto para lançar". Deveria existir um `planning/mvp-scope.md` que liste explicitamente o que entra e o que fica de fora.