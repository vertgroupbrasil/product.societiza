---
nome: societario-workflow-process
modulo_backend: WorkflowProcess
status: existente
data_atualizacao: 2026-05-06
responsavel: Equipe Backend
---

# Contrato de Integração — Workflow Process (Board Operacional do Societário)

## 1. Identificação da feature

- **Nome da feature**: WorkflowProcess (Processos Societários)
- **Módulo backend responsável**: WorkflowProcess (write + read)
- **Status atual**: `existente`
- **Data da última atualização**: 2026-05-06
- **Responsável técnico**: Equipe Backend

A feature está inteiramente implementada e foca em oferecer o domínio de execução de processos societários baseados nos templates pré-configurados. Trata de criar instâncias de processo, recuperar o Board (Kanban), recuperar os detalhes por processo e realizar transições através da resolução (Complete/Skip) de "Tasks" e preenchimento de "Fields".

---

## 2. Objetivo de negócio da feature

O **WorkflowProcess** resolve a falta de uma visão operacional unificada para o setor societário. Antes, processos como Abertura, Alteração e Baixa de empresas ficavam rastreados em planilhas ou memórias de contadores. Agora, essa feature fornece um Kanban (Board) de tracking por Tenant (Accountancy).

Ela visa beneficiar diretamente o `AccountingAdmin` e o `AccountingEmployee` na sua rotina, oferecendo visibilidade. Cada card reflete um estado real derivado de tarefas concluídas. O processo possui Etapas (Steps), Tarefas (Tasks) atreladas à etapa, e sub-campos (Fields) nessas tarefas/etapas para armazenamento de texto.

---

## 3. Escopo atual versus escopo futuro

**O que já está implementado no backend hoje:**
- `POST /api/v1/workflow/processes` — Criação do processo clonando snapshot do Template.
- `GET /api/v1/workflow/processes` — Retorna os processos InProgress formatados para colunas de Board.
- `GET /api/v1/workflow/processes/{id}` — Retorna o overview completo: processo, steps, tasks, status atuais e campos.
- `POST /api/v1/workflow/processes/{id}/steps/{stepId}/tasks/{taskId}/complete` — Marca task finalizada.
- `POST /api/v1/workflow/processes/{id}/steps/{stepId}/tasks/{taskId}/skip` — Marca task como ignorada/saltada.
- `POST /api/v1/workflow/processes/{id}/steps/{stepId}/fields/{fieldId}/fill` — Preenche dado textual em uma coluna de etapa.
- `POST /api/v1/workflow/processes/{id}/steps/{stepId}/tasks/{taskId}/fields/{fieldId}/fill` — Preenche dado em tarefa.

**O que ainda não está implementado:**
- Alteração do tipo do processo societário ou responsável titular (não existem claims de responsabilidade nesta versão).
- Exclusão ou arquivamento de processo concluído.
- Drag-and-drop do Card explícito chamando um update de etapa.

**O que existe no domínio mas não está exposto por endpoint:**
- Remoção / Undo. Se preencher errado ou pular sem querer, ainda não há rollback mapeado explicitamente da tarefa na v1.

**O que ainda está em discussão / gaps de produto:**
- Ausências de validações restritivas profundas no Backend sobre o frontend deixar um campo de text em branco ao "salvar/completar" uma task.

---

## 4. Contexto de produto e semântica da feature

**Interação de UI vs API:**
O usuário **NÃO** avança as etapas arrastando o card.
A UI apresenta etapas baseadas nos steps do card (em `CurrentStepTitle`), e o backend que transiciona esse card de coluna **quando** todas as Tarefas obrigatórias do Step atual estiverem Resolvidas (`Completed` ou `Skipped`). 

**Sucesso funcional esperado:**
Quando a feature chama `CompleteProcessTaskCommand`, se for a última tarefa do Step atual, o Backend irá fechar o Step, atualizar a coluna do seu card para a próxima Step que é aplicável, e se acabar tudo, colocará o Status do Card como `Completed`.

---

## 5. Dependências e relações com outras features

- **Accountancy:** Um `process` vive amarrado ao ID de `Accountancy`. A listagem do board dependerá desta entidade.
- **WorkflowTemplate:** É obrigatoriedade existir um Template de Processo (`Active`) que o Frontend passará o `TemplateId` para ser clonado (em formato de snapshot).

---

## 6. Rotas públicas

| Método | Path | Tipo | Finalidade | Params de rota |
|---|---|---|---|---|
| POST | `/api/v1/workflow/processes` | Command | Criação do card. | nenhum |
| GET | `/api/v1/workflow/processes` | Query | Listagem do Board. | query de Tenant implicita/explicita |
| GET | `/api/v1/workflow/processes/{id}` | Query | Detalhamento total do progresso. | `id` do Process |
| POST | `.../{id}/steps/{stepId}/tasks/{taskId}/complete` | Command | Resolver Task como Concluida | id, stepId, taskId |
| POST | `.../{id}/steps/{stepId}/tasks/{taskId}/skip` | Command | Resolver Task como Pulada | id, stepId, taskId |
| POST | `.../{id}/steps/{stepId}/fields/{fieldId}/fill` | Command | Salva campo associado à Stage | id, stepId, fieldId |
| POST | `.../tasks/{taskId}/fields/{fieldId}/fill` | Command | Salva campo associado à Task | id, stepId, taskId, fieldId |

---

## 7. Contrato de request

