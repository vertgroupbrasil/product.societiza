---
status: approved
spec: societario-workflow-process
created_at: 2026-04-16
approved_at: 2026-05-06
revised_at: 2026-05-06
---

# WorkflowProcess — Lógica de Negócio

## 1. Fluxo por perfil

### SystemAdmin

SystemAdmin não interage com o board operacional de nenhuma contabilidade.
Sua atuação no societário é restrita à gestão dos workflow templates
fornecidos pela plataforma — fora do escopo do WorkflowProcess.

---

### AccountingAdmin

O AccountingAdmin é o administrador da contabilidade. Tem as mesmas
capacidades operacionais que o AccountingEmployee dentro do board, com a
diferença de que gerencia membros e configurações da contabilidade — o que
está fora do escopo desta feature.

**Criação de processo:**

1. Acessa o board da contabilidade ativa.
2. Inicia a criação de um novo processo.
3. Informa o `TemplateId` do template a ser usado.
4. Escolhe o tipo: Abertura, Alteração ou Baixa.
5. Preenche o cliente-alvo (texto livre).
6. Confirma a criação.
7. O sistema valida que o template está Active e materializa todos os steps,
   tasks, fields e task fields como instâncias — independente do ProcessType.
   O `TemplateVersion` é registrado como string snapshot.
8. A primeira step fica InProgress; as demais ficam NotStarted.
   Todas as tasks de todas as steps nascem como Pending.
9. O contador vê o novo card no board, na coluna da primeira step.

Se o template informado não existir ou não estiver Active: a criação é
recusada.

**Operação do board:**

1. Acessa o board e vê todos os processos InProgress da contabilidade
   em kanban, organizados por step.
2. Processos concluídos não aparecem no board — ficam na vista de
   histórico separada.
3. Pode filtrar o board por tipo de processo.
4. Abre um processo para ver seus detalhes: steps, status individual
   de cada step, tasks com status, campos e seus valores.

**Execução de tarefas:**

1. Dentro de qualquer step do processo, o contador vê as tasks.
2. Preenche os campos configuráveis de steps ou tasks a qualquer momento,
   independentemente do status do step ou da task.
3. **Concluir uma task** → task passa para Completed.
   - Pré-condições: task em Pending; step em NotStarted ou InProgress;
     task obrigatória (IsOptional = false) de Order imediatamente anterior
     deve estar resolvida (Completed ou Skipped).
4. **Pular uma task** → task passa para Skipped.
   - Pré-condições: task em Pending; IsOptional = true; task obrigatória
     de Order imediatamente anterior deve estar resolvida.
5. **Reverter uma task** → task volta para Pending.
   - Pré-condições: task em Completed ou Skipped; nenhum step downstream
     (Order superior ao step atual) pode ter progresso (tasks Completed ou
     Skipped em steps posteriores).
   - Ao reverter, se o step estava Completed, volta para InProgress.
6. Quando todas as required tasks (IsOptional = false) do step estiverem
   resolvidas (Completed ou Skipped), o step fecha automaticamente e o
   próximo step passa para InProgress — sem nenhuma ação adicional.
7. Quando todos os steps do processo estiverem Completed, o processo
   conclui automaticamente. O card sai do board.

**Ações não permitidas e o que acontece:**

- Tentar completar ou pular uma task cuja task obrigatória anterior não
  está resolvida: recusado com erro de pré-condição.
- Tentar pular uma task obrigatória (IsOptional = false): recusado.
- Tentar reverter uma task com progresso em steps downstream: recusado.
- Tentar criar processo com template inexistente ou não-Active: recusado.
- Tentar deletar ou arquivar um processo concluído: essa ação não existe.
- Tentar alterar o tipo de um processo já criado: essa ação não existe.

---

### AccountingEmployee

O AccountingEmployee tem as mesmas capacidades operacionais que o
AccountingAdmin dentro do board: cria processos, abre detalhes, completa,
pula e reverte tasks, preenche campos. Nenhuma operação do board é restrita
por perfil — qualquer membro da contabilidade pode operar qualquer processo.

As mesmas ações não permitidas descritas para o AccountingAdmin aplicam-se
igualmente ao AccountingEmployee.

---

## 2. Estados e transições

### Processo

