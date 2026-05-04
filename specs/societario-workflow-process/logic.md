---
status: draft
spec: societario-workflow-process
created_at: 2026-04-16
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
3. Escolhe o tipo: abertura, alteração ou baixa.
4. Preenche o cliente-alvo (nome do cliente em texto livre).
5. Confirma a criação.
6. O processo nasce com a primeira etapa em andamento. As demais etapas
   nascem como não iniciadas. Todas as tarefas da etapa ativa nascem como
   pendentes.
7. O contador vê o novo card no board, na coluna da primeira etapa.

Se não houver template ativo na contabilidade: a opção de criar processo
não está disponível. O contador vê uma indicação de que não há template
ativo.

**Operação do board:**

1. Acessa o board e vê todos os processos InProgress da contabilidade em
   kanban. Processos concluídos não aparecem aqui — ficam na vista de
   histórico separada.
2. As colunas do board são a união dos steps em que existem processos ativos.
   A ordem segue a versão `Active` mais recente do template como referência
   canônica. Steps presentes apenas em versões anteriores (Superseded)
   aparecem ao final, na ordem relativa original. O contador não percebe
   versões — só vê processos nos seus steps.
3. Pode filtrar o board por tipo de processo (abertura, alteração, baixa).
4. Abre um processo para ver seus detalhes: etapas, etapa atual, tarefas
   da etapa ativa, campos configuráveis e seus valores.

**Execução de tarefas:**

1. Dentro da etapa ativa do processo, o contador vê as tarefas pendentes.
2. Preenche os campos configuráveis da etapa ou da tarefa, se houver.
3. Conclui uma tarefa → tarefa passa para "concluída".
4. Ou salta uma tarefa → tarefa passa para "saltada" com registro de quem
   e quando.
5. Quando todas as tarefas obrigatórias da etapa ativa estiverem resolvidas
   (concluídas ou saltadas), a etapa fecha automaticamente e a próxima etapa
   ativa abre — sem nenhuma ação adicional do contador.
6. Quando todas as etapas obrigatórias do processo estiverem concluídas, o
   processo conclui automaticamente. O card sai do board e passa a estar
   acessível apenas pela vista de histórico.

**Ações não permitidas e o que acontece:**

- Tentar operar tarefas de uma etapa que não está ativa: ação bloqueada.
  O contador pode ver as etapas passadas e futuras mas não pode modificá-las.
- Tentar deletar ou arquivar um processo concluído: essa ação não existe.
  Processos concluídos são permanentes e ficam acessíveis pela vista de
  histórico (ver spec `societario-process-history`), não pelo board.
- Tentar alterar o tipo de um processo já criado: essa ação não existe.
  O tipo é definido na criação e imutável.

---

### AccountingEmployee

O AccountingEmployee tem as mesmas capacidades operacionais que o
AccountingAdmin dentro do board: cria processos, abre detalhes, conclui
e salta tarefas, preenche campos. Nenhuma operação do board é restrita por
perfil — qualquer membro da contabilidade pode operar qualquer processo.

As mesmas ações não permitidas descritas para o AccountingAdmin aplicam-se
igualmente ao AccountingEmployee.

---

## 2. Estados e transições

### Processo

**Estados:** em andamento · concluído

**Transições:**

- **Criação → em andamento:** imediato na criação. Não há estado intermediário.
- **Em andamento → concluído:** automático quando todas as etapas obrigatórias
  do processo estão concluídas. O backend realiza a transição, remove o
  processo do board ativo e publica o integration event
  `WorkflowProcessCompleted`. Nenhuma ação do contador.

**Restrições:**

- "Concluído" é estado terminal. Processo concluído não volta a "em andamento".
- Processo concluído é imutável. Não pode ser editado, deletado ou arquivado.
- Processo concluído sai do board e passa a ser acessível apenas pela vista
  de histórico (ver spec `societario-process-history`).

---

### Etapa do processo

**Estados:** não iniciada · em andamento · concluída

**Transições:**

- **Criação do processo → primeira etapa fica em andamento:** imediato na
  criação. Todas as demais etapas nascem como "não iniciadas".
- **Em andamento → concluída:** automático quando todas as tarefas obrigatórias
  da etapa estão resolvidas (concluídas ou saltadas). O backend realiza a
  transição e simultaneamente move a próxima etapa para "em andamento".
- **Não iniciada → em andamento:** automático quando a etapa anterior conclui.

**Restrições:**

- Uma etapa só fica "em andamento" se a etapa anterior foi concluída.
- Não há salto de etapa inteira — o contador age no nível das tarefas.
- Etapa "concluída" não volta a estados anteriores.
- Etapa sem tarefas obrigatórias conclui automaticamente assim que o processo
  chega a ela, sem nenhuma ação do contador.

