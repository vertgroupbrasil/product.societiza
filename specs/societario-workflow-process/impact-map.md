# Impact Map — WorkflowProcess (Board operacional do societário)

> Spec de origem: `specs/societario-workflow-process/spec.md`
> Data: 2026-04-11
> Status: `rascunho`

---

## Resumo da feature

O WorkflowProcess é a camada de execução do societário: um board kanban único
por contabilidade onde cada card representa um processo societário em andamento,
nascido de um WorkflowTemplate e tipado como abertura, alteração ou baixa. Ele
transforma a estrutura definida no template em operação real, substituindo
planilhas e memória humana por um estado visível, rastreável e compartilhado
entre a equipe da contabilidade.

---

## Repos impactados

| Repo | Tipo de impacto | Prioridade |
|------|----------------|------------|
| `api.societiza` | backend | alta |
| `web.societiza` | frontend | alta |

---

## `api.societiza` — O que precisa ser feito

### Domínio / módulo afetado

Novo bounded context / módulo: **WorkflowProcess** (dentro do domínio
societário). É um módulo novo, irmão do já existente `WorkflowTemplate`,
pertencente ao mesmo contexto de societário e scoped por `accountancyId`.

Módulos existentes tocados indiretamente (leitura apenas, sem alteração de
contrato):
- `WorkflowTemplate` — fonte do snapshot copiado para o Process no momento da
  criação.
- `Accountancy` — fornece o contorno multi-tenant via `accountancyId`.
- `Identity` — fornece o usuário autenticado para auditoria de salto/conclusão
  de tarefa.

### Novos endpoints

| Método | Path | Finalidade |
|--------|------|-----------|
| POST | `/workflow-process` | Criar um novo processo a partir do template ativo da contabilidade, recebendo `type` (abertura / alteração / baixa) e o cliente-alvo em texto livre. Materializa snapshot do template (`ProcessStepInstance`, `ProcessStepTaskInstance`, `ProcessStepFieldInstance`, `ProcessStepTaskFieldInstance`) com filtro `appliesTo` pelo tipo. |
| GET | `/workflow-process` | Listar processos da Accountancy ativa para alimentar o board. Suporta filtro opcional por `type` e por estado (em andamento / concluído). |
| GET | `/workflow-process/{processId}` | Obter o estado completo de um processo (steps, tasks, fields de step e de task, valores, timestamps). Usado ao abrir o detalhe do card. |
| POST | `/workflow-process/{processId}/steps/{stepId}/tasks/{taskId}/complete` | Marcar uma `ProcessStepTaskInstance` como concluída. Registra auditoria (quem e quando). Dispara reavaliação automática da Step: se todas as tasks obrigatórias estiverem resolvidas, a Step é fechada e o Process avança automaticamente para a próxima — e, se era a última Step obrigatória, o Process é concluído automaticamente. |
| POST | `/workflow-process/{processId}/steps/{stepId}/tasks/{taskId}/skip` | Saltar manualmente uma `ProcessStepTaskInstance` em runtime. Registra auditoria. Conta como "resolvida" para efeito do avanço automático da Step (mesma cascata do endpoint de complete). |
| PUT | `/workflow-process/{processId}/steps/{stepId}/fields/{fieldId}` | Preencher / atualizar valor de um `ProcessStepFieldInstance`. |
| PUT | `/workflow-process/{processId}/steps/{stepId}/tasks/{taskId}/fields/{fieldId}` | Preencher / atualizar valor de um `ProcessStepTaskFieldInstance`. |

**Não existem endpoints de avanço explícito nem de conclusão explícita.** A spec é firme: o avanço de Step e a conclusão do Process são consequências implícitas de `complete`/`skip` nas tasks obrigatórias — nunca ações do usuário. Não há `POST /workflow-process/{id}/steps/{id}/advance` nem `POST /workflow-process/{id}/complete`. Não existe também salto de Step inteira: o contador age sempre no nível da Task.

Observação: paths finais e formato do payload ficam a critério do backend, desde
que respeitem a semântica da spec. Este impact map define a superfície
funcional, não o contrato fechado (o contrato final será documentado em
`integration.md` após a implementação).

### Alterações em endpoints existentes

Nenhuma alteração obrigatória em endpoints do `WorkflowTemplate`, `Accountancy`
ou `Identity`. O Process apenas consome o template em leitura no momento da
criação.

### Novos modelos / entidades

Aggregate raiz **WorkflowProcess** contendo (convenção canônica de nomes com
sufixo `Instance` para entidades de materialização runtime):

