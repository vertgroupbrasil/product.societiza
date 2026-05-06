# Integration — WorkflowTemplate (Societário)

## 1. Identificação da feature

- **Nome da feature**: WorkflowTemplate — Modelos de processo societário
- **Módulo backend responsável**: `WorkflowTemplate` (`src/Modules/WorkflowTemplate`)
- **Status atual**: `existente`
- **Data da última atualização**: 2026-05-06
- **Responsável técnico**: Gabriel Carvalho (gabriellccarvalho.dev@gmail.com)

A feature está implementada e exposta em `/api/v1/workflow-template/...`. Todos
os endpoints CRUD para template, step, task, step-field e task-field estão
publicados, autenticados e com policy de SystemAdmin para operações de escrita.
A leitura é aberta a qualquer usuário autenticado (com filtragem por role na
listagem).

---

## 2. Objetivo de negócio da feature

O `WorkflowTemplate` é o módulo onde o `SystemAdmin` da Societiza cria e
mantém os modelos canônicos de processo societário. Cada template descreve, de
forma abstrata, quais etapas (steps) compõem o processo, quais tarefas (tasks)
existem em cada etapa e quais campos (fields) precisam ser preenchidos ao longo
da execução.

Antes da feature, não havia estrutura central que padronizasse processos entre
contabilidades. Com ela, o `SystemAdmin` configura o template uma vez e o molde
fica disponível para que qualquer contabilidade execute o processo real do
cliente — habilitando a feature futura `WorkflowProcess`.

A feature entra na jornada de setup e manutenção da plataforma: lançamento de
novos tipos de processo, atualização de processos existentes (via versionamento
em outra feature, `societario-template-versioning`) e arquivamento de modelos
obsoletos. Os contadores (`AccountancyAdmin` e `AccountancyEmployee`) aparecem
como consumidores de leitura, com visibilidade restrita a templates `Active`.

A feature se conecta diretamente com:

- `Identity` — para autenticação e role do usuário corrente.
- `WorkflowProcess` (módulo separado) — consome a estrutura snapshot do
  template ativo via interface anti-corrupção `IWorkflowTemplateDataService`.
- `societario-template-versioning` (feature futura) — depende deste módulo
  para evoluir templates ativos sem interromper processos em andamento.

---

## 3. Escopo atual versus escopo futuro

**O que já está implementado no backend hoje:**

- CRUD completo de template (create, update, activate, archive, list, detail).
- CRUD completo de step (add, update, remove com cascata).
- CRUD completo de task (add, update, remove com cascata).
- CRUD completo de step field (add, update, remove).
- CRUD completo de task field (add, update, remove).
- Validações FluentValidation para entrada de cada operação.
- Regras de negócio: `TemplateNotEditable`, `ActivationRequiresAtLeastOneStep`,
  `ActivationRequiresAllStepsToHaveTask`, unicidade de títulos/labels com
  trim e case-insensitive, insert-shift de order, cascata de remoção.
- Filtragem por role na listagem (`includeDraft`/`includeArchive` aplicáveis
  apenas para `SystemAdmin`).
- Paginação na listagem (`pageNumber`, `pageSize`).
- Event sourcing completo: snapshots e store events em PostgreSQL (schema
  `workflow_template`).
- Projections MongoDB para leitura: `WorkflowTemplate`, `WorkflowTemplateStep`,
  `WorkflowTemplateStepTask`, `TemplateStepField`, `TemplateStepTaskField`.
- Autorização: policy `CanManageWorkflowTemplates` exige `SystemAdmin` para
  todas as escritas; leituras aceitam qualquer usuário autenticado.

**O que ainda não está implementado:**

- Não há endpoint para listar `templates` por tipo de processo nem por domínio.
- Não há mecanismo público de "criar nova versão a partir de template ativo"
  (parte da spec `societario-template-versioning`).
- Não há limite ou restrição de quantos templates ativos podem coexistir.

**O que existe no domínio mas não está exposto por endpoint:**

- O agregado `WorkflowTemplate` expõe métodos `Activate()` e `Archive()`, mas
  apenas `Active` e `Archived` aparecem como ações via API. Não há endpoint
  para reverter (`Active → Draft`, `Archived → Active`) — alinhado com a regra
  de "transições não revertem" descrita na logic.
- O value object `TemplateVersion` existe e suporta parsing/incremento, mas
  não há comando de incremento exposto. Versão é fixa em `v1.0` na criação.

**O que é apenas visão futura de produto:**

- Versionamento de templates (feature `societario-template-versioning`).
- Personalização de templates por contabilidade (não previsto).
- Execução automática de tasks `Automatic` (parte de `WorkflowProcess`).
- Fluxo de aprovação de tasks `Approval` (parte de `WorkflowProcess`).

**O que ainda está em discussão:**

- Nenhum item identificado no código com TODO/comentário pendente.

---

## 4. Contexto de produto e semântica da feature

O usuário (`SystemAdmin`) percebe a feature como um editor de templates de
processo: cria um template em rascunho, monta as etapas e tarefas livremente,
e publica quando estiver pronto. A partir da publicação, o template vira o
molde compartilhado para todas as contabilidades.

Sucesso significa: o template é criado em `Draft`, recebe steps com tasks,
recebe fields onde fizer sentido e é ativado sem erro de validação. Ao ser
ativado, o template aparece para os contadores em `GetAllWorkflowTemplates`
sem necessidade de query param adicional.

