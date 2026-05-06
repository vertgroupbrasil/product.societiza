---
status: draft
spec: societario-workflow-process
domain: societario
created_at: 2026-05-06
---

# WorkflowProcess — Contrato de Integração Backend → Frontend

## 1. Identificação da feature

- **Nome da feature**: WorkflowProcess / Board operacional do societário (`societario-workflow-process`)
- **Módulo backend responsável**: `WorkflowProcess` (`src/Modules/WorkflowProcess/*`) com adaptação cross-module em `src/Web/Services/CrossModule/WorkflowTemplateDataService.cs`
- **Status atual**: `parcial`
- **Data da última atualização**: 2026-05-06
- **Responsável técnico**: não identificado no código

O código atual implementa a base de domínio e parte da camada Application para
criação de processos, preenchimento de campos e resolução de tarefas. Isso já
é suficiente para descrever o modelo de dados, os comandos internos e os erros
de negócio. Ainda assim, a feature **não está pronta para integração frontend**:
não há endpoints HTTP, não há handlers de query para board/detalhe, não há
projeção populada para leitura e o módulo não está instalado no `Program.cs`.

Em outras palavras: o backend já sabe o que é um `WorkflowProcess`, mas o host
web ainda não oferece uma API pública para operar esse conceito.

---

## 2. Objetivo de negócio da feature

No produto, o `WorkflowProcess` existe para ser o board operacional onde a
contabilidade executa de fato os processos societários. Ele é a camada que
transforma um `WorkflowTemplate` configurado em trabalho real: criação de um
processo para um cliente, materialização das etapas e tarefas daquele fluxo,
preenchimento de campos durante a operação e avanço do processo até conclusão.

Esse papel é central para a proposta da Societiza. O contexto de produto deixa
claro que a plataforma quer substituir planilhas, memórias individuais e
comunicação fragmentada por uma operação estruturada, rastreável e multi-tenant
dentro da `accountancy`. O `WorkflowProcess` é a peça que fecha esse ciclo.
Sem ele, `WorkflowTemplate` continua sendo apenas configuração.

Do ponto de vista do usuário final esperado, esta seria a feature diária do
contador: criar processos de abertura, alteração e baixa; abrir um processo;
acompanhar o andamento; preencher campos; concluir ou pular tarefas; e observar
o avanço do card no board. No código atual, porém, essa jornada só existe
parcialmente no backend. A parte implementada é a fundação de escrita
(`write-side`). A experiência de board e leitura operacional ainda não está
exposta ao frontend.

A conexão com outras features é direta:

- depende de `accountancy` como contorno multi-tenant;
- depende de `workflow-template` como fonte da estrutura do processo;
- deveria alimentar histórico, dashboard, notificações e versionamento de
  template no futuro;
- hoje ainda não entrega essas integrações externamente.

---

## 3. Escopo atual versus escopo futuro

**O que já está implementado no backend hoje:**

- Aggregate `WorkflowProcess` com snapshot do template na criação.
- SmartEnums de tipo e status:
  `ProcessType`, `ProcessStatus`, `StepInstanceStatus`, `TaskInstanceStatus`.
- Materialização runtime de:
  `ProcessStepInstance`, `ProcessStepTaskInstance`,
  `ProcessStepFieldInstance`, `ProcessStepTaskFieldInstance`.
- Comandos internos de aplicação:
  - `CreateWorkflowProcessCommand`
  - `CompleteProcessTaskCommand`
  - `SkipProcessTaskCommand`
  - `FillProcessStepFieldCommand`
  - `FillProcessTaskFieldCommand`
- Validators FluentValidation para os cinco comandos.
- Erros de domínio/aplicação específicos do módulo:
  `TemplateNotActiveOrNotFound`, `ProcessNotFound`,
  `ProcessAlreadyCompleted`, `StepNotFound`, `TaskNotFound`,
  `TaskAlreadyResolved`, `StepFieldNotFound`, `TaskFieldNotFound`.
- DTOs compartilhados já desenhados para leitura futura:
  `GetProcessBoardQuery`, `GetProcessDetailQuery`,
  `ProcessBoardItemResponse`, `ProcessDetailResponse` e DTOs de steps/tasks/fields.
- Contextos de persistência e configuração de event store/snapshot para o
  aggregate.
- Serviço cross-module `WorkflowTemplateDataService` que converte
  `WorkflowTemplate` em snapshots de steps/tasks/fields para a criação do
  processo.

**O que ainda não está implementado:**

- Endpoints HTTP em `src/Web/Endpoints/Modules/WorkflowProcess/`.
- Rotas públicas em `src/Web/Endpoints/Routes/`.
- Query handlers para `GetProcessBoardQuery` e `GetProcessDetailQuery`.
- Projeção de leitura do board/detalhe populada por handlers de evento.
- Publicação de `WorkflowProcessCompleted` como integration event consumível
  por outros módulos.