- `WorkflowProcess` — id, `accountancyId`, `templateId` + versão copiada,
  `type` (value object com os três tipos pré-definidos), `targetClient`
  (texto livre — não há entidade `Client`), estado (`InProgress` /
  `Completed`), `createdAt`, `completedAt`.
- `ProcessStepInstance` — snapshot da Step do template: nome, ordem,
  obrigatório (sim/não), estado (`NotStarted` / `InProgress` / `Completed`),
  timestamps. Fechada automaticamente quando todas as suas
  `ProcessStepTaskInstance` obrigatórias estão resolvidas (concluídas ou
  saltadas).
- `ProcessStepTaskInstance` — snapshot da Task do template: nome, ordem,
  estado (`Pending` / `Completed` / `Skipped`), timestamps, auditoria (quem
  executou ou saltou, quando).
- `ProcessStepFieldInstance` — materialização runtime dos StepFields do
  template: label, tipo, obrigatório, valor preenchido, timestamp de
  preenchimento. Vinculado a uma `ProcessStepInstance`.
- `ProcessStepTaskFieldInstance` — materialização runtime dos TaskFields do
  template (campos configuráveis vinculados a uma Task específica). Mesma
  estrutura do `ProcessStepFieldInstance`, mas vinculado a uma
  `ProcessStepTaskInstance`. Essa entidade estava ausente do impact map
  original e corresponde aos TaskFields já existentes no WorkflowTemplate.
- `ProcessType` — value object representando os três tipos pré-definidos
  (`Opening`, `Amendment`, `Closure`). Não é enum fixo — a spec determina
  explicitamente que é value object para preparar a futura personalização por
  contabilidade.

Materialização: no momento da criação, o Process copia do template apenas as
steps/tasks cujo `appliesTo` inclui o tipo escolhido. Essa lógica depende de
que o WorkflowTemplate já suporte (ou passe a suportar) o atributo `appliesTo`
nos steps — **gap a validar com o backend**, ver seção "Gaps".

### Migrations

Sim. Nova migration criando as tabelas/schema do aggregate WorkflowProcess e
suas projeções. Nenhuma alteração em tabelas existentes de Template,
Accountancy ou Identity é exigida pela spec — com uma ressalva: se o
`appliesTo` por step no Template ainda não existir, será necessário adicionar
esse atributo no schema do Template (alteração localizada, não-destrutiva).

### Dependências internas

- **WorkflowTemplate** — leitura do template ativo da Accountancy no momento
  da criação do processo; cópia profunda para snapshot.
- **Accountancy** — resolução do `accountancyId` da sessão; filtragem de todos
  os endpoints pelo contorno multi-tenant.
- **Identity** — identificação do usuário corrente para auditoria de tarefas
  (quem concluiu, quem saltou). Observação: a spec não exige responsável do
  processo nem autor do processo — a dependência de Identity é limitada a
  auditoria de ações em tarefas.
- **Eventos de domínio** — emissão de eventos (ex.: `ProcessCreated`,
  `TaskCompleted`, `TaskSkipped`, `StepCompleted`, `ProcessCompleted`) prepara
  o terreno para o Dashboard (P2) e Notifications (P3). Não é exigência da
  spec do MVP, mas é fortemente recomendado estabelecer o padrão de eventos
  desde já para não ter retrabalho.

---

## `web.societiza` — O que precisa ser feito

### Páginas / rotas afetadas

Novas páginas:

- `/societario` (ou rota equivalente já reservada) — **Board kanban** único
  da Accountancy ativa. É a landing do societário operacional. Colunas
  representam as steps do template da contabilidade; cards representam
  processos.
- `/societario/processes/new` — Fluxo de criação de processo. Seleção de tipo
  (abertura / alteração / baixa) e do cliente-alvo. Pode ser modal ou rota
  dedicada — decisão de UI.
- `/societario/processes/[processId]` — Detalhe do processo. Mostra a step
  atual, tasks, campos preenchíveis e ações de concluir/saltar tarefa. Não
  há ação de avançar step nem de concluir processo — ambas são consequências
  automáticas das ações nas tasks.

Rotas existentes afetadas: nenhuma diretamente. Se já existir uma página stub
de societário apenas com o template, ela deve ser substituída ou absorvida
pelo board.

### Novos componentes

- `KanbanBoard` — orquestração do board. Responsável por renderizar colunas a
  partir das steps do template da Accountancy e distribuir os cards pelos
  processos retornados pela API.
