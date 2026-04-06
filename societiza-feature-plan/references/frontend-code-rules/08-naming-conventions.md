# Convenções de Nomenclatura — Societiza

## Objetivo

Nomear bem é parte da arquitetura. Neste projeto, nomenclatura deve comunicar:

- o papel do arquivo;
- a camada em que ele vive;
- o domínio da feature;
- e se aquilo é UI, hook, service, type, schema ou documentação.

Este documento fixa o padrão ideal para novas implementações e revisões.

---

## Convenção escolhida

### Pastas

Sempre `kebab-case`.

Exemplos:

```txt
src/features/workflow-template/
src/features/accountancy/
src/features/workflow/contexts/ui/
```

### Hooks e services

Arquivos em `camelCase`.

Exemplos:

```txt
useAccountancyQueries.ts
useAccountancyMutations.ts
useTemplatePublishFlow.ts
accountancy.service.ts
template.service.ts
```

### Componentes React exportados

Componentes exportados em `PascalCase`.

Exemplos:

```txt
AccountancyFormDialog.tsx
TemplateHeader.tsx
ProcessCard.tsx
```

### Barrels

Sempre `index.ts`.

---

## Features

Feature folder deve refletir o domínio real.

Exemplos corretos:

```txt
accountancy
auth
dashboard
workflow
workflow-template
```

Evitar:

- siglas obscuras;
- nomes técnicos em vez de nomes de domínio;
- nomes genéricos como `utils-feature`;
- pastas camelCase ou PascalCase em `src/features`.

---

## Arquivos por camada

| Camada | Convenção | Exemplo |
|---|---|---|
| Schema | `{entity}.schema.ts` | `accountancy.schema.ts` |
| Service | `{entity}.service.ts` | `accountancy.service.ts` |
| Types | `{entity}.types.ts` | `template.types.ts` |
| Query hook | `use{Name}Queries.ts` | `useAccountancyQueries.ts` |
| Mutation hook | `use{Name}Mutations.ts` | `useTemplateMutations.ts` |
| Form hook | `use{Name}Form.ts` | `useAccountancyForm.ts` |
| Component hook | `use{Name}.ts` | `useTemplateColumn.ts` |
| Query options | `query-options.ts` | `query-options.ts` |
| Visual form | `{Name}Form.tsx` | `AccountancyForm.tsx` |
| Componente | `{Name}.tsx` | `TemplateBuilder.tsx` |
| Constants | `{feature}.constants.ts` | `template.constants.ts` |
| README | `README.md` | `README.md` |

### Observação

`query-options.ts` é um nome fixo, com hífen, porque representa um arquivo
estrutural muito específico do contrato da feature.

---

## Convenção de nomes para hooks

### Hooks de query

Devem comunicar leitura e escopo.

Exemplos:

```ts
useAccountancyQueries
useWorkflowQueries
useTemplateQueries
```

Se a feature optar por expor sub-hooks internos:

```ts
const { useList, useDetail } = useTemplateQueries();
```

### Hooks de mutation

Devem comunicar escrita e agrupamento por domínio.

Exemplos:

```ts
useAccountancyMutations
useTemplateMutations
useWorkflowMutations
```

### Hooks de formulário

Devem comunicar o formulário dono daquela lógica.

Exemplos:

```ts
useAccountancyForm
useProcessForm
useEditProcessForm
```

### Hooks de orchestration de componente

Devem comunicar o componente ou fluxo orquestrado.

Exemplos:

```ts
useTemplateColumn
useTemplatePublishFlow
useBoardCard
```

---

## Convenção de nomes para functions

### Services de leitura

Prefixo `fetch`.

```ts
fetchAccountancies()
fetchAccountancyById(id)
fetchWorkflowTemplateById(id)
```

### Services de escrita

Prefixos de ação explícita.

```ts
createAccountancy(data)
updateAccountancy(id, data)
deleteAccountancy(id)
publishWorkflowTemplate(id)
archiveWorkflowTemplate(id)
```

### Handlers locais

Prefixo `handle`.

```ts
handleSubmit()
handleDelete()
handlePublish()
handleMoveStep()
```

### Helpers puros

Nome pelo efeito, sem prefixo React.

```ts
normalizeCnpj()
getNextIndexedLabel()
buildTemplateFieldPayload()
```

---

## Convenção de nomes para tipos

### Entidades

`PascalCase`, singular.

```ts
type Accountancy = ...
type WorkflowTemplateDetail = ...
type WorkflowTemplateStep = ...
```

### DTOs

`PascalCase` + sufixo `DTO`.

```ts
type CreateAccountancyDTO = ...
type UpdateTemplateDTO = ...
type UpdateStepDTO = ...
```

### Props

`PascalCase` + `Props`.

```ts
interface TemplateHeaderProps {}
interface ProcessCardProps {}
```

### Context value

`PascalCase` + `ContextValue`.

```ts
interface CorporateUIContextValue {}
```

---

## Query keys

Nome da constante:

```ts
accountancyQueryKeys
workflowQueryKeys
workflowTemplateQueryKeys
```

Nome da estrutura:

```ts
all
lists
details
detail
filtered
```

Isso mantém previsibilidade entre features.

---

## Constantes

Constantes exportadas devem usar `SCREAMING_SNAKE_CASE`.

Exemplos:

```ts
TASK_TYPE_OPTIONS
TEMPLATE_STATUS_LABELS
TEMPLATE_STATUS_COLORS
MAX_STEPS_PER_TEMPLATE
```

### Regra

- constante exportada: `SCREAMING_SNAKE_CASE`;
- helper puro: `camelCase`;
- enum value/string literal: respeitar o contrato do backend.

---

## Branches e scopes de commit

O padrão de branch continua em `kebab-case`.

Exemplos:

```txt
feat/dashboard-overview
fix/workflow-template-publish
refactor/accountancy-hooks
docs/frontend-code-rules
```

Scopes de commit também em `kebab-case`:

```txt
feat(accountancy): adicionar mutation de edição
fix(workflow-template): corrigir ordem de fields
docs(frontend-code-rules): revisar convenções de arquitetura
```

---

## O que evitar

- misturar kebab-case, camelCase e PascalCase sem critério;
- componente exportado com nome genérico demais;
- hook com nome que não diz se lê, escreve ou orquestra;
- DTO sem sufixo que o diferencie da entidade;
- `utils.ts` como nome de arquivo sem semântica clara;
- abreviações opacas em domínio de negócio.

---

## Resumo operacional

- pastas em `kebab-case`;
- hooks e services em `camelCase`;
- componentes exportados em `PascalCase`;
- barrels em `index.ts`;
- tipos de entidade em `PascalCase`;
- DTOs com sufixo `DTO`;
- constantes exportadas em `SCREAMING_SNAKE_CASE`.