**Estados:** InProgress · Completed

**Transições:**

- **Criação → InProgress:** imediato na criação. Não há estado intermediário.
- **InProgress → Completed:** automático quando todos os steps do processo
  estão Completed. O backend realiza a transição e publica o domain event
  `WorkflowProcessCompleted`. Nenhuma ação do contador.

**Restrições:**

- Completed é estado terminal. Processo concluído não volta a InProgress.
- Processo concluído é imutável. Nenhuma operação é aceita sobre ele.
- Processo concluído sai do board e passa a ser acessível pela vista de
  histórico.

---

### Step do processo (`ProcessStepInstance`)

**Estados:** NotStarted · InProgress · Completed

**Transições:**

- **Criação do processo:**
  - Primeira step → InProgress (imediato).
  - Demais steps → NotStarted.
- **InProgress → Completed:** automático quando todas as required tasks
  (IsOptional = false) do step estão resolvidas (Completed ou Skipped).
  Simultaneamente, a próxima step passa para InProgress.
- **NotStarted → InProgress:** automático quando a step anterior conclui.
- **Completed → InProgress:** ao reverter uma task, se o step estava
  Completed, ele volta para InProgress.

**Restrições:**

- Um step só fica InProgress quando o step anterior está Completed.
- Não existe ação explícita de "avançar step" — é consequência das tasks.
- Step com zero required tasks (IsOptional = false) conclui automaticamente
  assim que o processo a alcança, sem nenhuma ação do contador.
- Step revertido para InProgress pode ser auto-concluído novamente assim
  que suas required tasks forem resolvidas.

---

### Task do processo (`ProcessStepTaskInstance`)

**Estados:** Pending · Completed · Skipped · Reverted

**Transições:**

- **Criação do processo → todas as tasks nascem como Pending.**
- **Pending → Completed:** contador completa a task.
  - Pré-condições: step em NotStarted ou InProgress; task obrigatória
    (IsOptional = false) de Order imediatamente anterior deve estar Completed
    ou Skipped. Se não houver task obrigatória anterior, não há restrição.
- **Pending → Skipped:** contador pula a task.
  - Pré-condições: IsOptional = true; task obrigatória de Order
    imediatamente anterior deve estar Completed ou Skipped.
- **Completed → Reverted** ou **Skipped → Reverted:** contador reverte.
  - Pré-condição: nenhum step com Order superior ao step atual pode ter
    tasks Completed ou Skipped (sem progresso downstream).
  - A task volta para Pending após o revert.
  - Se o step estava Completed, volta para InProgress.

**O que conta como "resolvida" para fins de auto-conclusão de step:**

- Completed → conta.
- Skipped → conta.
- Pending / Reverted → não conta.

**Restrições:**

- Tasks com IsOptional = false não podem ser puladas.
- Completed e Skipped são reversíveis (via revert) enquanto não houver
  progresso downstream.
- Reverted é o estado transitório: a task está de volta a Pending e pode
  ser completada ou pulada novamente.

---

### Campo (`ProcessStepFieldInstance` / `ProcessStepTaskFieldInstance`)

**Estados:** sem valor · com valor (determinado pela presença de Value ≠ null)

**Transições:**

- Pode ser preenchido a qualquer momento, independentemente do status do
  step ou da task associada.
- O preenchimento registra `Value` (string) e `FilledAt` (timestamp).
- Um campo pode ser re-preenchido: sobrescreve `Value` e `FilledAt`.

**Restrições:**

- `Value` é sempre string. A interpretação por FieldType (Text, Boolean,
  Integer, Select, Date) é responsabilidade do cliente.
- O backend não valida o conteúdo do `Value` — nenhuma rejeição por formato.

---

## 3. Regras de negócio

### Criação do processo (`CreateWorkflowProcess`)

- O TemplateId deve referenciar um template existente com status Active.
- O sistema busca o template via `IWorkflowTemplateDataService` e mapeia
  para snapshots: `TemplateStepSnapshot`, `TemplateStepTaskSnapshot`,
  `TemplateStepFieldSnapshot`, `TemplateStepTaskFieldSnapshot`.
- Todos os steps, tasks, fields e task fields do template são materializados
  como instâncias no processo. Não há filtro por ProcessType.
