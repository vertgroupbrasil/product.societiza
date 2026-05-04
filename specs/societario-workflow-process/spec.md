---
status: approved
domain: societario
created_at: 2026-04-11
approved_at: 2026-04-11
revised_at: 2026-05-03
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
preenchidos. Criar um processo novo é escolher o tipo e identificar o
cliente. Avançar não é uma ação — é consequência: ao concluir as tarefas
obrigatórias da etapa atual, a etapa se fecha e o processo passa
automaticamente para a próxima. Concluir o processo é o resultado natural
de fechar todas as etapas obrigatórias. O estado do processo passa a existir no
sistema, não na memória. A equipe compartilha a mesma visão. Responder
"como está o processo do cliente X?" deixa de ser uma investigação e passa
a ser uma consulta direta.

---

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**

- **Accountancy** — todo processo pertence a uma contabilidade via
  `accountancyId`. O board só mostra os processos da Accountancy ativa.
- **WorkflowTemplate** — o processo nasce a partir do template único da
  contabilidade. A estrutura de etapas, tarefas e campos que o processo vai
  seguir vem do template.
- **WorkflowTemplate Versioning** (`societario-template-versioning`) — a
  composição de colunas do board precisa lidar com processos que nasceram
  de versões diferentes do template (Active e Superseded coexistindo). A
  ordenação canônica das colunas usa a versão `Active` mais recente como
  referência. A interface `IWorkflowProcessDataService` (declarada em
  WorkflowTemplate, implementada na Web layer pelo módulo WorkflowProcess)
  expõe a verificação "há processos InProgress?" usada por versionamento.
  A publicação de `WorkflowProcessCompleted` é consumida por versionamento
  para arquivar templates Superseded automaticamente.
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
- **Personalização de templates pela contabilidade (futuro)** — o desenho
  de tipo como value object (ver seção 8) prepara o terreno para quando a
  contabilidade puder criar e editar seus próprios templates e tipos de
  processo.

**Posição no produto:**

Feature core. É a peça central do MVP e fecha o ciclo do societário
transformando configuração em execução real. Sem ela, o produto não é
utilizável para a operação real da contabilidade.

---

## 6. O que não é (escopo negativo)

- **Não inclui** drag-and-drop de cards entre colunas. A movimentação entre
  etapas é explícita — evita operações acidentais e mantém o board como
  reflexo fiel do estado real do processo.
- **Não inclui** escolha de template pelo contador no momento da criação
  do processo. No MVP há um template único por contabilidade — o contador
  só escolhe o tipo. A escolha de template é uma capacidade futura.
- **Não inclui** criação, edição ou personalização de templates pela
  contabilidade. Os templates continuam sendo fornecidos pela Societiza.
- **Não inclui** criação, edição ou personalização dos tipos de processo
  pela contabilidade. No MVP, os três tipos são pré-definidos.
- **Não inclui** responsável principal do processo. Não há atribuição de
  dono no MVP — qualquer membro da contabilidade opera qualquer processo.
- **Não inclui** registro de quem criou o processo. A rastreabilidade
  pessoal do autor do processo fica fora do MVP.
- **Não inclui** justificativa obrigatória para salto de tarefa. O salto é
  rastreado por quem e quando, mas o motivo não é um campo exigido.
- **Não inclui** remoção, exclusão ou arquivamento destrutivo de processos
  concluídos — e isso não é só "fora do MVP", é uma restrição permanente
  do produto: processos concluídos são a única memória estável da relação
  com o cliente-alvo (que é texto livre) e precisam ser preservados.
  Processos concluídos saem do board ativo e passam a viver na vista de
  histórico (ver spec `societario-process-history`).
- **Não inclui** o histórico de processos concluídos como vista. O board
  mostra apenas processos InProgress; a vista de processos concluídos é
  feature separada (`societario-process-history`).
- **Não inclui** importação de processos existentes de outras fontes.
  Importação é uma capacidade futura (P4).
- **Não inclui** notificações, alertas de gargalo ou e-mails de
  acompanhamento. Notificações são pós-MVP (P3).