- Consumer configuration MassTransit específica do módulo.
- Qualquer contrato HTTP estável consumível pelo frontend.
- Instalação explícita do módulo `WorkflowProcess` no host web
  (`Program.cs` chama `InstallModulesFromAssemblies` apenas para Accountancy,
  Identity e WorkflowTemplate).

**O que existe no domínio mas não está exposto por endpoint:**

- Toda a semântica de criação e evolução do aggregate.
- As transições de estado de processo, etapa e tarefa.
- Os IDs estáveis de instância gerados na materialização.
- Os DTOs de board/detalhe, que hoje são apenas contratos internos.

**O que é apenas visão futura de produto:**

- Board kanban utilizável no frontend.
- Histórico de processos concluídos.
- Dashboard operacional.
- Notificações baseadas em mudança de estado.
- Integração com versionamento de template para archive automático.
- Portal do cliente.

**O que ainda está em discussão:**

- Como a camada HTTP será desenhada.
- Como o frontend obterá `accountancyId` e `templateId` sem expor escolhas que a
  spec de produto não quer no MVP.
- Como ficará a política real de autorização por papel para operação do board.
- Como serão tratados campos obrigatórios e validação por tipo em runtime.

---

## 4. Contexto de produto e semântica da feature

Na narrativa de produto, o usuário percebe `WorkflowProcess` como um board
operacional único por contabilidade. No código atual, a semântica real já
implementada é mais baixa e mais interna: o backend consegue criar um processo
baseado em um template ativo, copiar sua estrutura para dentro de um aggregate
próprio e registrar mutações subsequentes sobre tarefas e campos.

O que é considerado sucesso hoje, do ponto de vista do código:

1. um template informado pelo caller existe e está com `Status == "Active"`;
2. o processo é criado com `AccountancyId`, `TemplateId`, `TemplateVersion`,
   `ProcessType`, `TargetClient` e `Status = InProgress`;
3. steps, tasks e fields do snapshot são materializados como instâncias runtime;
4. o aggregate é persistido via `AppendEventsAsync`;
5. chamadas posteriores por IDs conseguem preencher campos ou resolver tarefas.

Essa semântica ainda não é a semântica final do produto. Dois exemplos
importantes:

- o código **não expõe board** nem detalhe do processo por HTTP;
- o código **não impõe** a ideia de "somente a etapa ativa pode ser operada".

Erro funcional, hoje, significa principalmente:

- o template informado não existe ou não está ativo;
- o processo não existe;
- o processo já está concluído e recebeu nova tentativa de mutação;
- step/task/field informados não existem dentro daquele aggregate;
- uma tarefa já foi resolvida antes e está recebendo nova resolução.

Também há uma diferença relevante entre a experiência pretendida e o comportamento
atual do aggregate:

- produto: primeira etapa deveria nascer em andamento;
- código: todas as etapas nascem `NotStarted` e uma etapa só vai para
  `InProgress` quando alguma tarefa dela é concluída ou pulada.

Isso afeta diretamente a futura UX do board e a interpretação dos estados.

---

## 5. Dependências e relações com outras features

**De quais outras features esta feature depende:**

- **WorkflowTemplate**: dependência obrigatória. O processo não nasce sozinho;
  ele é criado a partir de `GetActiveTemplateSnapshotAsync(templateId)`.
- **Accountancy**: o aggregate armazena `AccountancyId`, mas a validação do
  vínculo entre template e contabilidade não acontece dentro do módulo
  `WorkflowProcess`.
- **Identity/Auth**: dependência esperada do produto, mas ainda não
  materializada em endpoint, policy ou autorização nesta feature.

**Quais features consomem esta feature:**

- Hoje, nenhuma via contrato público.
- No desenho de produto, deveriam consumir:
  `societario-process-history`, `dashboard`, `notifications`,
  `societario-template-versioning` e portal do cliente.

**Se ela é central, complementar ou derivada no produto:**

Feature central do societário. É o write-side que deveria sustentar a operação
real da contabilidade.

**Dependências de dados de outro módulo:**

- `templateId` é fornecido pelo caller e resolvido por um serviço cross-module.
- `accountancyId` também é fornecido pelo caller; o módulo não o deriva do
  usuário autenticado.

**Ligação com dashboard, workflow, accountancy, notificações, etc.:**

- `workflow-template` fornece a estrutura.
- `accountancy` define o tenant.
- `dashboard` dependerá de read side ainda inexistente.
- `notifications` dependerão de evento de conclusão ainda não publicado como
  integration event.
- `template-versioning` é mencionado na spec de produto, mas ainda não há
  implementação de consumo/publicação nesse módulo.

---

## 6. Rotas públicas

Base path teórico da plataforma: `/api/v1/...`.

Estado real da feature hoje: **nenhuma rota pública de WorkflowProcess foi
registrada no host web**.