Erro funcional acontece quando:

- A ativação é tentada sem ao menos uma step (`ActivationRequiresAtLeastOneStep`).
- A ativação é tentada com alguma step sem task (`ActivationRequiresAllStepsToHaveTask`).
- Uma escrita em step/task/field é tentada em template não-Draft
  (`TemplateNotEditable`).
- Um título/label duplicado é submetido (case-insensitive com trim).

Estados e transições importam fortemente: o frontend precisa habilitar/desabilitar
ações conforme `status` do template:

- `Draft`: tudo é editável; ações disponíveis = update template, add/update/remove
  steps/tasks/fields, activate.
- `Active`: imutável para steps/tasks/fields; ações disponíveis = update nome
  e descrição, archive.
- `Archived`: terminal; nenhuma ação disponível.

Para `AccountancyAdmin` e `AccountancyEmployee`, a feature é apenas leitura de
templates `Active`. Templates não-Active não devem aparecer na lista nem ser
acessíveis por id direto (gap — ver seção 16).

---

## 5. Dependências e relações com outras features

**Depende de:**

- `Identity` — autenticação JWT, role do usuário (`SystemAdmin`,
  `AccountancyAdmin`, `AccountancyEmployee`) e a policy `CanManageWorkflowTemplates`.
- `Core` — primitivas de domínio (`AggregateRoot`, `Entity`), event store,
  unit of work, projection abstrata, `Result`/erros padronizados.

**Consumido por:**

- `WorkflowProcess` (futuro) — via interface
  `WorkflowProcess.Application.Services.IWorkflowTemplateDataService`. Essa
  interface tem método `GetActiveTemplateSnapshotAsync(templateId, ct)` que
  retorna `(string Version, List<TemplateStepSnapshot> Steps)?`. **A
  implementação concreta dessa interface não existe no código deste módulo
  hoje** — é gap (ver seção 16).

**Posicionamento no produto:**

Feature fundacional do societário. Pré-requisito para tudo na camada de
execução de processos.

---

## 6. Rotas públicas

Todas as rotas estão sob `/api/v{version:apiVersion}/workflow-template`
(equivalente a `/api/v1/workflow-template/...` para o consumo atual).

| Método | Path | Tipo | Finalidade | Params de rota | Pronta para frontend? |
|---|---|---|---|---|---|
| POST | `/api/v1/workflow-template` | command | Criar template em `Draft` | — | sim |
| PUT | `/api/v1/workflow-template/{workflowTemplateId}` | command | Atualizar nome e descrição do template | `workflowTemplateId` | sim |
| PATCH | `/api/v1/workflow-template/{workflowTemplateId}/activate` | command | Ativar template (`Draft → Active`) | `workflowTemplateId` | sim |
| PATCH | `/api/v1/workflow-template/{workflowTemplateId}/archive` | command | Arquivar template (`Active → Archived`) | `workflowTemplateId` | sim |
| GET | `/api/v1/workflow-template` | query | Listar templates paginado | — | sim |
| GET | `/api/v1/workflow-template/{workflowTemplateId}` | query | Detalhe completo do template | `workflowTemplateId` | sim |
| POST | `/api/v1/workflow-template/{workflowTemplateId}/step` | command | Adicionar step | `workflowTemplateId` | sim |
| PUT | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}` | command | Atualizar step | `workflowTemplateId`, `workflowStepId` | sim |
| DELETE | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}` | command | Remover step (cascata em tasks/fields) | `workflowTemplateId`, `workflowStepId` | sim |
| POST | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task` | command | Adicionar task | `workflowTemplateId`, `workflowStepId` | sim |
| PUT | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task/{workflowTaskId}` | command | Atualizar task | `workflowTemplateId`, `workflowStepId`, `workflowTaskId` | sim |
| DELETE | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task/{workflowTaskId}` | command | Remover task (cascata em fields) | `workflowTemplateId`, `workflowStepId`, `workflowTaskId` | sim |
| POST | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/fields` | command | Adicionar step field | `workflowTemplateId`, `workflowStepId` | sim |
| PUT | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/fields/{workflowFieldId}` | command | Atualizar step field | `workflowTemplateId`, `workflowStepId`, `workflowFieldId` | sim |
| DELETE | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/fields/{workflowFieldId}` | command | Remover step field | `workflowTemplateId`, `workflowStepId`, `workflowFieldId` | sim |
| POST | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task/{workflowTaskId}/fields` | command | Adicionar task field | `workflowTemplateId`, `workflowStepId`, `workflowTaskId` | sim |
| PUT | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task/{workflowTaskId}/fields/{workflowFieldId}` | command | Atualizar task field | `workflowTemplateId`, `workflowStepId`, `workflowTaskId`, `workflowFieldId` | sim |
| DELETE | `/api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task/{workflowTaskId}/fields/{workflowFieldId}` | command | Remover task field | `workflowTemplateId`, `workflowStepId`, `workflowTaskId`, `workflowFieldId` | sim |

Todos os endpoints requerem token JWT. As escritas (POST/PUT/PATCH/DELETE em
template, step, task e fields) exigem role `SystemAdmin` via policy
`CanManageWorkflowTemplates`. As leituras (GETs) aceitam qualquer usuário
autenticado.

---

## 7. Contrato de request

### Request — Create template