- **Não inclui** dashboard ou visão agregada de métricas. Dashboard é P2.
- **Não inclui** acesso do cliente final ao processo. Portal do cliente é
  capacidade futura.

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
| Tipos de processo no MVP | Três tipos: abertura, alteração e baixa. |
| Estrutura dos tipos de processo | Tipo é um value object com os três tipos pré-definidos no MVP — não é enum fixo. A estrutura prepara a trajetória futura em que a contabilidade poderá personalizar tipos e templates próprios. |
| Tipo visível no card | Cada card exibe explicitamente qual é o tipo do processo. |
| Filtragem por tipo | O board permite filtrar por tipo. Por padrão, mostra todos os tipos juntos. |
| Template no MVP | Um único template por contabilidade. O contador não escolhe template — só escolhe o tipo do processo. |
| Compartilhamento entre fluxos | Os três tipos compartilham grande parte dos passos do template. A premissa de trabalho é ~70% de passos compartilhados, a ser validada na operação real. |
| Materialização por tipo | Steps marcados com `appliesTo` no template materializam apenas para os tipos relevantes quando o Process nasce. |
| Versionamento do Process | Versionamento por fotografia: o Process copia o template (incluindo versão) no momento da criação e não reflete mudanças posteriores do template. |
| Criação do processo | Criação manual pelo contador. Não há criação automática no MVP. |
| Materialização runtime — convenção de nomes | As entidades runtime criadas a partir do template usam o sufixo `Instance` para deixar explícito que são materializações, não definições. Nomes canônicos: `ProcessStepInstance`, `ProcessStepTaskInstance`, `ProcessStepFieldInstance`, `ProcessStepTaskFieldInstance`. |
| Campos do Process | Referência ao template (com versão), tipo, cliente-alvo (texto livre — ver linha dedicada abaixo), estado (em andamento / concluído), timestamps de criação e conclusão. |
| Identificação do cliente-alvo | No MVP, o cliente-alvo do processo é um campo de **texto livre** preenchido pelo contador. Não há referência a uma entidade `Client` — essa entidade não existe no MVP. |
| Responsável do processo | Não há responsável principal no MVP. |
| Autor do processo | Não é registrado no MVP quem criou o processo. |
| Estado do Process | Dois estados: em andamento e concluído. |
| Composição do board | O board mostra apenas processos InProgress. As colunas são a união de todos os steps presentes nesses processos — não derivam diretamente do template. Cada `ProcessStepInstance` carrega seu próprio `CurrentStepTitle` e `StepOrder` materializados, então o board pode renderizar processos de versões diferentes do template lado a lado sem inconsistência. |
| Ordenação das colunas do board | A ordem das colunas usa o template `Active` mais recente da linhagem como referência canônica. Steps presentes apenas em versões anteriores (Superseded) aparecem ao final, na sua ordem relativa original. Quando uma coluna esvazia (último processo daquele step concluiu) e o template correspondente é arquivado, a coluna desaparece naturalmente. |
| Projeção de leitura do board | O board é servido por uma projeção MongoDB recalculada quando: um processo muda de step, um template é ativado (recalcula ordem canônica) ou um template é arquivado (remove colunas vazias). |
| Saída do board na conclusão | Quando o processo conclui, ele sai do board e passa a estar acessível pela vista de histórico (`societario-process-history`). O board não mantém uma coluna "concluído". |
| Publicação de conclusão | Ao concluir, o processo publica o integration event `WorkflowProcessCompleted` (RabbitMQ/MassTransit). Outras features consomem (ex.: archive automático de templates Superseded em `societario-template-versioning`). |
| Avanço de etapa | Avanço é **automático**, não uma ação. Ao concluir (ou saltar) todas as Tasks obrigatórias de uma `ProcessStepInstance`, a Step é marcada como concluída e o Process passa automaticamente para a próxima Step na ordem. Não existe ação/endpoint explícito de "avançar etapa". Não existe salto de Step inteira — se o contador precisa pular trabalho, ele age no nível das Tasks. |
| Critério de conclusão do Process | O Process é concluído automaticamente quando todas as `ProcessStepInstance` obrigatórias estão concluídas. Segue exatamente a mesma mecânica do avanço de etapa, aplicada ao nível do Process. |
| Retenção de processos concluídos | Processos concluídos **permanecem armazenados permanentemente**. Essa retenção é requisito **load-bearing**, não apenas uma escolha operacional: como o cliente-alvo é texto livre e não há entidade `Client`, os processos societários concluídos são a única memória estável da relação operacional entre a contabilidade e aquele cliente. Perder processos concluídos significa perder o histórico societário do cliente. A vista para consultar processos concluídos é separada do board e está descrita na spec `societario-process-history`. |
| Campos da `ProcessStepInstance` | Nome e ordem copiados do template, estado (não iniciado / em andamento / concluído), timestamps, indicação de obrigatório sim/não. |
| Campos da `ProcessStepTaskInstance` | Nome e ordem copiados do template, estado (pendente / concluída / saltada), timestamps. |
| Salto de tarefa | Uma `ProcessStepTaskInstance` pode ser saltada manualmente em runtime. Tarefa saltada permanece visível, marcada como "saltada", e conta como "resolvida" para fins de avanço automático da Step. |
| Auditoria de tarefa | Registra quem e quando saltou ou concluiu a tarefa. Justificativa não é obrigatória. |
| `ProcessStepFieldInstance` (campos de etapa) | Materialização runtime dos StepFields do template. Definição copiada do template (label, tipo, obrigatório), valor preenchido e timestamp de preenchimento. |
| `ProcessStepTaskFieldInstance` (campos de tarefa) | Materialização runtime dos TaskFields do template — campos configuráveis vinculados a uma Task específica. Mesma estrutura do `ProcessStepFieldInstance` (label, tipo, obrigatório, valor, timestamp), mas vinculado à `ProcessStepTaskInstance` em vez da `ProcessStepInstance`. |

---

## 9. Perguntas abertas

- **Premissa dos ~70% de passos compartilhados entre tipos.** A hipótese
  de que os três fluxos (abertura, alteração, baixa) compartilham a maior
  parte das etapas ainda precisa ser validada na operação real. O desenho
  do template e do mecanismo `appliesTo` assume essa premissa — caso a
  validação mostre que os fluxos divergem mais do que o esperado, a
  estrutura pode precisar de revisão.

- **Trajetória futura de personalização (fora do MVP).** Não é decisão do
  MVP, mas está no radar: tipo de processo virando entidade configurável;
  contabilidades criando e editando templates próprios; múltiplos templates
  por contabilidade. A estrutura de value object para tipo existe
  justamente para não travar essa evolução.