| Método | Path | Tipo | Finalidade | Params de rota | Pronta para frontend? |
|---|---|---|---|---|---|
| — | — | — | Não existem endpoints HTTP de `WorkflowProcess` em `src/Web/Endpoints/Modules/WorkflowProcess/` | — | não |

Isso significa que o frontend **não consegue** hoje:

- criar um processo via API;
- listar board;
- abrir detalhe do processo;
- concluir tarefa;
- pular tarefa;
- preencher campo de etapa;
- preencher campo de tarefa.

Contratos internos já modelados, mas ainda não expostos:

- `CreateWorkflowProcessCommand`
- `CompleteProcessTaskCommand`
- `SkipProcessTaskCommand`
- `FillProcessStepFieldCommand`
- `FillProcessTaskFieldCommand`
- `GetProcessBoardQuery`
- `GetProcessDetailQuery`

---

## 7. Contrato de request

As operações abaixo são **contratos internos de aplicação**, não requests HTTP
públicos. O frontend ainda não chama esses contratos diretamente.

### Request — CreateWorkflowProcessCommand (interno)

Estrutura:

- `accountancyId`: `Guid`, obrigatório, não pode ser `Guid.Empty`.
- `templateId`: `Guid`, obrigatório, não pode ser `Guid.Empty`.
- `processType`: `string`, obrigatório, precisa ser exatamente um dos nomes do
  `SmartEnum`:
  - `Abertura`
  - `Alteracao`
  - `Baixa`
- `targetClient`: `string`, obrigatório, não vazio.

Transformações obrigatórias no frontend:

- Nenhuma definida no código atual.
- Atenção: os valores aceitos para `processType` são os **nomes técnicos**
  sem acento, em PascalCase. A spec de produto fala em "abertura, alteração,
  baixa", mas o código aceita literalmente `Abertura`, `Alteracao`, `Baixa`.

Validações relevantes:

- `AccountancyId is required`
- `AccountancyId must be a valid GUID`
- `TemplateId is required`
- `TemplateId must be a valid GUID`
- `ProcessType is required`
- `ProcessType is not valid`
- `TargetClient is required`

Restrições de negócio adicionais observadas no handler:

- o template precisa existir e estar com status `"Active"`;
- a checagem é feita por `templateId`;
- o módulo não valida, por si só, se o `templateId` pertence à mesma
  `accountancyId` recebida.

### Request — CompleteProcessTaskCommand (interno)

Estrutura:

- `processId`: `Guid`, obrigatório, não pode ser `Guid.Empty`.
- `stepInstanceId`: `Guid`, obrigatório, não pode ser `Guid.Empty`.
- `taskInstanceId`: `Guid`, obrigatório, não pode ser `Guid.Empty`.

Transformações obrigatórias no frontend:

- Nenhuma definida.

Validações relevantes:

- `ProcessId is required`
- `ProcessId must be a valid GUID`
- `StepInstanceId is required`
- `StepInstanceId must be a valid GUID`
- `TaskInstanceId is required`
- `TaskInstanceId must be a valid GUID`

Restrições de negócio adicionais observadas no handler:

- o processo precisa existir;
- o processo não pode estar concluído;
- o step informado precisa existir dentro do processo;
- a task informada precisa existir dentro do step;
- a task precisa estar em `Pending`.

Ponto crítico:

- o handler **não valida** se o step é a etapa ativa do processo;
- o handler **não valida** ordem sequencial entre steps.

### Request — SkipProcessTaskCommand (interno)

Estrutura idêntica à de `CompleteProcessTaskCommand`:

- `processId`: `Guid`, obrigatório.
- `stepInstanceId`: `Guid`, obrigatório.
- `taskInstanceId`: `Guid`, obrigatório.

Validações relevantes:

- mesmas de `CompleteProcessTaskCommand`.

Restrições de negócio adicionais observadas no handler:

- mesmas de `CompleteProcessTaskCommand`.

Ponto crítico:

- pular uma tarefa é tratado com a mesma liberdade de navegação por IDs:
  qualquer task existente de um processo em andamento pode ser marcada como
  `Skipped`, sem guarda de etapa ativa.

### Request — FillProcessStepFieldCommand (interno)

Estrutura:

- `processId`: `Guid`, obrigatório.
- `stepInstanceId`: `Guid`, obrigatório.
- `fieldInstanceId`: `Guid`, obrigatório.
- `value`: `string`, obrigatório, não vazio.

Transformações obrigatórias no frontend:

- Nenhuma definida no código atual.

Validações relevantes:

- `ProcessId is required`
- `ProcessId must be a valid GUID`
- `StepInstanceId is required`
- `StepInstanceId must be a valid GUID`
- `FieldInstanceId is required`
- `FieldInstanceId must be a valid GUID`
- `Value is required`

Restrições de negócio adicionais observadas no handler:

- o processo precisa existir;
- o processo não pode estar concluído;
- o step precisa existir;
- o field precisa existir dentro do step.

Pontos críticos:

- não existe validação por tipo de campo (`Text`, `Date`, `Select`, etc.);
- não existe validação contra lista de opções;
- não existe bloqueio por etapa ativa.

### Request — FillProcessTaskFieldCommand (interno)

Estrutura:

- `processId`: `Guid`, obrigatório.
- `stepInstanceId`: `Guid`, obrigatório.
- `taskInstanceId`: `Guid`, obrigatório.
- `fieldInstanceId`: `Guid`, obrigatório.
- `value`: `string`, obrigatório, não vazio.

Transformações obrigatórias no frontend:

- Nenhuma definida.

Validações relevantes:

- `ProcessId is required`
- `ProcessId must be a valid GUID`
- `StepInstanceId is required`
- `StepInstanceId must be a valid GUID`
- `TaskInstanceId is required`
- `TaskInstanceId must be a valid GUID`
- `FieldInstanceId is required`
- `FieldInstanceId must be a valid GUID`
- `Value is required`

Restrições de negócio adicionais observadas no handler:

- o processo precisa existir;
- o processo não pode estar concluído;
- o step precisa existir;
- a task precisa existir dentro do step;
- o field precisa existir dentro da task.

Pontos críticos:

- também não existe validação por tipo;
- também não existe validação contra `Options`;
- também não existe bloqueio por etapa ativa.

### Request — GetProcessBoardQuery (contrato interno ainda sem handler)

Estrutura prevista:

- `accountancyId`: `Guid`
- `processType`: `string?`

O contrato existe em `Shared`, mas não há handler lendo isso nem endpoint
expondo a operação. Logo, ainda não existe comportamento real para documentar
sobre paginação, ordenação, agrupamento ou filtro efetivo do board.

### Request — GetProcessDetailQuery (contrato interno ainda sem handler)

Estrutura prevista:

- `processId`: `Guid`

Também existe apenas como contrato compartilhado. Não há handler, endpoint ou
read side implementada para respondê-lo.

---

## 8. Contrato de response

Como a feature ainda não expõe HTTP, as respostas abaixo descrevem o retorno
dos handlers e DTOs internos já modelados.

### Response — CreateWorkflowProcessCommand

Retorno de sucesso:

- `id`: `Guid` (`IdentifierResponse.Id`)

Significado:

- é o identificador estável do aggregate recém-criado;
- esse é o único payload de retorno hoje;
- nenhum snapshot do processo criado é devolvido junto;
- nenhum detalhe de steps/tasks/fields retorna no sucesso da criação.

Impacto no frontend:

- quando a API pública existir, o frontend provavelmente precisará de uma
  leitura adicional para montar o detalhe completo, a menos que o endpoint
  futuro decida enriquecer a resposta.

### Response — CompleteProcessTaskCommand

Retorno de sucesso:

- nenhum payload (`Result.Success()` sem body de domínio)

Impacto no frontend:

- não há DTO de resposta com o processo atualizado;
- quando isso virar endpoint, uma mutation dessas exigirá refetch de detalhe ou
  uso de resposta enriquecida, se o contrato futuro mudar.

### Response — SkipProcessTaskCommand

Retorno de sucesso:

- nenhum payload (`Result.Success()` sem body de domínio)

Impacto no frontend:

- mesmo impacto de `CompleteProcessTaskCommand`.

### Response — FillProcessStepFieldCommand

Retorno de sucesso:

- nenhum payload

Impacto no frontend:

- o frontend não receberia eco do valor salvo nem do `FilledAt`;
- precisaria refetch do detalhe, caso a operação seja exposta sem mudança de
  contrato.

### Response — FillProcessTaskFieldCommand

Retorno de sucesso:

- nenhum payload

Impacto no frontend:

- mesmo comportamento de `FillProcessStepFieldCommand`.

### Response — GetProcessBoardQuery (DTO definido, mas sem implementação)

DTO previsto por `ProcessBoardItemResponse`:

- `id`: `Guid`
- `processType`: `string`
- `targetClient`: `string`
- `status`: `string`
- `currentStepTitle`: `string?`
- `totalSteps`: `int`
- `completedSteps`: `int`
- `createdAt`: `DateTimeOffset`
- `completedAt`: `DateTimeOffset?`

Observação importante:

- o DTO existe, mas nenhum código atual o preenche.

### Response — GetProcessDetailQuery (DTO definido, mas sem implementação)

DTO previsto por `ProcessDetailResponse`:

- `id`: `Guid`
- `accountancyId`: `Guid`
- `templateId`: `Guid`
- `templateVersion`: `string`
- `processType`: `string`
- `targetClient`: `string`
- `status`: `string`
- `createdAt`: `DateTimeOffset`
- `completedAt`: `DateTimeOffset?`
- `steps`: `List<ProcessStepInstanceResponse>`

