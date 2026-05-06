---
status: approved
domain: societario
created_at: 2026-04-11
approved_at: 2026-04-11
revised_at: 2026-05-06
---

# WorkflowProcess — Board operacional do societário

## 1. Visão da feature

O WorkflowProcess é o board operacional onde o contador executa o dia a dia
do societário. É um kanban único, por contabilidade, onde cada card
representa um processo societário em andamento. Ao abrir o board, o contador
vê imediatamente onde cada processo está, o que precisa de atenção e o que
já foi concluído. O movimento entre etapas acontece de forma explícita — não
há drag-and-drop no uso operacional.

No MVP, o contador pode criar processos de três tipos: abertura, alteração
e baixa. O tipo é escolhido no momento da criação do processo e fica
explícito no card. O board mostra todos os processos juntos e permite
filtrar por tipo quando o contador quer focar em um recorte específico da
operação.

A feature permite, pela primeira vez, que a contabilidade tenha uma visão
operacional real e centralizada do que está acontecendo em cada processo
societário, substituindo planilhas, conversas espalhadas e memória humana
por um único lugar onde o estado de cada processo é visível, rastreável e
atualizado.

---

## 2. Problema que resolve

Hoje, o contador executa processos societários sem um lugar único para
acompanhar o andamento. Os dados de um mesmo processo ficam despejados em
planilhas, sites de prefeitura, portais de viabilidade, documentos
espalhados, conversas no WhatsApp e outros softwares que não compartilham
contexto entre si. Isso acontece diariamente, em todo processo societário
em andamento.

O workaround atual é manter planilhas paralelas e depender fortemente da
memória operacional do contador para saber em que etapa cada processo está,
o que já foi feito, o que está parado e o que precisa de atenção. Esse
workaround é insuficiente porque não escala, perde informação, não oferece
rastreabilidade, não deixa o estado visível para a equipe e quebra quando
há mais de uma pessoa operando a contabilidade.

A camada de template (WorkflowTemplate) já existe no produto, mas sem a
camada de execução (WorkflowProcess) o contador não tem onde operar os
processos de verdade. A feature é o elo que transforma a estrutura do
workflow em operação real.

---

## 3. Para quem é

**Beneficiário direto:** o contador (AccountingAdmin ou AccountingEmployee)
que executa processos societários dentro de uma contabilidade. É ele quem
cria os processos, acompanha o andamento, avança etapas, executa tarefas e
conclui processos.

**Beneficiário indireto:** a própria contabilidade como organização, que
passa a ter visibilidade compartilhada da operação — o estado deixa de
depender da memória de uma pessoa específica e passa a ser visível para
toda a equipe com acesso.

O contexto de uso é a rotina operacional diária do societário. Quando há
um novo cliente entrando, o contador cria um processo. Quando um processo
precisa avançar, o contador abre o board. Quando a equipe quer saber o que
está parado, o board é a fonte de verdade.

O cliente final (empresário) não é beneficiário desta feature no MVP —
não tem acesso ao board. Essa é uma capacidade prevista apenas no portal
do cliente, em horizonte futuro.

---

## 4. Como melhora a experiência atual

**Antes:**
O contador abre planilhas, vasculha conversas no WhatsApp, consulta
documentos espalhados e tenta reconstruir mentalmente em que etapa cada
processo está. Cada processo existe só na cabeça de quem o está tocando.
Não há maneira confiável de responder "como está o processo do cliente X?"
sem parar para investigar. Se o contador que conduz o processo fica
indisponível, a equipe perde contexto. O pior momento do fluxo é quando
vários processos precisam ser acompanhados ao mesmo tempo — não há visão
agregada.

**Depois:**
O contador abre o board e vê todos os processos da contabilidade, cada um
no seu lugar, com tipo explícito, etapa atual, tarefas pendentes e campos
preenchidos. Criar um processo novo é escolher o template, o tipo e
identificar o cliente. Avançar não é uma ação — é consequência: ao concluir
as tarefas obrigatórias da etapa atual, a etapa se fecha e o processo passa
automaticamente para a próxima. Concluir o processo é o resultado natural
de fechar todas as etapas. O estado do processo passa a existir no sistema,
não na memória. A equipe compartilha a mesma visão. Responder "como está o
processo do cliente X?" deixa de ser uma investigação e passa a ser uma
consulta direta.

---

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**

- **Accountancy** — todo processo pertence a uma contabilidade via
  `accountancyId`. O board só mostra os processos da Accountancy ativa.