---

### Tarefa da etapa

**Estados:** pendente · concluída · saltada

**Transições:**

- **Criação do processo → todas as tarefas nascem como pendentes.**
- **Pendente → concluída:** o contador marca a tarefa como concluída.
- **Pendente → saltada:** o contador marca a tarefa como saltada manualmente.

**Restrições:**

- Tarefas de etapas que não estão "em andamento" não podem ser modificadas.
- Tarefa "concluída" ou "saltada" não volta ao estado "pendente".
- Tarefa saltada é registrada: quem saltou e quando. Justificativa não é
  obrigatória.
- Tarefa saltada conta como "resolvida" para fins de avanço automático da etapa.

---

### Campo de etapa / campo de tarefa

**Estados:** não preenchido · preenchido

**Transições:**

- Preenchido pelo contador a qualquer momento enquanto a etapa está ativa.
- Valor salvo no preenchimento, com timestamp.

**Restrições:**

- Campos de etapas que não estão ativas não podem ser preenchidos.
- Se um campo é do tipo Select, apenas os valores configurados na lista de
  opções do template são válidos.

---

## 3. Regras de negócio

- Um processo pertence à contabilidade em que foi criado via identificador
  da contabilidade. Nunca cruza para outra contabilidade.
- Um processo só pode ser criado se a contabilidade tiver um template ativo.
- O tipo do processo (abertura, alteração, baixa) é definido na criação e
  não pode ser alterado.
- No momento da criação, apenas as etapas e tarefas do template marcadas
  para o tipo escolhido são incluídas no processo. Etapas e tarefas fora do
  tipo não existem no processo.
- O template é congelado no processo no momento da criação com sua versão
  atual. Mudanças posteriores no template não afetam processos já existentes.
- O avanço de etapa e a conclusão do processo são automáticos — o backend
  realiza as transições como consequência de ações do contador sobre tarefas.
  Não existe ação explícita de "avançar etapa" pelo contador.
- Uma etapa conclui quando todas as suas tarefas obrigatórias estão
  resolvidas (concluídas ou saltadas).
- O processo conclui quando todas as suas etapas obrigatórias estão concluídas.
- Tarefas de etapas que não estão ativas não podem ser modificadas.
- Processos concluídos são permanentes. Não podem ser deletados, arquivados
  ou modificados.
- O board exibe apenas processos InProgress da contabilidade ativa do usuário.
  Processos concluídos saem do board e ficam na vista de histórico. Ao trocar
  de contabilidade, o contexto do board muda completamente.
- As colunas do board são a união dos steps presentes nos processos InProgress.
  A ordem segue como referência canônica a versão `Active` mais recente do
  template; steps presentes apenas em versões Superseded aparecem ao final na
  ordem relativa original. Quando uma coluna esvazia e o template
  correspondente é arquivado, a coluna desaparece naturalmente.
- A projeção de leitura do board (MongoDB) é recalculada quando: um processo
  muda de step, um template é ativado (recalcula ordem canônica) ou um
  template é arquivado (remove colunas vazias).
- Ao concluir, o processo publica o integration event `WorkflowProcessCompleted`
  via RabbitMQ/MassTransit. Outras features consomem esse evento sem acoplar
  ao módulo de processo (ex.: archive automático de templates Superseded).
- A interface `IWorkflowProcessDataService` (declarada em WorkflowTemplate,
  implementada na Web layer pelo módulo WorkflowProcess) expõe a verificação
  "há processos InProgress para este TemplateId?", consumida por
  WorkflowTemplate para bloquear archive manual e disparar archive automático.
- Não há responsável atribuído a um processo. Qualquer membro da contabilidade
  pode operar qualquer processo.
- A auditoria de tarefa (quem e quando concluiu ou saltou) é registrada
  automaticamente sem nenhum input adicional do contador além da própria ação.

---

## 4. Pontos de integração