`POST /api/v1/workflow-template`

Corpo (JSON):

- `name`: string, obrigatório, máx 150 caracteres — nome do template.
- `description`: string, obrigatório, máx 500 caracteres — descrição funcional.

Transformações obrigatórias no frontend: nenhuma.

Validações relevantes (FluentValidation):
- `Name` `NotEmpty` e `MaximumLength(150)`.
- `Description` `NotEmpty` e `MaximumLength(500)`.

### Request — Update template

`PUT /api/v1/workflow-template/{workflowTemplateId}`

Body:

- `name`: string, obrigatório, máx 150 caracteres.
- `description`: string, obrigatório, máx 500 caracteres.

Observação: o handler `UpdateWorkflowTemplateHandler` **não verifica
status** antes de atualizar — ou seja, atualizar nome/descrição é permitido
independente de o template estar `Draft`, `Active` ou `Archived`. Ver seção 16.

### Request — Activate template

`PATCH /api/v1/workflow-template/{workflowTemplateId}/activate`

Body: vazio.

Pré-condições do handler:
- Template precisa existir.
- Status precisa ser `Draft` (caso contrário retorna `InvalidStatusTransition`).
- Template precisa ter ao menos 1 step.
- Cada step precisa ter ao menos 1 task.

### Request — Archive template

`PATCH /api/v1/workflow-template/{workflowTemplateId}/archive`

Body: vazio.

Pré-condições do handler:
- Template precisa existir.
- Status precisa ser `Active` (caso contrário retorna `InvalidStatusTransition`).

Nota: a Description do Swagger do `ArchiveWorkflowTemplateEndpoint` afirma que
"status permitidos de origem: Draft, Active". O código do handler permite
**apenas `Active`**. Ver seção 16.

### Request — List templates

`GET /api/v1/workflow-template`

Query string:

- `includeDraft`: bool, default `false` — só efetivo para `SystemAdmin`.
- `includeArchive`: bool, default `false` — só efetivo para `SystemAdmin`.
- `pageNumber`: int, default `1` — base 1.
- `pageSize`: int, default `10`.

Observação: não há validador FluentValidation registrado para
`GetAllWorkflowTemplatesQuery`. O Swagger menciona `pageNumber >= 1` e
`pageSize entre 1 e 100`, mas a validação prática depende do `Paging` em
`Core.Application.Pagination` — não validado neste módulo.

### Request — Get template by id

`GET /api/v1/workflow-template/{workflowTemplateId}`

Sem body. Sem query string.

### Request — Add step

`POST /api/v1/workflow-template/{workflowTemplateId}/step`

Body:

- `title`: string, obrigatório, máx 150 caracteres.
- `description`: string, obrigatório, máx 500 caracteres.
- `order`: int, obrigatório, > 0.

Pré-condições do handler:
- Template precisa estar em `Draft` (caso contrário retorna `TemplateNotEditable`).
- `title.Trim()` não pode duplicar título de step existente (case-insensitive)
  → `StepTitleAlreadyExists`.
- `order` é `Math.Clamp(request.Order, 1, template.Steps.Count + 1)` antes de
  passar ao agregado. Se já existe step com aquele order, há insert-shift dos
  demais.

### Request — Update step

`PUT /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}`

Body:

- `title`: string, obrigatório, máx 150 caracteres.
- `description`: string, obrigatório, máx 500 caracteres.
- `order`: int, obrigatório, > 0.

Diferença: clamp em update é `Math.Clamp(request.Order, 1, template.Steps.Count)`
(não `Count + 1`).

### Request — Remove step

`DELETE /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}`

Sem body. Cascata: remove step + step fields + tasks + task fields da step.

### Request — Add task

`POST /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task`

Body:

- `title`: string, obrigatório, máx 150 caracteres.
- `description`: string, obrigatório, máx 500 caracteres.
- `type`: string, obrigatório — um de `Manual`, `Automatic`, `Approval`.
- `configuration`: string, **obrigatório** (validador `NotNull`); JSON string;
  o agregado mantém default `"{}"` na entidade, mas o validador rejeita null.
- `order`: int, obrigatório, > 0.
- `isOptional`: bool, obrigatório.

Transformações obrigatórias no frontend: o campo `configuration` é livre e
deve ser uma string JSON válida (estrutura específica por tipo é responsabilidade
da camada de execução `WorkflowProcess`).

### Request — Update task

`PUT /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task/{workflowTaskId}`

Body: idêntico ao de Add task.

### Request — Remove task

`DELETE /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task/{workflowTaskId}`

Sem body. Cascata: remove task + task fields.

### Request — Add step field

`POST /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/fields`

Body:

- `label`: string, obrigatório, máx 150 caracteres.
- `fieldType`: string, obrigatório — um de `Text`, `Boolean`, `Integer`,
  `Select`, `Date`.
- `options`: string nullable. **Obrigatório se `fieldType = Select`**; deve
  ser **null** para os demais.
- `order`: int, obrigatório, > 0.

### Request — Update step field

`PUT /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/fields/{workflowFieldId}`

Body: idêntico ao de Add step field.

### Request — Remove step field

`DELETE /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/fields/{workflowFieldId}`

Sem body.

### Request — Add task field

`POST /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task/{workflowTaskId}/fields`

Body: idêntico ao step field, com regra de `options` por `fieldType`.

### Request — Update task field

