# Domínio de Workflow — Regras Específicas da Societiza

## Objetivo

Este documento registra as regras de domínio mais importantes para planejamento
frontend nas áreas de `workflow` e `workflow-template`.

Ele não substitui documentação de produto nem documentação de integração com
backend. A função aqui é evitar decisões erradas de frontend sobre:

- nomenclatura;
- ordem;
- relação entre template e uso real;
- diferenças entre domínio novo e legado;
- e confusões comuns sobre “formulário”.

---

## Dois domínios conectados, mas diferentes

O projeto lida com dois mundos relacionados:

### `workflow-template`

É o domínio novo de configuração do fluxo.

Ele usa nomenclatura majoritariamente em inglês e modela:

- template;
- step;
- task;
- field;
- ordem;
- draft/publicação.

### `workflow`

É o domínio operacional legado do board societário.

Ele usa convenções históricas mais heterogêneas, muitas vezes em português, e
modela:

- processos;
- etapas do board;
- tarefas operacionais;
- drawers, filtros e visualização do trabalho em andamento.

### Regra

Frontend não deve “normalizar” os dois mundos na marra. Eles são conectados, mas
não idênticos.

---

## `workflow-template` é configuração

`workflow-template` representa o desenho configurável do fluxo que a contabilidade
quer usar.

A hierarquia principal é:

```txt
WorkflowTemplate
  └── steps
        ├── fields
        └── tasks
              └── fields
```

### Implicações para o frontend

- template builder precisa preservar essa hierarquia;
- field de step e field de task não são a mesma lista;
- ordem de step, task e field importa;
- edição de template não deve ser confundida com execução do processo real.

---

## `workflow` é operação

`workflow` é a superfície operacional onde os processos aparecem no board e o
usuário acompanha o trabalho acontecendo.

O foco aqui é:

- visualizar processos;
- abrir detalhe;
- acompanhar tarefas;
- aplicar filtros;
- entender urgência e estado operacional.

### Regra

O board não deve assumir o mesmo modelo mental do template builder. Ele consome
e representa operação, não configuração estrutural.

---

## Template em uso vs template configurável

Essa distinção é obrigatória:

### Workflow template

- estrutura configurável;
- editável;
- pode estar em draft ou ativo;
- usado para definir o fluxo.

### Workflow em uso

- operação real;
- centrado em processos concretos;
- leitura de andamento e execução;
- não é o mesmo objeto mental da tela de configuração.

### Regra

O frontend deve nomear, documentar e implementar telas deixando essa diferença
clara.

---

## Status de template

No domínio atual de `workflow-template`, o contrato frontend estabilizado usa:

```ts
'Draft' | 'Active' | 'Archived'
```

### Leitura semântica

- `Draft`: estado de rascunho/configuração;
- `Active`: template publicável/em uso como referência ativa;
- `Archived`: template arquivado, sem protagonismo operacional.

### Regra

Ao planejar frontend, usar apenas nomenclaturas de status que o contrato atual da
feature realmente expõe. Não inventar estados intermediários não formalizados.

---

## Ordem (`order`)

`order` é parte central da experiência do template builder.

Ele vale para:

- steps;
- tasks;
- fields de step;
- fields de task.

### Regra semântica

- frontend deve tratar `order` como parte do contrato da entidade;
- a interface deve respeitar ordenação antes de renderizar;
- reorder precisa preservar consistência entre cache, UI e backend;
- UI otimista é desejável quando houver ganho de experiência, com reconciliação
  posterior.

### Regra de produto

A ordenação de fields de step e de fields de task são independentes. O frontend
não deve tratar tudo como uma lista única misturada.

---

## Tipos de field

No contrato estabilizado da feature, `fieldType` hoje trabalha com:

```ts
'Text' | 'Select'
```

### Semântica

- `Text`: valor textual livre;
- `Select`: valor escolhido entre opções.

### Regra

Se o field for `Select`, a UI precisa respeitar a existência de opções coerentes.
O frontend não deve inventar interpretação visual para tipos não formalizados no
contrato atual.

---

## Tipos de task

O contrato atual do frontend em `workflow-template` ainda convive com valores de
tipo modelados como string e opções de tarefa registradas em constantes da
feature. O conjunto hoje usado pela interface inclui:

```ts
'Form' | 'Checklist' | 'Document' | 'Approval'
```

### Regra importante

Se houver divergência entre backend, legado e opções visíveis da UI, o frontend
deve seguir o contrato efetivamente exposto e documentar a discrepância em
integração, não “corrigir” localmente inventando enum novo sem coordenação.

---

## `configuration` em task

No template builder, `configuration` é tratada como string serializada.

### Implicações

- frontend precisa tratar o campo com cuidado;
- parse e stringify precisam respeitar o contrato da feature;
- não assumir estrutura interna única se ela não estiver formalizada na doc de
  integração.

### Regra

O schema do frontend pode manter `configuration` como string quando essa for a
forma estável do contrato. Parse local só deve acontecer em pontos conscientes da
interface.

---

## Diferença entre campos configuráveis e feature futura de formulário

Isto é crítico para o planejamento:

Hoje, `workflow-template` possui **fields configuráveis** dentro de steps e
tasks. Isso não significa que o produto já tenha uma feature formal e consolidada
de formulário de abertura de empresa.

### Regra

Ao documentar ou planejar frontend:

- não tratar os fields do template builder como se fossem a futura feature de
  formulário nativo do produto;
- descrever esses fields como parte do workflow configurável atual;
- tratar a futura feature de formulário como visão de produto separada.

---

## Diferença de linguagem entre domínios

`workflow-template` usa inglês com contratos mais modernos.

`workflow` legado usa contrato mais heterogêneo e histórico.

### Regra

Não tentar padronizar superficialmente a semântica só renomeando campos no
frontend. O schema deve refletir o contrato real de cada domínio.

---

## Board e template devem permanecer conectados

Mesmo sendo domínios diferentes, eles se complementam:

- template define estrutura;
- workflow mostra a operação acontecendo;
- dashboard futuramente deve ler a operação e levar de volta para ela.

### Regra de produto refletida no frontend

O planejamento de UI e navegação deve evitar tratar essas áreas como módulos
isolados. Elas se relacionam, mas sem colapsar uma dentro da outra.

---

## O que o frontend não deve assumir sem documentação adicional

- que todo status legado já está normalizado;
- que todo valor de tipo está rigidamente fechado no backend;
- que board e template compartilham shape interno;
- que `workflow` e `workflow-template` podem ser unificados só no frontend;
- que fields atuais equivalem ao futuro módulo de formulário.

---

## Resumo operacional

- `workflow-template` é configuração;
- `workflow` é operação;
- template em uso e template configurável não são a mesma coisa;
- `order` é parte central do contrato;
- fields de step e de task são escopos diferentes;
- campos configuráveis atuais não equivalem à futura feature de formulário;
- frontend deve respeitar as diferenças reais entre domínio novo e legado.