Cada `ProcessStepInstanceResponse` foi desenhado para conter:

- `id`: `Guid`
- `title`: `string`
- `order`: `int`
- `status`: `string`
- `completedAt`: `DateTimeOffset?`
- `tasks`: lista de `ProcessStepTaskInstanceResponse`
- `fields`: lista de `ProcessStepFieldInstanceResponse`

Cada `ProcessStepTaskInstanceResponse` foi desenhado para conter:

- `id`: `Guid`
- `title`: `string`
- `order`: `int`
- `isOptional`: `bool`
- `status`: `string`
- `completedAt`: `DateTimeOffset?`
- `skippedAt`: `DateTimeOffset?`
- `fields`: lista de `ProcessStepTaskFieldInstanceResponse`

Cada field response foi desenhado para conter:

- `id`: `Guid`
- `label`: `string`
- `fieldType`: `string`
- `options`: `string?`
- `isRequired`: `bool`
- `order`: `int`
- `value`: `string?`
- `filledAt`: `DateTimeOffset?`

### 8.1. Formato canônico versus formato de exibição

O código atual sugere os seguintes formatos canônicos:

- `Guid` para todos os identificadores.
- `DateTimeOffset` para timestamps (`CreatedAt`, `CompletedAt`, `SkippedAt`,
  `FilledAt`).
- `ProcessType` e statuses serializados pelos seus nomes:
  - `Abertura`, `Alteracao`, `Baixa`
  - `InProgress`, `Completed`
  - `NotStarted`, `InProgress`, `Completed`
  - `Pending`, `Completed`, `Skipped`
- `targetClient` como texto livre, sem máscara, sem normalização aparente.
- `options` como `string?` bruta, sem tipo forte. O código não define se a
  string é CSV, JSON, ou outro formato.

Nada disso chega hoje ao frontend porque não existe endpoint. Ainda assim, esse
é o formato que os DTOs e o domínio já apontam como contrato técnico provável.

### 8.2. Nullabilidade e ausência de campo

Campos explicitamente anuláveis nos DTOs já modelados:

- `currentStepTitle`: pode ser `null`
- `completedAt`: pode ser `null` em processo, step e task
- `skippedAt`: pode ser `null`
- `options`: pode ser `null`
- `value`: pode ser `null`
- `filledAt`: pode ser `null`

Semântica observada no código:

- `completedAt == null` significa "ainda não concluído";
- `skippedAt == null` significa "tarefa não foi pulada";
- `value == null` significa "campo ainda não preenchido";
- `options == null` significa que o template snapshot não informou opções.

O código não usa string vazia como estado especial. Nos validators, `Value` e
`TargetClient` vazios são rejeitados; logo, o preenchimento válido tende a ser
texto não vazio, não string vazia semântica.

---

## 9. Estados de negócio e transições

### Processo

Estados implementados no domínio:

- `InProgress`
- `Completed`

Transições implementadas:

- `Create()` cria o processo já em `InProgress`.
- O processo vai para `Completed` quando `Steps.All(s => s.Status == Completed)`.

Bloqueios implementados:

- processos `Completed` rejeitam:
  - completar tarefa;
  - pular tarefa;
  - preencher campo de etapa;
  - preencher campo de tarefa.

### Etapa

Estados implementados:

- `NotStarted`
- `InProgress`
- `Completed`

Transições implementadas:

- toda etapa nasce `NotStarted`;
- uma etapa vira `InProgress` quando alguma tarefa dela é concluída ou pulada e
  ela ainda estava `NotStarted`;
- uma etapa vira `Completed` quando todas as tarefas **não opcionais** da etapa
  estão em `Completed` ou `Skipped`.

Pontos críticos:

- a primeira etapa **não nasce** em `InProgress`;
- não há no aggregate o conceito de "etapa corrente única";
- o domínio não impede que múltiplas etapas saiam de `NotStarted` fora de ordem.

### Tarefa

Estados implementados:

- `Pending`
- `Completed`
- `Skipped`

Transições implementadas:

- toda task nasce `Pending`;
- `Pending -> Completed` via `CompleteTask`;
- `Pending -> Skipped` via `SkipTask`.

Bloqueios implementados:

- uma task que já não está `Pending` gera `WorkflowProcess.TaskAlreadyResolved`.

O que não existe:

- retorno de `Completed` ou `Skipped` para `Pending`;
- captura de justificativa ao pular;
- captura de quem concluiu/pulou.

### Campo de etapa / campo de tarefa

Estados observáveis:

- sem valor (`Value == null`)
- preenchido (`Value != null`)

Transições implementadas:

- qualquer field existente pode receber `Fill(...)` enquanto o processo estiver
  em andamento.

O que o código não faz:

- validar tipo do valor;
- validar se o campo é obrigatório para permitir avanço;
- validar se o step está ativo.