`PUT /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task/{workflowTaskId}/fields/{workflowFieldId}`

Body: idêntico ao Add task field.

### Request — Remove task field

`DELETE /api/v1/workflow-template/{workflowTemplateId}/step/{workflowStepId}/task/{workflowTaskId}/fields/{workflowFieldId}`

Sem body.

---

## 8. Contrato de response

### Response — Create template

`200 OK` com `IdentifierResponse`:

- `id`: GUID — id do template criado.

### Response — Update template / Activate / Archive

`200 OK` sem corpo significativo.

### Response — List templates

`200 OK` com `IPagedResult<WorkflowTemplateResponse>`. Cada item:

- `id`: GUID.
- `name`: string.
- `description`: string.
- `version`: string — formato `v{Major}.{Minor}` (ex.: `v1.0`).
- `status`: string — `Draft` | `Active` | `Archived`.
- `createdAt`: ISO-8601 `DateTimeOffset`.

Ordenação fixa do backend: `OrderBy(Name).ThenBy(CreatedAt)` — preservar.

### Response — Get template by id

`200 OK` com `WorkflowTemplateDetailResponse`:

- `id`: GUID.
- `name`: string.
- `description`: string.
- `version`: string.
- `status`: string — `Draft` | `Active` | `Archived`.
- `createdAt`: ISO-8601.
- `updatedAt`: ISO-8601 nullable.
- `steps`: array de `WorkflowTemplateStepDetailResponse`.

Cada `WorkflowTemplateStepDetailResponse`:

- `id`: GUID.
- `title`: string.
- `description`: string.
- `order`: int (1-based, sem garantia de ser sequencial após reordens — apenas
  ordenado crescentemente).
- `fields`: array de `WorkflowTemplateStepFieldDetailResponse`.
- `tasks`: array de `WorkflowTemplateStepTaskDetailResponse`.

Cada `WorkflowTemplateStepFieldDetailResponse`:

- `id`: GUID.
- `label`: string.
- `fieldType`: string — `Text` | `Boolean` | `Integer` | `Select` | `Date`.
- `options`: string nullable. Não nulo se `fieldType = Select`; nulo nos demais.
- `order`: int.

Cada `WorkflowTemplateStepTaskDetailResponse`:

- `id`: GUID.
- `title`: string.
- `description`: string.
- `type`: string — `Manual` | `Automatic` | `Approval`.
- `configuration`: string — JSON string (default `"{}"`).
- `order`: int.
- `isOptional`: bool.
- `fields`: array de `WorkflowTemplateStepTaskFieldDetailResponse`.

`WorkflowTemplateStepTaskFieldDetailResponse`: idêntico em forma ao step field.

Backend ordena os arrays:

- `steps` por `order` crescente.
- `fields` (de step ou task) por `order` crescente.
- `tasks` por `order` crescente.

### Response — Add step / Add task / Add field (step ou task)

`200 OK` com `IdentifierResponse`:

- `id`: GUID — id da entidade criada (stepId, taskId ou fieldId).

### Response — Errors

Envelope padrão de erro do projeto: `{ statusCode, message, path, traceId, errors, detail }`.

Códigos de erro de negócio (ver seção 10):

- `WorkflowTemplate.NotEditable` — `409 Conflict` (ConflictError).
- `WorkflowTemplate.StepTitleAlreadyExists` — `409 Conflict`.
- `WorkflowTemplate.TaskTitleAlreadyExists` — `409 Conflict`.
- `WorkflowTemplate.StepFieldLabelAlreadyExists` — `409 Conflict`.
- `WorkflowTemplate.TaskFieldLabelAlreadyExists` — `409 Conflict`.
- `WorkflowTemplate.InvalidStatusTransition` — sem prefixo de tipo
  (`new(...)` direto, não embrulhado em `ConflictError`); mapeamento HTTP
  segue o default do `Error` base. Ver seção 16.
- `WorkflowTemplate.ActivationRequiresAtLeastOneStep` — idem.
- `WorkflowTemplate.ActivationRequiresAllStepsToHaveTask` — idem.
- `WorkflowTemplate.StepNotFound` / `TaskNotFound` / `StepFieldNotFound` /
  `TaskFieldNotFound` — `404 NotFound` (NotFoundError).

### 8.1. Formato canônico versus formato de exibição

- `createdAt`, `updatedAt`: ISO-8601 com offset (`DateTimeOffset`). Frontend
  formata para exibição.
- `version`: `v{Major}.{Minor}` (ex.: `v1.0`) — chega pronto para exibição.
- `status`, `fieldType`, `type` (task): chegam como o **Name** do SmartEnum
  (PascalCase, sem tradução). Frontend deve mapear para texto localizado se
  necessário.
- `options` (campo Select): JSON string no formato decidido pelo cliente —
  o backend não valida estrutura interna além de "não vazio".
- `configuration` (task): JSON string opaca; default `"{}"`. Backend não
  valida estrutura.

### 8.2. Nullabilidade e ausência de campo

- `updatedAt`: pode vir `null` (template recém-criado, sem updates posteriores).
- `options` (em fields): vem `null` para `fieldType ≠ Select`; vem string não
  vazia para `fieldType = Select`.
- `configuration` (em tasks): nunca null. Default é `"{}"`.
- `description` (template, step, task): obrigatório no input — nunca chega
  nulo no output.
- Listas vazias (`steps`, `tasks`, `fields`): aparecem como `[]`, nunca null.