- `KanbanColumn` — representa uma step (coluna). Recebe a lista de processos
  que estão atualmente nessa step.
- `ProcessCard` — card do processo no board. Exibe explicitamente: tipo do
  processo (abertura / alteração / baixa), cliente-alvo (texto livre),
  indicação visual de pendências. Sem drag-and-drop.
- `ProcessTypeBadge` — badge visual para o tipo do processo (deve existir
  como componente reutilizável).
- `ProcessTypeFilter` — filtro por tipo no topo do board. Default: todos.
- `CreateProcessDialog` (ou página) — formulário de criação: seleção de tipo
  e cliente-alvo em texto livre.
- `ProcessDetailView` — view de detalhe do processo com steps, tasks, fields
  de step e de task, auditoria.
- `StepPanel` — painel de visualização da step atual. Mostra nome, estado,
  progresso das tasks obrigatórias e os `ProcessStepFieldInstance`. **Não
  tem ação de avançar** — a step fecha sozinha quando suas tasks obrigatórias
  são resolvidas. É puramente uma camada de leitura e contexto sobre a step
  em andamento.
- `TaskItem` — item de task com ações "concluir" e "saltar". É o único
  ponto do UI que dispara avanço de step e conclusão de processo, ambos
  implicitamente via cascata no backend.
- `FieldInput` — inputs de campo respeitando os tipos suportados (Text,
  Boolean, Integer, Select, Date). Devem reaproveitar componentes já usados
  no WorkflowTemplate se existirem.

### Endpoints a registrar

Adicionar em `src/routes/endpoints.ts` (nomes exatos a combinar com o path
final definido pelo backend — registrar após a implementação):

- `workflowProcesses.list`
- `workflowProcesses.getById`
- `workflowProcesses.create`
- `workflowProcesses.completeTask`
- `workflowProcesses.skipTask`
- `workflowProcesses.updateStepField`
- `workflowProcesses.updateTaskField`

Não há endpoint `advanceStep` nem `complete` — avanço e conclusão são
implícitos no backend.

### Queries e mutations

Queries:
- `useWorkflowProcessesQuery({ accountancyId, type?, status? })` — lista do
  board. Deve invalidar sempre que uma mutation de mutação de estado ocorrer.
- `useWorkflowProcessQuery(processId)` — detalhe do processo.

Mutations:
- `useCreateWorkflowProcessMutation`
- `useCompleteTaskMutation`
- `useSkipTaskMutation`
- `useUpdateStepFieldMutation`
- `useUpdateTaskFieldMutation`

Não existem mutations `useAdvanceStepMutation` nem
`useCompleteWorkflowProcessMutation`: avanço de step e conclusão de processo
são efeitos colaterais no backend disparados pelo `complete`/`skip` de task.
Todas as mutations devem invalidar as queries de board e detalhe após
sucesso — em especial as de `completeTask` e `skipTask`, já que podem ter
promovido o processo para a próxima step ou concluído o processo inteiro
sem que o frontend soubesse a priori.

### Estados de UX relevantes

- **Loading** — shimmer do board com colunas vazias enquanto a lista carrega.
- **Empty state** — Accountancy ainda não tem processos: CTA claro de criar
  o primeiro processo, sem mensagens genéricas.
- **Error state** — board indisponível: mensagem acionável, opção de retry,
  manter contexto do que falhou.
- **Estado de processo concluído** — cards na coluna "concluído" devem ser
  visualmente distintos. Retenção é **load-bearing**: processos concluídos
  são a única memória estável da relação com o cliente-alvo (texto livre,
  sem entidade `Client`) e não podem ser apagados, arquivados
  destrutivamente ou removidos do board. O acúmulo ao longo do tempo é
  pergunta aberta (ver seção Gaps), mas qualquer tratamento de UX precisa
  ser puramente visual (filtro, ocultação, aba separada) e nunca destrutivo.
- **Tarefa saltada** — visualmente distinta de "concluída" e de "pendente".
  Exibe quem saltou e quando.
- **Step não iniciada / em andamento / concluída** — três estados visuais
  distintos. A transição entre eles é consequência automática das ações nas
  tasks, não de um clique em "avançar".
- **Transição automática de step** — quando `completeTask`/`skipTask` dispara
  o fechamento da step atual e a entrada na próxima, o UI precisa refletir
  a nova step sem exigir ação do usuário. Feedback visual claro para que a
  transição não pareça um "bug" ou um estado inconsistente.