- O ProcessType (Abertura, Alteração, Baixa) é registrado no processo para
  classificação e filtragem; não influencia quais instâncias são criadas.
- O `TemplateVersion` é capturado como string e gravado no processo.
- A primeira step fica InProgress; as demais ficam NotStarted.
- Todas as tasks nascem Pending.
- O processo pertence à contabilidade via AccountancyId e nunca cruza para
  outra contabilidade.

### Completar tarefa (`CompleteProcessTask`)

- Task deve estar Pending.
- Step associado deve estar NotStarted ou InProgress.
- A task obrigatória (IsOptional = false) de Order imediatamente anterior
  (Order - 1) deve estar Completed ou Skipped. Se não existir task
  obrigatória anterior, não há restrição sequencial.
- Ao completar, se todas as required tasks do step estiverem resolvidas
  → step auto-completa (InProgress → Completed) e próxima step passa para
  InProgress.
- Se todos os steps do processo estiverem Completed → processo auto-completa
  (InProgress → Completed) e o domain event `WorkflowProcessCompleted` é
  publicado.

### Pular tarefa (`SkipProcessTask`)

- Task deve estar Pending.
- IsOptional deve ser true. Tasks obrigatórias (IsOptional = false) não
  podem ser puladas.
- A task obrigatória de Order imediatamente anterior deve estar Completed
  ou Skipped. Se não existir, não há restrição sequencial.
- Ao pular, a mesma lógica de auto-conclusão de step e processo aplica-se.

### Reverter tarefa (`RevertProcessTask`)

- Task deve estar Completed ou Skipped.
- Nenhum step com Order superior ao step atual pode ter progresso — nenhuma
  task Completed ou Skipped em steps downstream.
- Ao reverter: task volta para Pending.
- Se o step estava Completed: step volta para InProgress.
- Não há operação de "reverter step inteiro" ou "reverter processo" —
  apenas task individual.

### Preencher campo (`FillProcessStepField` / `FillProcessTaskField`)

- Qualquer campo pode ser preenchido a qualquer momento, independente do
  status do step ou da task.
- `Value` é string. Não há validação de formato — o FieldType orienta
  apenas a renderização no cliente.
- O preenchimento registra `FilledAt` com o timestamp da operação.
- Re-preenchimento sobrescreve os valores anteriores.

### Geral

- O board exibe apenas processos InProgress da contabilidade ativa do
  usuário autenticado.
- Processos concluídos são permanentes e imutáveis. Qualquer operação sobre
  um processo Completed é rejeitada.
- Não há responsável atribuído a um processo.
- Não há registro de userId em tasks (quem completou, pulou ou reverteu).
  Os timestamps (CompletedAt, SkippedAt) são registrados; o ator não.

---

### Tabela de pré-condições por operação

| Operação | Pré-condição |
|---|---|
| `CompleteProcessTask` | Task Pending; step NotStarted ou InProgress; task obrigatória anterior (Order-1) Completed ou Skipped |
| `SkipProcessTask` | Task Pending; IsOptional = true; task obrigatória anterior (Order-1) Completed ou Skipped |
| `RevertProcessTask` | Task Completed ou Skipped; sem progresso em steps downstream |
| `FillProcessStepField` | Nenhuma — qualquer campo pode ser preenchido a qualquer momento |
| `FillProcessTaskField` | Nenhuma — qualquer campo pode ser preenchido a qualquer momento |
| `CreateWorkflowProcess` | Template Active; TargetClient preenchido |

---

## 4. Pontos de integração