---

## 9. Estados de negócio e transições

### Template

Estados:

- `Draft` (valor 0): editável; é o estado inicial obrigatório.
- `Active` (valor 1): publicado; imutável para steps/tasks/fields.
- `Archived` (valor 2): terminal; nenhuma escrita aceita.

Transições suportadas pela API:

- `criação → Draft`: automática no `POST /workflow-template`.
- `Draft → Active`: via `PATCH /activate` — exige `Steps.Count >= 1` e cada
  step com `Tasks.Count >= 1`.
- `Active → Archived`: via `PATCH /archive`.

Transições **não** suportadas:

- `Draft → Archived` (handler exige `status == Active`).
- `Active → Draft`.
- `Archived → *`.

CTAs e telas dependentes:

- Editar steps/tasks/fields: apenas em `Draft`.
- Botão "Ativar template": apenas em `Draft`, e desabilitado se faltar step
  ou step sem task.
- Botão "Arquivar template": apenas em `Active`.
- Atualizar nome/descrição do template: handler **não bloqueia por status**
  (qualquer template responde 200, mesmo `Active` ou `Archived`). Ver gap.

### Step / Task / Field

Não têm `status` próprio. Editabilidade segue o `status` do template-pai
(controlada pelos handlers, que verificam `template.Status != TemplateStatus.Draft`
antes de qualquer escrita).

### 9.1. Ordenação, filtros e agrupamentos com semântica de negócio

**Ordenação:**

- Listagem (`GetAll`): backend ordena por `Name ASC, CreatedAt ASC`. Frontend
  deve preservar essa ordem (não reordenar localmente).
- Detalhe (`GetById`): `steps`, `fields` e `tasks` chegam ordenados por
  `Order ASC`. Mesmo princípio: preservar.

**Filtros implícitos na listagem:**

- `IsDeleted == false` sempre.
- `Status == Active` sempre incluído (independente de role).
- `Status == Draft` incluído **apenas se** `role == SystemAdmin && includeDraft == true`.
- `Status == Archived` incluído **apenas se** `role == SystemAdmin && includeArchive == true`.

**Filtros não suportados:**

- Não há filtro por nome, por domínio, por data de criação ou por versão.

**Insert-shift no order:**

Após qualquer Add/Update com `order`, outros itens do mesmo escopo podem ter
seu `order` alterado pelo backend. O frontend deve **recarregar a lista**
após essas operações — não reordenar localmente.

---

## 10. Erros esperados e edge cases