- **WorkflowTemplate** — o processo nasce a partir de um template ativo.
  O contador informa o `TemplateId` ao criar o processo; o sistema valida
  que o template existe e está com status Active. A estrutura de steps,
  tasks e fields é copiada do template como fotografia imutável no momento
  da criação. O módulo WorkflowProcess declara a interface
  `IWorkflowTemplateDataService` (implementada na Web layer) para buscar
  a estrutura do template durante a criação.
- **Identity** — a autenticação define quem está operando e qual é a
  Accountancy ativa do usuário.

**O que esta feature habilita no futuro:**

- **Dashboard operacional (P2)** — é a fonte de dados para a leitura
  agregada do estado da operação. Sem WorkflowProcess não há dashboard.
- **Notificações (P3)** — eventos de mudança de estado em processos são
  o gatilho das notificações futuras.
- **Portal do cliente (futuro)** — o cliente verá uma visão derivada do
  processo societário dele.
- **Automação e importação (P4)** — precisam de uma camada de execução
  onde operar; o WorkflowProcess é essa camada.
- **Template Versioning (`societario-template-versioning`)** — o campo
  `TemplateVersion` registrado como string e o domain event
  `WorkflowProcessCompleted` publicado ao concluir são os pontos de extensão
  que o módulo de versionamento consumirá quando for construído.
- **Personalização de templates pela contabilidade (futuro)** — a estrutura
  de `TemplateId` explícito na criação prepara o terreno para quando a
  contabilidade puder usar múltiplos templates.

**Posição no produto:**

Feature core. É a peça central do MVP e fecha o ciclo do societário
transformando configuração em execução real. Sem ela, o produto não é
utilizável para a operação real da contabilidade.

---

## 6. O que não é (escopo negativo)

- **Não inclui** filtro de steps por tipo de processo na materialização
  (`appliesTo`). Todos os steps e tasks do template são materializados ao
  criar um processo, independentemente do ProcessType. O tipo classifica o
  processo para fins de visualização e filtragem no board — não determina
  quais steps existem no processo.
- **Não inclui** restrição de template único por contabilidade. O contador
  informa o `TemplateId` explicitamente na criação; qualquer template Active
  da plataforma pode ser usado.
- **Não inclui** drag-and-drop de cards entre colunas. A movimentação entre
  etapas é consequência automática da execução de tarefas — não é uma ação
  direta do contador.
- **Não inclui** criação, edição ou personalização de templates pela
  contabilidade. Os templates continuam sendo fornecidos pela Societiza.
- **Não inclui** criação, edição ou personalização dos tipos de processo
  pela contabilidade. No MVP, os três tipos são pré-definidos.
- **Não inclui** responsável principal do processo. Não há atribuição de
  dono no MVP — qualquer membro da contabilidade opera qualquer processo.
- **Não inclui** registro de quem criou o processo ou de quem executou cada
  tarefa. A auditoria pessoal (userId por ação) não está implementada no MVP.
- **Não inclui** justificativa obrigatória para skip ou revert de tarefa.
- **Não inclui** remoção, exclusão ou arquivamento destrutivo de processos
  concluídos — restrição permanente do produto: como o cliente-alvo é texto
  livre e não há entidade `Client`, os processos societários concluídos são
  a única memória estável da relação operacional com aquele cliente.
- **Não inclui** o histórico de processos concluídos como vista. O board
  mostra apenas processos InProgress; a vista de processos concluídos é
  feature separada (`societario-process-history`).
- **Não inclui** importação de processos existentes de outras fontes.
- **Não inclui** notificações, alertas de gargalo ou e-mails.
- **Não inclui** dashboard ou visão agregada de métricas.
- **Não inclui** acesso do cliente final ao processo.
- **Não inclui** filtros avançados no board por status, período ou critérios
  além do tipo. Esses filtros são capacidade futura.

---

## 7. Custos adicionais

Nenhum custo externo identificado.

A feature opera inteiramente dentro do ecossistema já existente da
Societiza. Não depende de APIs de terceiros, serviços externos, provedores
de e-mail, integrações governamentais ou infraestrutura adicional além da
que já suporta o WorkflowTemplate e a Accountancy.