### 9.1. Ordenação, filtros e agrupamentos com semântica de negócio

O que o código já preserva:

- `step.Order` vem do snapshot do template;
- `task.Order` vem do snapshot do template;
- `field.Order` também vem do snapshot.

O que isso significa:

- quando houver leitura pública, a ordem oficial do domínio tende a ser a ordem
  carregada do template no momento da criação do processo.

O que ainda não existe:

- ordenação oficial do board;
- agrupamento por colunas;
- filtro implementado por `processType`;
- paginação de board;
- qualquer regra pública de "mostrar apenas processos em andamento".

Os DTOs de query existem, mas a semântica de leitura ainda não foi codificada.

---

## 10. Erros esperados e edge cases

### Template não encontrado ou não ativo

Quando acontece:

- `CreateWorkflowProcessHandler` chama
  `GetActiveTemplateSnapshotAsync(templateId)` e recebe `null`.

Por que acontece:

- o template não existe;
- ou existe, mas o status retornado pelo módulo de template não é exatamente
  `"Active"`.

O que o frontend deveria comunicar:

- hoje não há endpoint, então não há UX real;
- quando existir, a mensagem deve ser orientada a negócio:
  "não foi possível criar o processo porque o template informado não está ativo".

### Processo inexistente

Quando acontece:

- qualquer mutation por `processId` tenta carregar o aggregate e falha.

Por que acontece:

- ID inválido, processo inexistente ou não carregável pelo application service.

O que o frontend deveria comunicar:

- "processo não encontrado".

### Processo já concluído

Quando acontece:

- qualquer mutation é enviada para um processo com `Status == Completed`.

Por que acontece:

- o domínio trata concluído como estado terminal.

O que o frontend deveria comunicar:

- "este processo já foi concluído e não pode mais ser alterado".

### Step, task ou field inexistente

Quando acontece:

- o caller manda IDs que não pertencem ao aggregate carregado.

Por que acontece:

- stale data no frontend;
- montagem incorreta de request;
- tentativa de operar elemento removido/nunca materializado.

O que o frontend deveria fazer:

- refetch completo do detalhe quando a API existir;
- nunca assumir que IDs de template são iguais aos IDs de instância runtime.

### Task já resolvida

Quando acontece:

- tentativa de concluir ou pular task que já está `Completed` ou `Skipped`.

Por que acontece:

- clique duplicado;
- race condition;
- UI desatualizada.

O que o frontend deveria fazer:

- desabilitar ação duplicada localmente;
- tratar conflito como sinal de que o estado já mudou.

### Valor inválido semanticamente para o tipo do campo

Quando acontece:

- por exemplo, enviar texto arbitrário para um campo que no template é `Date`,
  `Boolean`, `Integer` ou `Select`.

Por que acontece:

- o módulo `WorkflowProcess` não valida tipo nem opções na operação de fill.

O que o frontend deveria saber:

- a proteção atual precisa vir da UI, não do backend do módulo;
- isso é um gap técnico importante, não um comportamento desejável.

### Operação fora da etapa "esperada"

Quando acontece:

- concluir/pular tarefa ou preencher campo de uma etapa futura ou passada.

Por que acontece:

- o código atual só valida existência dos IDs e estado do processo;
- não existe guarda de "etapa ativa".

O que o frontend deveria fazer:

- enquanto não houver regra backend, a UI sozinha não garante integridade;
- isso precisa ser tratado como limitação do backend atual.

### Etapa sem tarefas obrigatórias

Quando acontece:

- step com zero tasks não opcionais.

Comportamento esperado pelo produto:

- deveria concluir automaticamente.

Comportamento real do código:

- não existe gatilho de auto-conclusão na criação;
- a checagem só roda dentro de `CompleteTask` e `SkipTask`;
- se a etapa não tiver task resolvível, ela pode ficar presa em `NotStarted`.

### Feature sem endpoint público

Quando acontece:

- sempre, no estado atual do código.

Impacto:

- o frontend deve bloquear a integração desta feature como operação real;
- não existe base para construir tela funcional consumindo a API.

---

## 11. Semântica de autenticação e autorização

- A feature exige autenticação?
  - **No produto, sim.**
  - **No código atual do WorkflowProcess, ainda não há endpoint para aplicar auth.**

- Qual contexto de usuário ela assume?
  - O produto assume usuário operacional da contabilidade.
  - O código atual recebe `accountancyId` e `templateId` do caller e não deriva
    isso do usuário autenticado.

- Ela já depende de permissão por papel?
  - Não na implementação atual.
  - Não há `[Authorize]`, policy, checagem de role nem matriz de acesso dentro
    do módulo.

- Hoje todos os usuários têm o mesmo poder nessa feature?
  - No nível de código interno, qualquer caller que consiga disparar o comando
    com os IDs corretos exerce o mesmo poder.