| Código | Quando acontece | Por que acontece | O que o frontend deve comunicar |
|---|---|---|---|
| `WorkflowTemplate.NotEditable` | Qualquer escrita em step/task/field quando o template está em `Active` ou `Archived`. | Templates publicados são imutáveis; templates arquivados são terminais. | "O template não pode ser editado neste status." Bloquear preventivamente os CTAs de edição quando `status != Draft`. |
| `WorkflowTemplate.InvalidStatusTransition` | Tentativa de ativar não-`Draft` ou arquivar não-`Active`. | Transições são unidirecionais e não cíclicas. | Mensagem do backend já é específica ("Cannot transition from X to Y"). Frontend não deve permitir o CTA quando o status não permite. |
| `WorkflowTemplate.ActivationRequiresAtLeastOneStep` | Activate em template sem steps. | Pré-condição de ativação. | "Adicione ao menos uma etapa antes de ativar." Bloquear CTA "Ativar" enquanto `steps.length === 0`. |
| `WorkflowTemplate.ActivationRequiresAllStepsToHaveTask` | Activate com alguma step sem task. | Pré-condição de ativação. | "Cada etapa precisa ter ao menos uma tarefa antes de ativar." Bloquear CTA quando alguma step.tasks.length === 0`. |
| `WorkflowTemplate.StepTitleAlreadyExists` | Add/Update step com título duplicado no template (case-insensitive, com trim). | Unicidade de title por template. | Erro inline no campo `title` da step. |
| `WorkflowTemplate.TaskTitleAlreadyExists` | Add/Update task com título duplicado na step. | Unicidade de title por step. | Erro inline no campo `title` da task. |
| `WorkflowTemplate.StepFieldLabelAlreadyExists` | Add/Update step field com label duplicado na step. | Unicidade de label por step. | Erro inline no campo `label`. |
| `WorkflowTemplate.TaskFieldLabelAlreadyExists` | Add/Update task field com label duplicado na task. | Unicidade de label por task. | Erro inline no campo `label`. |
| `WorkflowTemplate.StepNotFound` | Step inexistente em update/remove. | Id passado não corresponde a nenhuma step. | Mensagem genérica de "etapa não encontrada"; recarregar o template. |
| `WorkflowTemplate.TaskNotFound` | Task inexistente em update/remove. | Idem. | "Tarefa não encontrada"; recarregar. |
| `WorkflowTemplate.StepFieldNotFound` / `TaskFieldNotFound` | Field inexistente em update/remove. | Idem. | "Campo não encontrado"; recarregar. |
| `WorkflowTemplateNotFound` | `GetById` com id inexistente. | Não há projection para o id. | `404`. Tela de "Template não encontrado". |
| `400` ValidationError | Body inválido pelo FluentValidation. | `Name`, `Title`, `Description`, `Type`, `FieldType`, `Order`, `Configuration` faltando ou fora de regra. | Mostrar mensagens do backend nos campos correspondentes. |
| `401` | Token ausente/inválido. | Sem auth. | Redirecionar ao login. |
| `403` | Role insuficiente em escrita (`!= SystemAdmin`). | Policy `CanManageWorkflowTemplates`. | Mensagem de "permissão negada". |

Edge cases:

- Reordenamento em escopo com 1 item: clamp reduz para máx adequado.
- Update de order para o mesmo valor que o item já possui: idempotente.
- Template ativo com tasks todas opcionais: ativação bem-sucedida (regra olha
  só "ao menos 1 task", não `isOptional`).
- Múltiplos templates ativos: permitido pelo backend; restrição é de outra feature.

---

## 11. Semântica de autenticação e autorização

- Todos os endpoints exigem `Authorize` (token JWT).
- Escritas (`POST`/`PUT`/`PATCH`/`DELETE`) exigem `Authorize(Policy =
  IdentityPolicies.CanManageWorkflowTemplates)`. Essa policy resolve para
  `HasRoleRequirement(UserRole.SystemAdmin.Name)`. Apenas `SystemAdmin`.
- Leituras (`GET /workflow-template` e `GET /workflow-template/{id}`) usam
  `[Authorize]` simples — qualquer usuário autenticado.
- Filtragem por role na listagem é feita **na camada de aplicação**
  (`GetAllWorkflowTemplatesHandler`), não na policy. `currentUserService.Role
  == IdentityRoles.SystemAdmin` decide se `includeDraft`/`includeArchive` são
  aplicados.

Hoje a auth está pronta no contrato: todos os endpoints estão de fato
autenticados e o filtro de role na listagem está implementado.

Gap (ver seção 16): no `GetWorkflowTemplateByIdHandler`, **não há filtro por
role nem por status do template**. Um `AccountancyAdmin` pode acessar um
template `Draft` ou `Archived` por id direto. A spec exige `NotFoundError`
nesse caso.

---

## 12. Impacto de UX que o frontend precisa saber

- `POST` retornam `IdentifierResponse` com o `id` da entidade criada.
- `Add`/`Update` em step/task/field reordenam outros itens do mesmo escopo
  (insert-shift). Frontend deve recarregar o detalhe do template após essas
  operações para refletir os novos `order`.
- `Activate` muda `status` para `Active` e congela a edição — frontend
  precisa atualizar o template carregado e revalidar quais CTAs estão
  disponíveis.
- `Archive` muda `status` para `Archived` e bloqueia tudo. Pode redirecionar
  para a listagem.
- `RemoveStep` / `RemoveTask` removem em cascata fields/tasks/task fields
  — frontend deve recarregar para refletir a perda das entidades.
- `RemoveStepField` / `RemoveTaskField` removem só o field.
- A listagem é paginada; frontend precisa controlar `pageNumber`/`pageSize`.
- Mensagens de erro de unicidade devem ser exibidas **inline no campo**
  correspondente (`title`, `label`).
- Não há máscara em nenhum campo (todos são textos simples ou enums por nome).
- `configuration` (task): frontend deve enviar **string** (JSON serializado),
  não objeto.
- `options` (field Select): também **string** (JSON serializado).

---

## 13. Impacto de dados no frontend

Após cada mutation:

| Mutation | Leituras desatualizadas |
|---|---|
| Create template | Lista (`GET /workflow-template`). |
| Update template | Lista + detalhe (`GET /workflow-template/{id}`). |
| Activate template | Lista + detalhe; muda `status` e `updatedAt`. |
| Archive template | Lista + detalhe; muda `status` e `updatedAt`. |
| Add step | Detalhe. Outros steps podem ter `order` alterado. |
| Update step | Detalhe. Outros steps podem ter `order` alterado. |
| Remove step | Detalhe. Cascata afeta tasks e fields aninhados. |
| Add/Update/Remove task | Detalhe; outros tasks da step podem ter `order` alterado. |
| Add/Update/Remove field (step ou task) | Detalhe; outros fields do escopo podem ter `order` alterado. |

UI otimista é segura para `Update` simples (template name/description sem
mudança de status). Para qualquer operação com `order`, **não** assumir
otimismo: o backend pode ter rebalanceado outros itens — recarregar.

Risco de stale data: alto se frontend reordenar localmente após um Add/Update
sem recarregar — pode mostrar `order` divergente do banco.

### 13.1. Identificadores estáveis e chaves de reconciliação

- `id` (template, step, task, field): GUID estável; chave canônica de
  reconciliação. Nunca muda após criação.
- `templateId`/`stepId`/`taskId`/`fieldId` em rotas: idem.
- `name`, `title`, `label`: **mutáveis**; nunca usar como chave.
- `order`: **mutável a cada Add/Update**; nunca usar como chave.
- `version` (template): muda apenas via `societario-template-versioning`;
  hoje sempre `v1.0`. Não é chave.
- `status`: muda em activate/archive; não é chave.

### 13.2. Compatibilidade e evolução de contrato

- Não há contrato legado conhecido; este é o primeiro release.
- A feature `societario-template-versioning` (futura) deve introduzir
  endpoints novos sem quebrar os atuais.
- `IWorkflowTemplateDataService` (consumido por `WorkflowProcess`) é interna
  ao backend; não afeta contrato HTTP.

---

## 14. Observabilidade e suporte

- O envelope global de erro (`Core`) inclui `traceId`. Frontend deve logar
  `traceId` em qualquer erro reportado pelo usuário.
- Os event handlers de projection logam falhas via `ILogger`
  (`logger.LogError`). Não há log estruturado adicional configurado neste
  módulo no nível Application.
- O event store (`workflow_template.WorkflowTemplateStoreEvents`) é a fonte
  de verdade — toda mudança fica auditável aí, mesmo sem log explícito.

---

## 15. Exemplos reais de payload

```json
// Request — POST /api/v1/workflow-template
{
  "name": "Abertura de Empresa LTDA",
  "description": "Modelo padrão para abertura de empresas LTDA com documentação completa."
}