---

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Board único por contabilidade | Há um único board operacional por Accountancy. Todos os processos convivem no mesmo kanban, independente do tipo. |
| Tipos de processo no MVP | Três tipos: Abertura, Alteração e Baixa (enum `ProcessType`). Pré-definidos — a contabilidade não cria nem edita tipos. |
| Tipo visível no card | Cada card exibe explicitamente qual é o tipo do processo. |
| Filtragem por tipo | O board permite filtrar por tipo. Por padrão, mostra todos os tipos juntos. |
| Seleção de template na criação | O contador informa o `TemplateId` ao criar o processo. O sistema valida que o template existe e está Active. Não há restrição de template único por contabilidade no MVP. |
| Materialização do processo | Todos os steps, tasks, fields e task fields do template são materializados como instâncias no momento da criação, independente do ProcessType. O ProcessType classifica o processo — não filtra quais steps são incluídos. |
| Versionamento do processo | Fotografia: o processo armazena o `TemplateVersion` como string no momento da criação. Mudanças posteriores no template não afetam processos existentes. |
| Materialização runtime — convenção de nomes | Entidades runtime usam sufixo `Instance`. Nomes canônicos: `ProcessStepInstance`, `ProcessStepTaskInstance`, `ProcessStepFieldInstance`, `ProcessStepTaskFieldInstance`. |
| Campos do processo | AccountancyId, TemplateId, TemplateVersion (string snapshot), ProcessType, TargetClient (texto livre), Status (InProgress/Completed), CompletedAt. |
| Identificação do cliente-alvo | No MVP, o cliente-alvo do processo é um campo de texto livre preenchido pelo contador. Não há referência a uma entidade `Client` — essa entidade não existe no MVP. |
| Responsável do processo | Não há responsável principal no MVP. |
| Auditoria de tarefa | A task registra os timestamps de `CompletedAt` e `SkippedAt`, mas não registra o userId de quem executou a ação. Auditoria pessoal é capacidade futura. |
| Estado do processo | Dois estados: InProgress e Completed. |
| Composição do board | O board mostra apenas processos InProgress da contabilidade ativa. A projeção MongoDB retorna `ProcessBoardItemResponse` agrupados por step, com ordenação pelo `Order` dos `ProcessStepInstance` materializados. |
| Saída do board na conclusão | Quando o processo conclui, ele sai do board e passa a ser acessível pela vista de histórico. O board não mantém uma coluna "concluído". |
| Publicação de conclusão | Ao concluir, o processo publica o domain event `WorkflowProcessCompleted` via EventBus (MassTransit/RabbitMQ), consumido pelo consumer de projeção do próprio módulo. Outros módulos (ex.: versionamento) poderão consumir esse evento quando construídos. |
| Avanço de step | Automático: quando todas as required tasks (IsOptional = false) do step estão resolvidas (Completed ou Skipped), o step auto-conclui e o próximo step passa para InProgress. Não existe ação explícita de "avançar step". |
| Critério de conclusão do processo | O processo conclui automaticamente quando todos os steps estão Completed. Segue a mesma mecânica do avanço de step, aplicada ao nível do processo. |
| Reversão de tarefa | Uma task Completed ou Skipped pode ser revertida (volta para Pending), desde que nenhum step downstream tenha progresso. Ao reverter, se o step estava Completed, volta para InProgress. |
| Preenchimento de campos | Qualquer campo (de step ou de task) pode ser preenchido a qualquer momento, independentemente do status do step ou da task. O `Value` é string; a interpretação por FieldType é responsabilidade do cliente. |
| Sequência de execução de tasks | Para completar ou pular uma task, a task obrigatória (IsOptional = false) com Order imediatamente anterior deve estar resolvida (Completed ou Skipped). |
| Retenção de processos concluídos | Processos concluídos permanecem armazenados permanentemente. Como o cliente-alvo é texto livre e não há entidade `Client`, esses processos são a única memória estável da relação operacional entre a contabilidade e aquele cliente. |
| Campos da `ProcessStepInstance` | Title, Order, Status (NotStarted/InProgress/Completed), CompletedAt. |
| Campos da `ProcessStepTaskInstance` | Title, Order, IsOptional, Status (Pending/Completed/Skipped/Reverted), CompletedAt, SkippedAt. |
| Campos de `ProcessStepFieldInstance` e `ProcessStepTaskFieldInstance` | Label, FieldType, Options, Order, Value (string), FilledAt. |

---

## 9. Perguntas abertas

- **Trajetória futura de personalização (fora do MVP).** Não é decisão do
  MVP, mas está no radar: tipo de processo virando entidade configurável;
  contabilidades criando e editando templates próprios; múltiplos templates
  por contabilidade com seleção na criação do processo. A estrutura de
  `TemplateId` explícito na criação existe justamente para não travar essa
  evolução.