- Existe gap entre auth real e auth esperada?
  - Sim, grande.
  - Produto: `AccountingAdmin` e `AccountingEmployee` operam dentro da
    contabilidade ativa.
  - Código atual: nenhuma dessas garantias está materializada em uma API pública
    ou em guards do módulo.

---

## 12. Impacto de UX que o frontend precisa saber

O principal impacto de UX hoje é simples: **não integrar a feature como pronta**.

Mais especificamente:

- não existe rota para tela de board;
- não existe rota para detalhe de processo;
- não existe mutation pública para criar processo;
- não existe resposta enriquecida após mutation;
- não existe garantia backend de etapa ativa;
- não existe validação backend de tipo/option dos campos;
- não existe read model pública para sincronizar a UI depois das mutations.

Se a API pública for adicionada reaproveitando os contratos atuais sem mudança:

- criação retorna apenas `id`;
- concluir/pular tarefa e preencher campos retornam sucesso vazio;
- o frontend precisará reler o detalhe depois de quase toda mutation;
- UI otimista é arriscada porque o backend ainda não garante sequência de steps
  nem bloqueio de etapa fora de ordem.

Também é relevante para UX futura:

- `processType` usa nomes técnicos sem acento;
- `targetClient` é texto livre;
- field values não vêm tipados fortemente.

---

## 13. Impacto de dados no frontend

Toda mutation desta feature, quando existir como API pública, tende a
desatualizar mais de uma leitura ao mesmo tempo:

- board da contabilidade;
- detalhe do processo;
- progresso por etapa;
- contadores agregados de steps concluídos;
- eventuais leituras derivadas de histórico e dashboard.

Hoje, como não existe read side pública, o risco de stale data é ainda maior:
o frontend não teria uma fonte oficial para reconciliar o resultado das
mutations.

UI otimista não é segura no estado atual do domínio exposto porque:

- completar/pular task pode completar step e processo em cascata;
- mas a API atual não devolve o novo estado;
- e o backend não impõe a mesma semântica de etapa ativa descrita no produto.

### 13.1. Identificadores estáveis e chaves de reconciliação

IDs estáveis já existentes no modelo:

- `processId`
- `stepInstanceId`
- `taskInstanceId`
- `fieldInstanceId`

IDs que **não** devem ser confundidos com identidade runtime:

- `templateId`
- `templateStepId`
- `templateTaskId`
- `templateFieldId`

O frontend futuro nunca deve usar:

- `title`
- `label`
- `targetClient`
- `order`

como chave de reconciliação. Esses campos são semânticos, mas não identitários.

### 13.2. Compatibilidade e evolução de contrato

Hoje não existe contrato HTTP legado nem novo para coexistir. O que existe são
contratos internos de aplicação.

Isso tem dois efeitos:

- o frontend ainda não tem compromisso público de retrocompatibilidade aqui;
- o time de backend ainda pode e provavelmente vai alterar os contratos antes da
  primeira exposição pública.

Mudanças que exigirão ajuste obrigatório no frontend quando a API surgir:

- definição real dos endpoints;
- forma de obter `accountancyId` e `templateId`;
- shape dos erros HTTP;
- decisão sobre resposta enriquecida versus refetch;
- decisão sobre validação tipada de campos;
- decisão sobre board e queries de leitura.

---

## 14. Observabilidade e suporte

- O endpoint retorna `traceId`?
  - Não se aplica hoje, porque não há endpoint.

- Existem logs relevantes para diagnóstico?
  - O host web tem logging global e request logging, mas não existe request de
    WorkflowProcess hoje.
  - O aggregate usa event sourcing; então os eventos do domínio tendem a ser a
    principal trilha técnica desta feature quando o módulo estiver ligado.

- Como o frontend pode registrar um erro de forma útil para suporte?
  - No estado atual, só faria sentido em ambiente de desenvolvimento interno.
  - Os identificadores úteis para suporte são:
    - `processId`
    - `stepInstanceId`
    - `taskInstanceId`
    - `fieldInstanceId`
    - `templateId`
    - `accountancyId`

Também vale registrar:

- qual comando foi tentado;
- qual `processType` foi usado na criação;
- qual valor foi enviado em `Value` ao preencher campo.

Não há hoje mecanismos específicos de observabilidade do board, do read model ou
de consumidores porque essas partes ainda não existem no módulo.

---

## 15. Exemplos reais de payload

Os exemplos abaixo refletem os **contratos reais já modelados no código**. Eles
não correspondem a endpoints HTTP públicos, porque esses endpoints ainda não
existem.

```json
// Request válido — CreateWorkflowProcessCommand (interno)
{
  "accountancyId": "4d3e0d8e-6b22-4d5f-95e0-8e2f837cc7d8",
  "templateId": "f9d88b6f-1e94-4408-b647-92156dbdce7f",
  "processType": "Abertura",
  "targetClient": "ACME Comercio LTDA"
}
```