// Response 200 — POST /api/v1/workflow-template
{
  "id": "8a1f3e6d-94b1-4c0e-9d3a-2c8e4b5a1f0c"
}

// Request — PUT /api/v1/workflow-template/{id}
{
  "name": "Abertura de Empresa LTDA — v2",
  "description": "Versão atualizada com novas etapas."
}

// Request — PATCH /api/v1/workflow-template/{id}/activate
// (corpo vazio)

// Request — PATCH /api/v1/workflow-template/{id}/archive
// (corpo vazio)

// Request — POST /api/v1/workflow-template/{id}/step
{
  "title": "Coleta de documentos",
  "description": "Receber documentos do cliente.",
  "order": 1
}

// Request — POST /api/v1/workflow-template/{id}/step/{stepId}/task
{
  "title": "Receber RG e CPF",
  "description": "Solicitar e validar RG e CPF dos sócios.",
  "type": "Manual",
  "configuration": "{}",
  "order": 1,
  "isOptional": false
}

// Request — POST /api/v1/workflow-template/{id}/step/{stepId}/fields
// fieldType = Text
{
  "label": "Nome fantasia",
  "fieldType": "Text",
  "options": null,
  "order": 1
}

// Request — POST /api/v1/workflow-template/{id}/step/{stepId}/fields
// fieldType = Select
{
  "label": "Regime tributário",
  "fieldType": "Select",
  "options": "[\"Simples Nacional\",\"Lucro Presumido\",\"Lucro Real\"]",
  "order": 2
}

// Response 200 — POST /step (qualquer Add)
{
  "id": "1c2b3a4d-5e6f-7081-9aab-bcdef0123456"
}

// Response 200 — GET /api/v1/workflow-template (paged)
{
  "items": [
    {
      "id": "8a1f3e6d-94b1-4c0e-9d3a-2c8e4b5a1f0c",
      "name": "Abertura de Empresa LTDA",
      "description": "Modelo padrão...",
      "version": "v1.0",
      "status": "Active",
      "createdAt": "2026-05-06T13:12:00+00:00"
    }
  ],
  "pageNumber": 1,
  "pageSize": 10,
  "totalCount": 1
}

// Response 200 — GET /api/v1/workflow-template/{id}
{
  "id": "8a1f3e6d-94b1-4c0e-9d3a-2c8e4b5a1f0c",
  "name": "Abertura de Empresa LTDA",
  "description": "Modelo padrão...",
  "version": "v1.0",
  "status": "Draft",
  "createdAt": "2026-05-06T13:12:00+00:00",
  "updatedAt": null,
  "steps": [
    {
      "id": "11111111-1111-1111-1111-111111111111",
      "title": "Coleta de documentos",
      "description": "Receber documentos do cliente.",
      "order": 1,
      "fields": [
        {
          "id": "22222222-2222-2222-2222-222222222222",
          "label": "Nome fantasia",
          "fieldType": "Text",
          "options": null,
          "order": 1
        }
      ],
      "tasks": [
        {
          "id": "33333333-3333-3333-3333-333333333333",
          "title": "Receber RG e CPF",
          "description": "Solicitar e validar...",
          "type": "Manual",
          "configuration": "{}",
          "order": 1,
          "isOptional": false,
          "fields": []
        }
      ]
    }
  ]
}

// Erro de validação (400)
{
  "statusCode": 400,
  "message": "One or more validation errors occurred.",
  "path": "/api/v1/workflow-template",
  "traceId": "00-abc-def-00",
  "errors": [
    { "field": "Name", "message": "Name is required" },
    { "field": "Description", "message": "Description cannot exceed 500 characters" }
  ],
  "detail": null
}

// Conflito — template não editável (status != Draft)
{
  "statusCode": 409,
  "message": "Only draft templates can be modified.",
  "path": "/api/v1/workflow-template/{id}/step",
  "traceId": "00-...",
  "errors": [
    { "code": "WorkflowTemplate.NotEditable", "message": "Only draft templates can be modified." }
  ],
  "detail": null
}

// Conflito — título de step duplicado
{
  "statusCode": 409,
  "message": "Step title 'Coleta de documentos' already exists.",
  "path": "/api/v1/workflow-template/{id}/step",
  "traceId": "00-...",
  "errors": [
    { "code": "WorkflowTemplate.StepTitleAlreadyExists",
      "message": "Step title 'Coleta de documentos' already exists." }
  ],
  "detail": null
}