- **Conclusão automática de processo** — quando a última step obrigatória
  fecha, o processo é movido para a coluna "concluído" automaticamente.
  Mesma exigência de feedback visual claro.
- **Filtro por tipo ativo** — indicação clara de qual filtro está aplicado.
- **Tipo visível no card** — requisito obrigatório da spec, não apenas
  estético.

---

## Dependências entre repos

**Backend primeiro.** O frontend depende do contrato de endpoints e da forma
do payload do processo (snapshot completo do template materializado). É
inviável começar o frontend antes de o backend ter pelo menos o endpoint de
criação e listagem funcionando com um payload estável.

**O que pode rodar em paralelo:**
- Design e protótipos de UI do board, card, detalhe de processo e fluxo de
  criação — não dependem do backend.
- Implementação de componentes de apresentação puros (`ProcessCard`,
  `ProcessTypeBadge`, `TaskItem`) com dados mockados.
- Definição do esquema de eventos de domínio do backend (preparatório para
  Dashboard/Notifications).

**O que é bloqueado até o backend entregar:**
- Integração real de queries e mutations.
- Finalização do registro de endpoints em `src/routes/endpoints.ts`.
- Geração do `integration.md` (acontece depois do backend implementar).

---

## Gaps e decisões abertas

1. **Atributo `appliesTo` no WorkflowTemplate.** A spec exige que steps
   marcados com `appliesTo` materializem apenas para os tipos relevantes no
   momento da criação do Process. Se o template atual não tem esse atributo,
   o backend precisa adicioná-lo (alteração localizada no Template) antes de
   implementar o Process. Validar com backend no início da implementação.

2. **Poluição da coluna "concluído" ao longo do tempo.** Pergunta aberta
   registrada na spec (seção 9). Não decidir no impact map. Backend e
   frontend devem conversar antes da implementação. Opções possíveis:
   ocultação temporal por default, filtro pré-aplicado, aba separada de
   histórico. **Restrição firme herdada da spec:** qualquer tratamento precisa
   ser puramente visual. Arquivamento destrutivo, exclusão ou remoção do
   board estão fora do produto (não apenas fora do MVP) porque a retenção
   de processos concluídos é load-bearing — é a única memória estável da
   relação com o cliente-alvo, que é texto livre.

3. **Eventos de domínio.** Não são exigência explícita da spec do MVP, mas
   são pré-condição natural do Dashboard (P2) e Notifications (P3). Decisão
   do backend: emitir eventos desde o MVP ou deixar para depois.
   Recomendação de produto: emitir desde o MVP para evitar retrabalho.

4. **Proteção de rotas / middleware de autenticação.** O `identity-context`
   documenta que os endpoints da API ainda não exigem autenticação
   (middleware de proteção não aplicado). Esta feature, por ser o coração
   operacional da contabilidade, deve ser protegida. Se a proteção global
   ainda não existir, pelo menos os endpoints do WorkflowProcess devem
   validar token JWT + pertencimento ao `accountancyId`. Sinalizar para o
   backend.

---

## Ordem de execução sugerida

1. **Backend — aggregate e materialização.** Criar o aggregate
   `WorkflowProcess`, o value object `ProcessType`, as entidades snapshot
   (`ProcessStepInstance`, `ProcessStepTaskInstance`,
   `ProcessStepFieldInstance`, `ProcessStepTaskFieldInstance`) e a lógica
   de materialização a partir do template ativo da Accountancy respeitando
   `appliesTo`. Migrations + projeções.
2. **Backend — endpoints de criação e leitura.** `POST /workflow-process`
   e `GET /workflow-process` (lista do board) + `GET /workflow-process/{id}`
   (detalhe). Esses três são o mínimo viável para o frontend começar a
   integrar.
3. **Frontend (paralelo a partir do passo 2) — board vazio + criação.**
   Implementar o KanbanBoard lendo a lista, o ProcessCard, o filtro por tipo
   e o fluxo de criação de processo (tipo + cliente-alvo em texto livre).
4. **Backend — mutações de execução com cascata automática.** Endpoints de
   concluir tarefa, saltar tarefa, preencher step fields, preencher task
   fields. Auditoria de quem/quando. Lógica implícita de cascata: ao
   resolver a última task obrigatória de uma `ProcessStepInstance`, a Step
   é fechada e o Process avança para a próxima; ao fechar a última Step
   obrigatória, o Process é concluído. Tudo no mesmo request, sem endpoint
   separado.