```
Frontend precisa saber:
  - Estado atual do processo (em andamento) e tipo, para renderizar
    o card corretamente no kanban (processos concluídos não aparecem no board)
  - Cliente-alvo do processo (texto livre), para exibir no card
  - Lista de etapas do processo com estado individual (não iniciada / em
    andamento / concluída) e se cada uma é obrigatória, para exibir o
    progresso do processo
  - Tarefas da etapa ativa com estado (pendente / concluída / saltada),
    se são obrigatórias, e seus campos configuráveis com valores preenchidos
  - Campos da etapa ativa com valores preenchidos
  - Registro de quem e quando concluiu ou saltou cada tarefa (para auditoria
    visível ao contador)
  - Se a contabilidade tem template ativo (para habilitar ou desabilitar a
    criação de processo)
  - Quais tipos de processo (abertura, alteração, baixa) o template ativo
    suporta (para exibir apenas as opções válidas na criação)
  - Estrutura de colunas do board já resolvida pela projeção: união de steps
    de processos InProgress, ordenados pela versão Active mais recente, com
    steps de versões Superseded ao final. O frontend não precisa conhecer
    versões de template — só renderiza a lista de colunas que o backend
    devolve.

Backend precisa garantir:
  - Que a transição de estado de etapa e de processo acontece na mesma
    operação de conclusão ou salto de tarefa — o frontend não faz chamada
    separada para "avançar etapa"
  - Que a resposta de conclusão/salto de tarefa inclui o estado atualizado
    do processo inteiro (etapas + estado do processo) para que o frontend
    reflita a mudança sem precisar de nova requisição
  - Que o board retorna apenas processos InProgress da contabilidade do usuário
    autenticado, sem exceção
  - Que tentativas de operar tarefas em etapas inativas são rejeitadas com
    indicação clara do motivo
  - Que tentativas de criar processo sem template ativo são rejeitadas
  - Que a versão do template é registrada no processo no momento da criação
  - Que processos concluídos são imutáveis — qualquer operação sobre eles
    é rejeitada
  - Que a auditoria de tarefa é preenchida automaticamente com o usuário
    autenticado que realizou a ação, sem depender de input do frontend
  - Que a projeção MongoDB do board é atualizada nos eventos: processo muda
    de step, template é ativado, template é arquivado
  - Que ao concluir o processo, o integration event `WorkflowProcessCompleted`
    é publicado via RabbitMQ/MassTransit
  - Que a interface `IWorkflowProcessDataService` (Web layer) está disponível
    para que WorkflowTemplate consulte "há processos InProgress para este
    TemplateId?"
```

---

## 5. Casos de borda

**Template arquivado após criação do processo**
O processo possui a fotografia do template na versão em que foi criado. O
arquivamento posterior do template não afeta nenhum processo existente. Todos
continuam operacionais com a versão congelada. Novos processos não podem mais
ser criados a partir do template arquivado.

**Etapa sem tarefas obrigatórias**
Uma etapa pode ter zero tarefas obrigatórias — seja porque tem apenas tarefas
opcionais, seja porque não tem tarefas para o tipo do processo. Nesse caso, a
etapa conclui automaticamente assim que o processo chega a ela, sem nenhuma
ação do contador. O processo avança imediatamente para a próxima etapa
obrigatória.

**Processos de versões diferentes do template no mesmo board**
Quando uma nova versão do template é ativada, processos InProgress nascidos
da versão anterior continuam executando a estrutura original. O board mostra
todos juntos: as colunas são a união dos steps presentes nesses processos,
ordenadas pela versão `Active` mais recente como referência canônica. Steps
que existem apenas na versão Superseded aparecem ao final na ordem relativa
original. O contador não percebe a coexistência de versões — só vê processos
nos seus steps.

**Coluna esvaziando após conclusão de último processo**
Quando o último processo InProgress de um step (presente apenas em template
Superseded) conclui, a coluna fica vazia. Se o template Superseded também é
arquivado nesse momento (regra de archive automático em
`societario-template-versioning`), a coluna desaparece do board na próxima
recomputação da projeção.

**Cliente-alvo em branco na criação**
`[pergunta aberta]` A spec indica que criar um processo inclui "identificar o
cliente", sugerindo que o campo é obrigatório. Mas não há declaração explícita
de obrigatoriedade. Precisa de decisão antes da implementação: o campo
cliente-alvo é obrigatório para criar o processo ou pode ficar em branco?

**Campos obrigatórios não preenchidos ao concluir tarefa**
`[pergunta aberta]` O sistema permite marcar uma tarefa como concluída mesmo
que seus campos obrigatórios não estejam preenchidos? E ao saltar uma tarefa?
Se não, o backend deve rejeitar a operação e o frontend precisa mostrar quais
campos faltam. Se sim, campos obrigatórios são apenas uma indicação visual.
Precisa de decisão antes da implementação.

**Processo com todas as etapas opcionais para o tipo escolhido**
`[pergunta aberta]` Se o template não tiver nenhuma etapa marcada como
obrigatória para o tipo de processo selecionado, o processo concluiria
automaticamente logo após a criação. Isso é comportamento esperado ou uma
inconsistência que deve ser prevenida? Precisa de validação ao criar o
processo ou ao configurar o template.

**Premissa dos ~70% de etapas compartilhadas entre tipos**
A hipótese de que abertura, alteração e baixa compartilham a maior parte das
etapas não foi validada na operação real. Se os fluxos divergirem mais do que
o esperado, a estrutura de materialização por tipo (`appliesTo`) pode precisar
de revisão de design antes de escalar para outros templates.