// Ativação sem steps
{
  "statusCode": 400,
  "message": "Template must have at least one step to be activated.",
  "path": "/api/v1/workflow-template/{id}/activate",
  "traceId": "00-...",
  "errors": [
    { "code": "WorkflowTemplate.ActivationRequiresAtLeastOneStep",
      "message": "Template must have at least one step to be activated." }
  ],
  "detail": null
}
```

Os exemplos de envelope de erro acima refletem o **formato canônico do
projeto**; os campos exatos (`code`/`message`, posição do código no payload)
seguem o que o `Core.Shared.Errors` produz no projeto — não foram lidos
diretamente os mappers HTTP do `Core`, então o frontend deve consultar o
contrato real ao integrar.

---

## 16. Gaps, limitações e decisões abertas

### Gap 1 — Update de template não verifica status

O `UpdateWorkflowTemplateHandler` não checa `template.Status` antes de aplicar
`Update(name, description)`. Resultado: nome e descrição podem ser alterados
em template `Active` ou `Archived`. A spec/logic afirma:

> O SystemAdmin pode atualizar nome e descrição de um template em qualquer
> status, desde que o template não esteja arquivado.

**Divergência:** o código permite update mesmo em `Archived`. A spec proibia
update em `Archived`. O Swagger do `UpdateWorkflowTemplateEndpoint` diz
"Regras: Nenhuma".

**Decisão pendente:** alinhar handler à spec (bloquear update em `Archived`)
ou alinhar spec ao código (permitir update em qualquer status).

### Gap 2 — Get by id não filtra por role nem por status

O `GetWorkflowTemplateDetailByIdHandler` apenas busca o id na projection. Não
há filtro por `currentUserService.Role` nem por `template.Status`. A spec
afirma:

> Templates em Draft ou Archived retornam NotFoundError para roles que não
> são SystemAdmin, mesmo que o id exista.

**Divergência:** hoje, um `AccountancyAdmin` que conheça o id de um template
`Draft` ou `Archived` recebe `200 OK` com o detalhe completo.

**Decisão pendente:** adicionar filtro de role/status no handler de detalhe.

### Gap 3 — Description do Swagger do Archive diverge do código

`ArchiveWorkflowTemplateEndpoint` declara:

> Status permitidos de origem: Draft, Active.

O `ArchiveWorkflowTemplateHandler` rejeita qualquer status que não seja
`Active`. A spec/logic concorda com o código (`Active → Archived` apenas).
A descrição do Swagger está desalinhada.

**Decisão pendente:** corrigir o Description do `ArchiveWorkflowTemplateEndpoint`.

### Gap 4 — Implementação de IWorkflowTemplateDataService ausente

A interface anti-corrupção
`WorkflowProcess.Application.Services.IWorkflowTemplateDataService` está
declarada no módulo `WorkflowProcess`, mas não há classe que a implemente
neste módulo `WorkflowTemplate` (busca em
`src/Web/CrossModule/**/WorkflowTemplate*` retornou vazio). Sem essa
implementação, o `WorkflowProcess` não consegue resolver snapshots de
template ativo.

**Decisão pendente:** se este módulo é responsável pela implementação ou se
ela vive na Web layer/Cross-module em outro repositório de feature.

### Gap 5 — Erros de transição e ativação não usam ConflictError/BadRequestError

`WorkflowTemplateErrors.InvalidStatusTransition`,
`ActivationRequiresAtLeastOneStep` e `ActivationRequiresAllStepsToHaveTask`
são construídos como `new Error(code, message)` puro, sem embrulho em
`ConflictError`/`BadRequestError`. O mapeamento HTTP final depende do default
do `Error` base no `Core`. Os outros erros do mesmo arquivo
(`TemplateNotEditable`, `StepTitleAlreadyExists`, etc.) usam `ConflictError`.

**Impacto:** o status code real pode ser 500 ou 400 dependendo do default —
não foi confirmado por leitura do mapper HTTP. Frontend deve tratar pelo
`code`, não pelo status code, até que isso seja unificado.

**Decisão pendente:** padronizar para `BadRequestError` (validação de
pré-condição) ou `ConflictError` (estado inconsistente).

### Gap 6 — Sem validador para GetAllWorkflowTemplatesQuery

Não há FluentValidator para `GetAllWorkflowTemplatesQuery`. Os limites
mencionados no Swagger (`pageNumber >= 1`, `pageSize` entre 1 e 100) não são
verificados explicitamente neste módulo. Dependem do `Paging` em
`Core.Application.Pagination`.

### Gap 7 — Domínio aceita configuration null mas validador rejeita

A entidade `WorkflowTemplateStepTask` aceita `configuration` por construtor
sem validação; o agregado também não valida. O validador de input
(`AddWorkflowTemplateStepTaskValidator`) rejeita null com `NotNull`. Isso
significa que o caminho HTTP exige string não-nula, mas reconstrução do
agregado a partir do event store em teoria poderia aceitar null se um evento
antigo o tivesse persistido. Na prática, todos os eventos atuais persistem
com a string vinda da camada de aplicação.

### Gap 8 — Versão fixa em "v1.0"

A versão do template é sempre `v1.0` na criação. O value object
`TemplateVersion` suporta `Major`/`Minor`, mas não há comando para
incrementar. Esperado: ser endereçado pela feature
`societario-template-versioning`.

### Gap 9 — Não há RemoveTemplate / DeleteTemplate

Spec confirma: templates não são deletados, apenas arquivados. O backend não
expõe DELETE para template — coerente. Documentar para o frontend que
"excluir template" não existe; usar `Archive`.

---