```
Frontend precisa saber:
  - AccountancyId do usuário autenticado — para buscar o board correto
  - Lista de templates Active disponíveis — para oferecer ao contador
    na criação do processo (TemplateId é informado explicitamente)
  - Estado atual do processo (InProgress) e ProcessType — para renderizar
    o card corretamente no kanban
  - TargetClient (texto livre) — para exibir no card
  - Lista de steps com Status individual (NotStarted/InProgress/Completed),
    Order e CompletedAt
  - Tasks de cada step com Status (Pending/Completed/Skipped/Reverted),
    IsOptional, Order, CompletedAt, SkippedAt
  - Campos de step e de task com FieldType, Options, Value, FilledAt
  - Que completar ou pular uma task pode desencadear auto-conclusão de step
    e de processo — recarregar estado completo do processo após cada ação
  - Que reverter uma task pode reabrir o step — recarregar estado completo
  - Que skip é permitido apenas para IsOptional = true — desabilitar ação
    de skip para tasks obrigatórias na interface
  - Que campos podem ser preenchidos a qualquer momento — não há bloqueio
    por status de step ou task
  - Que a pré-condição de task obrigatória anterior pode causar rejeição —
    exibir mensagem contextualizada ao contador
  - Que revert é bloqueado quando há progresso downstream — exibir motivo

Backend precisa garantir:
  - Que CreateWorkflowProcess busca o template via IWorkflowTemplateDataService
    e valida que está Active antes de materializar
  - Que todos os steps/tasks/fields são materializados independentemente do
    ProcessType
  - Que o TemplateVersion é capturado como string no momento da criação
  - Que CompleteProcessTask verifica: task Pending; step NotStarted ou
    InProgress; task obrigatória anterior resolvida
  - Que SkipProcessTask verifica: IsOptional = true; task obrigatória
    anterior resolvida
  - Que RevertProcessTask verifica: task Completed ou Skipped; sem progresso
    downstream; reverte step se estava Completed
  - Que auto-conclusão de step e processo ocorre na mesma operação de
    complete ou skip — sem chamada separada do frontend
  - Que FillProcessStepField e FillProcessTaskField não verificam status do
    step ou da task — qualquer campo é preenchível a qualquer momento
  - Que o board retorna apenas processos InProgress da contabilidade do
    usuário autenticado, sem exceção
  - Que processos concluídos são imutáveis — qualquer operação é rejeitada
  - Que ao concluir o processo, o domain event WorkflowProcessCompleted é
    publicado via EventBus (MassTransit/RabbitMQ)
```

---

## 5. Casos de borda

**Template arquivado após criação do processo**
O processo possui a fotografia do template (TemplateVersion como string).
O arquivamento posterior do template não afeta nenhum processo existente.
Todos continuam operacionais com a estrutura congelada. Novos processos não
podem ser criados a partir do template arquivado.

**Step sem required tasks (zero tasks com IsOptional = false)**
Uma step sem required tasks conclui automaticamente assim que o processo a
alcança, sem nenhuma ação do contador. Se todas as steps não tiverem required
tasks, o processo pode concluir automaticamente logo após a criação — esse
é um comportamento válido do sistema.

**Cliente-alvo em branco**
O `TargetClient` é validado como obrigatório na camada de Application
(validação do comando de criação). A criação é recusada se o campo estiver
vazio.

**Campos não preenchidos ao completar ou pular tarefa**
Completar ou pular uma task não verifica se seus campos foram preenchidos.
O preenchimento de campos é independente do ciclo de vida da task. O
`FieldType` orienta a renderização; o backend não interpreta o `Value` para
validar formato ou obrigatoriedade.

**Reverter task com step já auto-concluído**
Se o step estava Completed (auto-concluiu), reverter uma task desfaz essa
conclusão: step volta para InProgress. Para que o revert seja permitido,
nenhum step de Order superior pode ter progresso. O contador deve reverter
tasks em ordem reversa caso queira retroceder múltiplos steps.

**Revert em cadeia**
Não há operação de "reverter step inteiro". Para reabrir um step concluído,
o contador deve reverter individualmente as tasks Completed ou Skipped nele.
Cada revert individual verifica que não há progresso downstream.

**Task revertida em step que estava Completed reabre o step**
Ao reverter a última task resolvida de um step Completed, o step volta para
InProgress. Se o step seguinte havia iniciado (InProgress) como consequência
do auto-avanço, o revert é bloqueado — o step downstream já tem o step em
InProgress, o que configura progresso downstream.

**Processo com todas as tasks de todos os steps opcionais**
Se todas as tasks de todos os steps têm IsOptional = true, cada step
auto-conclui ao ser alcançado (sem required tasks bloqueando). O processo
pode concluir automaticamente sem nenhuma ação do contador após a criação.
Esse comportamento é válido — a responsabilidade de evitar templates sem
required tasks pertence à validação de ativação do WorkflowTemplate.