5. **Frontend — detalhe do processo.** ProcessDetailView completa com
   StepPanel (apenas visualização da step atual), TaskItem (ações concluir
   e saltar), FieldInput. UI precisa reagir à transição automática de step
   e à conclusão automática de processo quando o retorno do backend mostrar
   mudança de estado.
6. **Frontend — estado concluído.** Visual diferenciado da coluna
   "concluído", estados de tarefa saltada, processo concluído. Reforçar que
   processos concluídos permanecem no board (retenção load-bearing).
7. **Backend — eventos de domínio** (recomendado, não bloqueante do MVP).
8. **Integration doc.** Após o backend entregar, o agente de produto gera
   `specs/societario-workflow-process/integration.md` lendo o código real
   via MCP do GitHub e cruzando com a spec.

---

## Checklist de validação por repo

### `api.societiza`
- [ ] Aggregate `WorkflowProcess` criado e scoped por `accountancyId`.
- [ ] Entidades snapshot seguem a convenção canônica de nomes com sufixo
  `Instance`: `ProcessStepInstance`, `ProcessStepTaskInstance`,
  `ProcessStepFieldInstance`, `ProcessStepTaskFieldInstance`.
- [ ] `ProcessType` como value object com os três tipos pré-definidos
  (não enum fixo).
- [ ] Snapshot do template copiado no momento da criação, respeitando
  `appliesTo` por step, e materializando também os TaskFields como
  `ProcessStepTaskFieldInstance`.
- [ ] `targetClient` armazenado como texto livre. Nenhuma FK para entidade
  `Client` — essa entidade não existe.
- [ ] Processos concluídos são retidos permanentemente. Nenhum endpoint ou
  job apaga, arquiva de forma destrutiva ou remove processos concluídos
  (retenção load-bearing).
- [ ] Tipo visível no payload de listagem do board.
- [ ] Filtro por tipo suportado no endpoint de listagem.
- [ ] Estado do processo: em andamento / concluído.
- [ ] Avanço de `ProcessStepInstance` é **automático** ao resolver a última
  task obrigatória (concluída ou saltada). Não existe endpoint de
  `advance`. Não existe salto de Step inteira.
- [ ] Conclusão do `WorkflowProcess` é **automática** ao fechar a última
  Step obrigatória. Não existe endpoint de `complete` do processo.
- [ ] Auditoria de quem e quando em conclusão e salto de tarefa.
- [ ] Endpoints filtram por `accountancyId` da sessão.
- [ ] Endpoints exigem autenticação (ou a proteção global foi aplicada).
- [ ] Nenhum endpoint permite autor do processo como requisito (escopo
  negativo da spec).
- [ ] Nenhum endpoint permite responsável do processo (escopo negativo).

### `web.societiza`
- [ ] Board único renderizando colunas a partir das steps do template.
- [ ] Cards exibindo tipo, cliente-alvo (texto livre) e pendências.
- [ ] Sem drag-and-drop. **Nenhuma ação de "avançar step" no UI** — Step
  Panel é puramente visualização.
- [ ] **Nenhuma ação de "concluir processo" no UI** — conclusão é
  consequência automática da última step obrigatória fechar.
- [ ] Transição automática de step e conclusão automática de processo
  refletidas visualmente no detalhe e no board após `completeTask` /
  `skipTask`.
- [ ] Filtro por tipo disponível no topo do board. Default: todos.
- [ ] Fluxo de criação de processo pedindo tipo + cliente-alvo em texto
  livre (não pede template — há template único por contabilidade; não
  referencia entidade `Client` — ela não existe).
- [ ] Detalhe do processo com steps, tasks e fields do snapshot, incluindo
  `ProcessStepFieldInstance` e `ProcessStepTaskFieldInstance`.
- [ ] Tarefas suportam ação de concluir e ação de saltar; saltadas ficam
  visíveis.
- [ ] Campos preenchíveis respeitando tipos (Text, Boolean, Integer,
  Select, Date), tanto em nível de step quanto em nível de task.
- [ ] Estados de UX tratados: loading, empty, error.
- [ ] Processos concluídos permanecem visíveis na coluna "concluído".
  Qualquer tratamento de poluição visual (filtros, ocultação, aba
  histórico) é puramente visual e **jamais destrutivo** — retenção é
  load-bearing.
- [ ] Endpoints registrados em `src/routes/endpoints.ts` (sem
  `advanceStep`, sem `complete`).
- [ ] Queries invalidadas corretamente após mutations — em especial
  `completeTask` e `skipTask`, que podem ter disparado cascata de avanço
  de step ou conclusão de processo no backend.