```json
// Response válida — CreateWorkflowProcessCommand
{
  "id": "e1835f4a-2e15-4f67-b1a5-b6d8a9956401"
}
```

```json
// Request válido — CompleteProcessTaskCommand (interno)
{
  "processId": "e1835f4a-2e15-4f67-b1a5-b6d8a9956401",
  "stepInstanceId": "82359eb4-99ee-4d7f-9f34-bffddbafc9f4",
  "taskInstanceId": "4f6b7d5a-68e7-451a-8fdb-c6b4f0182eb9"
}
```

```json
// Response válida — CompleteProcessTaskCommand
{}
```

```json
// Estrutura lógica de erro de validação do comando
{
  "errors": {
    "ProcessType": [
      "ProcessType is not valid"
    ],
    "TargetClient": [
      "TargetClient is required"
    ]
  }
}
```

```json
// Estrutura lógica de erro de negócio retornada pelo handler
{
  "code": "WorkflowProcess.AlreadyCompleted",
  "message": "Workflow process is already completed and cannot be modified."
}
```

```json
// DTO previsto — ProcessDetailResponse (ainda sem handler)
{
  "id": "e1835f4a-2e15-4f67-b1a5-b6d8a9956401",
  "accountancyId": "4d3e0d8e-6b22-4d5f-95e0-8e2f837cc7d8",
  "templateId": "f9d88b6f-1e94-4408-b647-92156dbdce7f",
  "templateVersion": "1.0.0",
  "processType": "Abertura",
  "targetClient": "ACME Comercio LTDA",
  "status": "InProgress",
  "createdAt": "2026-05-06T12:30:00+00:00",
  "completedAt": null,
  "steps": [
    {
      "id": "82359eb4-99ee-4d7f-9f34-bffddbafc9f4",
      "title": "Consulta de viabilidade",
      "order": 1,
      "status": "NotStarted",
      "completedAt": null,
      "tasks": [
        {
          "id": "4f6b7d5a-68e7-451a-8fdb-c6b4f0182eb9",
          "title": "Emitir protocolo",
          "order": 1,
          "isOptional": false,
          "status": "Pending",
          "completedAt": null,
          "skippedAt": null,
          "fields": []
        }
      ],
      "fields": []
    }
  ]
}
```

---

## 16. Gaps, limitações e decisões abertas

- `logic.md` desta feature ainda está com `status: draft` em `product.societiza`.
  Este documento foi gerado por aprovação explícita do usuário em 2026-05-06,
  aceitando esse risco.
- O acesso ao repositório GitHub `product.societiza` falhou (`404`) no conector
  disponível. Por isso, este `integration.md` foi salvo localmente em fallback,
  dentro de `api.societiza/specs/societario-workflow-process/integration.md`.
- Não existia `specs/societario-workflow-process/integration.md` no
  `product.societiza` local no momento da geração.
- O módulo `WorkflowProcess` não possui endpoints HTTP públicos.
- O módulo `WorkflowProcess` não possui rotas registradas em `Web/Endpoints/Routes`.
- `Program.cs` não instala `WorkflowProcess.Infrastructure` em
  `InstallModulesFromAssemblies`, então o módulo ainda não está ligado ao host
  como módulo operacional completo.
- Não há query handlers para `GetProcessBoardQuery` e `GetProcessDetailQuery`.
- Não há projection handlers populando Mongo para board/detalhe.
- Não há consumer configuration nem publicação efetiva de
  `WorkflowProcessCompleted` como integration event externo.
- A criação do processo exige `templateId` explícito, o que diverge da narrativa
  de produto do MVP onde o contador não escolheria template manualmente.
- Todas as etapas nascem `NotStarted`; a primeira etapa não entra em andamento
  automaticamente na criação.
- O código não garante uma única etapa ativa nem ordem sequencial de execução.
- Tasks de etapas futuras ou passadas podem ser resolvidas se os IDs forem
  conhecidos e o processo ainda estiver em andamento.
- Fields de qualquer etapa/task existente podem ser preenchidos sem validação de
  etapa ativa.
- Todos os fields são materializados com `IsRequired = false`.
- O módulo não valida tipo de valor nem lista de opções no preenchimento de
  fields.
- O módulo não registra "quem" concluiu/pulou tarefa; registra apenas timestamps.
- Não há auto-conclusão na criação para etapas sem tarefas obrigatórias,
  apesar de a semântica de produto apontar nessa direção.
- Não foram encontrados testes automatizados específicos de `WorkflowProcess`
  no repositório atual.

Esses gaps não invalidam a utilidade do documento. Eles definem, com precisão,
o estado real da feature hoje: o domínio já existe, mas o contrato público
frontend ainda não existe e várias garantias narradas pela spec/logic de
produto ainda não estão materializadas no código.

---