### Request — CreateWorkflowProcess
`POST /api/v1/workflow/processes`
- `AccountancyId` (Guid): Identificador da empresa assinante.
- `TemplateId` (Guid): Identificador do Template Ativo (`TemplateNotActiveOrNotFound` atrelado em erro).
- `ProcessType` (String): Abertura, Alteração ou Baixa.
- `TargetClient` (String): Nome da entidade fim (O cliente real da contabilidade, em texto livre).

### Request — Complete ou Skip Task
Ambas são baseadas na rota (`POST`), payload vazio `{}`, requerem via Path params: `id` (ProcessId), `stepId` e `taskId`.
Restrição via Rule: Retorna erro 409 se o Processo já finalizado (`ProcessAlreadyCompleted`) ou a Tarefa já resolvida (`TaskAlreadyResolved`).

### Request — Fill Field (Step ou Task)
Corpo único:
- `Value` (String): Valor textual livre. 

---

## 8. Contrato de response

### Response — CreateWorkflowProcess
Retorna um `IdentifierResponse` que contém o `{ id: Guid }`. UI pode redirecionar para a view daquele `{id}`.

### Response — Board Item (`ProcessBoardItemResponse`)
- `Id` (Guid): ID único do Process.
- `TargetClient` (String): Fica no topo do Kanban Card.
- `ProcessType` (String): Tipo para ícones/cores visuais da UI.
- `Status` (String): `InProgress`, `Completed`.
- `CurrentStepTitle` (String?): A string em si do Step onde fica pendurada a coluna no Front. Fica Nulo quando concluído.
- `TotalSteps` / `CompletedSteps` (Int): Usado p/ progress bar miniatura.
- `CreatedAt` (DateTimeOffset): Data nativa ISOS.

### Response — Detail (`ProcessDetailResponse`)
- Carrega as entidades de Steps aninhadas em `Steps: List<ProcessStepInstanceResponse>`.
  - Cada Step contém `Tasks: List<ProcessStepTaskInstanceResponse>` e `Fields: List<...FieldInstanceResponse>`.
- O Frontend precisa iterar este array para checar o Step com Status `InProgress`, abrir os Itens em modo Editável (Checkboxes das Tasks e Imputs de Texto para os Fields). Tudo que for Step de Status `Completed` ou não iniciado fica em read-only (visualmente fechado ou travado no front).

### Nullabilidade
- Arrays sempre vêm em `[]` vazio caso não houver field/tasks, nunca null.
- Datas de execução como `CompletedAt` e `SkippedAt` originam como `null`.

---

## 9. Estados de negócio e transições (MUITO IMPORTANTE PARA O FRONTEND)

Somente o estado individual das TASKS importam em transições acionárias.
1. Front exibe uma Lista de checkboxes em "aberto": Tasks de Status `Pending`.
2. Front atinge botões/ações para disparar chamadas de endpoints `Complete` ou `Skip`. 
3. *Não existe mutation manual para a Step (`Em Andamento` => `Concluida`).*
4. Quando Front chamar Endpoint de Task, Front **deve** realizar *refetch* (`invalidateQueries`) do Backend no Detalhe INTEIRO do Processo, pois as flags e Steps podem ter avançado sozinhas dentro do Engine de Workflow do C#. NUNCA fazer Mutate Optimista do status individual da tarefa sem sincronizar o card-mãe.

## 10. Erros esperados e edge cases

O Backend reportará em `404` (NotFound) ou `409` (Conflict) com chaves específicas:

- `WorkflowProcess.TemplateNotActiveOrNotFound`: Informar na UI que "O template escolhido para criação do board foi suspenso ou não existe".
- `WorkflowProcess.ProcessAlreadyCompleted`: "Ação não pode ser realizada, o processo societário já encerrou sua jornada." Front deve inibir botões.
- `WorkflowProcess.TaskAlreadyResolved`: Se houve double click ou dupla janela na aba. Front acata e recarrega.

## 11. Semântica de autenticação e autorização

Trata as roles e tenants da Contabilidade (Identity normal do Auth `AccountancyId`), via header de Bearer. SystemAdmin é bloqueado em interações transacionais. Ações apenas para `AccountancyAdmin` ou sub-employees.

## 12. Impacto de UX que o frontend precisa saber

Ao enviar preenchimento `.../fill`, ele salva em banco instantaneamente. A transição de card na UI é derivada da leitura dos Steps no Detail. 

## 13. Impacto de dados no frontend

- **A query de Detalhe e de Board perdem sincronia ao realizar tasks.** Ao concluir task, chamar o `invalidateQueries` do react-query no `GetProcessBoardQuery` assim como no `GetProcessDetailQuery`.
- Não existem ids "públicos customizados". Toda amarração usa Guid.
- Chave de reconciliação de steps e Tasks é fixada. `StepInstanceId` não mudará no backend.

## 16. Gaps, limitações e decisões abertas

- O TargetClient pode vir branco caso o preenchimento não seja feito pela API (Não há validação direta notada se NotEmpty no core code no momento, confirmar validação no pipeline).
- Os forms da tarefa ou step não forçam explicitamente um rollback caso as premissas de transição fossem violadas por regras de negócios não implementadas.
- O campo string puro de `CurrentStepTitle` no Response de Board é instável caso houver mudança no template (Embora snapshots blindem isso no ID, cuidado na agrupagem via Title ao invés da Guid original da etapa).
