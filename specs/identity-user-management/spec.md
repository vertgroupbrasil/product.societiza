---
status: draft
domain: identity
created_at: 2026-04-17
---

# Gestão de Usuários

## 1. Visão da feature

Depois que uma pessoa entra na Societiza pelo link de convite, ela não fica
em um limbo. Ela tem um perfil que pode ajustar, aparece em uma lista que a
equipe enxerga, pode ser promovida quando assume mais responsabilidade e
pode ser removida quando sai do time. A gestão de usuários é a camada que
cuida da vida do membro dentro da contabilidade — do primeiro dia ao
desligamento.

A experiência é direta. Qualquer usuário encontra seu próprio perfil no
menu da conta e consegue corrigir o nome quando quiser, sem cerimônia.
Dentro da contabilidade, existe uma área de **Equipe** onde os membros são
listados com nome e perfil. Um AccountancyAdmin vê a mesma lista, mas com
controles: promover um funcionário a administrador, remover quem não faz
mais parte do time. O SystemAdmin, do lado da plataforma, enxerga a lista
de outros SystemAdmins em sua área administrativa e, ao entrar no contexto
de uma contabilidade específica, enxerga os usuários daquela contabilidade
com autoridade para remover quando for operacionalmente necessário.

Ninguém edita o nome de outra pessoa. O dado pessoal é do usuário. O que
admins controlam é papel e presença — quem está dentro, com qual perfil.
O dado de cada pessoa segue com a própria pessoa.

## 2. Problema que resolve

Hoje, uma vez que o usuário entrou na plataforma pelo link de convite, não
existe nenhum fluxo formal de gestão da sua vida dentro do sistema. O
ciclo de convite termina na criação da conta e não há continuidade. Isso
cria três dores operacionais concretas e uma dor de autogestão menor.

**Promoção paralisada.** A contabilidade cresce. Um AccountancyEmployee
que começou operacional assume responsabilidades de administrador — passa
a onboardar a equipe, gerar links de convite, gerenciar processos em nome
da contabilidade. Hoje, para ele virar AccountancyAdmin, seria necessário
convidá-lo de novo, o que não faz sentido: ele já tem conta, já tem
histórico, já está dentro. Sem caminho de promoção, a autoridade
administrativa fica travada em quem foi o primeiro a entrar como admin.
Isso força contabilidades a duplicar contas ou a aceitar que autoridade
interna depende da ordem em que cada um foi convidado — não da função que
cada um exerce de fato.

**Saída sem caminho.** Pessoas saem da contabilidade. São desligadas,
pedem demissão, migram para outra função. O link de convite tem revogação,
mas usuários que já entraram não têm nenhum mecanismo de saída. Isso é um
problema de segurança (ex-funcionário mantém acesso à operação societária
da empresa), um problema operacional (lista da equipe fica poluída com
gente que não existe mais) e um problema de confiança (admin não consegue
responder "quem tem acesso aos dados da contabilidade hoje?"). O
workaround atual é não ter workaround — o acesso fica até alguém intervir
tecnicamente por fora.

**Operação sem lista.** O AccountancyAdmin precisa saber quem está dentro
da equipe. O SystemAdmin precisa saber quem administra cada contabilidade
e quem opera lá dentro. Sem listagem, não há como responder perguntas
básicas do cotidiano — "quantas pessoas temos nessa contabilidade?",
"quem é o outro admin que pode cobrir minha ausência?", "esse e-mail que
aparece no histórico do processo é de quem?". Hoje essas perguntas ou
ficam sem resposta ou exigem que alguém olhe no banco de dados.

**Autogestão travada.** Um usuário digitou o nome errado no momento da
criação da conta, ou mudou de nome, ou simplesmente quer ajustar a
apresentação. Hoje não tem caminho. Ele precisa pedir a alguém da Societiza
para corrigir por fora. É atrito desnecessário em um dado que é dele.

Esses quatro problemas acontecem continuamente conforme contabilidades
crescem, trocam membros e amadurecem dentro da plataforma. Não são casos
raros — são o fluxo normal de qualquer equipe em operação.

## 3. Para quem é

**Beneficiários diretos:**

- **SystemAdmin** — usa a listagem para ter visibilidade de quem
  administra a plataforma. Ao entrar no contexto de uma contabilidade,
  enxerga os usuários dela. Usa exclusão como ferramenta operacional de
  última instância — remover acesso indevido, desfazer conta duplicada,
  apoiar uma contabilidade que perdeu controle da própria equipe.
- **AccountancyAdmin** — é o maior beneficiário. Gerencia a equipe da
  sua contabilidade no dia a dia: lista quem está dentro, promove quem
  assume mais responsabilidade, remove quem sai. É o papel que mais exerce
  a feature.
- **Qualquer usuário** — edita o próprio nome quando precisar. Vê o
  próprio perfil. Tem a autogestão mínima que qualquer produto moderno
  oferece.

**Beneficiários indiretos:**

- **AccountancyEmployee** — ganha visibilidade da equipe com quem
  colabora sem precisar ter autoridade administrativa. Saber quem são os
  colegas é parte de operar em time.
- **A própria contabilidade** — passa a ter controle real sobre sua lista
  de acessos, reduzindo risco de segurança e ruído operacional.
- **Futuras features** — atribuição de tarefa, menção em comentário,
  rastreio de autoria em processos, dashboard por responsável. Todas
  pressupõem um usuário estável, identificável e com presença gerenciada.

O momento da jornada em que a feature entra: **ciclo de vida do usuário
depois da entrada**. Começa no primeiro login (autogestão de perfil),
passa pela operação contínua (listagem, promoção) e termina no
desligamento (exclusão).

## 4. Como melhora a experiência atual

**Antes:**

O AccountancyAdmin consegue convidar gente (pelos links de convite) mas
não consegue gerenciar quem já entrou. Se o funcionário que ele convidou
virar candidato a administrador, o caminho é convidar de novo pelo link
AA → AA — gerando uma segunda conta, com histórico perdido e e-mail
possivelmente diferente. Se o funcionário saiu da contabilidade, não há
nada a fazer: o acesso continua. Se o admin quer ver quem está na equipe,
não há lista. Se um usuário qualquer digitou o próprio nome errado na
criação da conta, ele vive com o erro até pedir ajuda por fora da
plataforma.

**Depois:**

O AccountancyAdmin abre a área de Equipe e vê todos os membros da
contabilidade — nome, perfil, quando entraram. No AccountancyEmployee que
virou candidato a admin, clica em "Promover a administrador" e confirma.
O perfil muda imediatamente. No funcionário que saiu, clica em "Remover
acesso" e confirma. A sessão daquela pessoa cai na hora, ela não aparece
mais na lista ativa, e seu histórico de ações passadas permanece íntegro
(quem criou o processo X continua sendo ela). Qualquer usuário, incluindo
o próprio AccountancyAdmin, abre o menu de perfil, edita o próprio nome e
salva. Pronto.

Para o SystemAdmin, o fluxo é análogo. No painel administrativo, vê a
lista de SystemAdmins. Ao entrar no contexto de uma contabilidade, vê os
AccountancyAdmins e AccountancyEmployees daquela contabilidade, com a
mesma capacidade de remover quando for operacionalmente necessário.

Para o AccountancyEmployee, a mudança é mais discreta mas importante: ele
pode corrigir o próprio nome quando quiser, e vê os colegas da equipe
listados — algo que parece trivial mas é a base para toda colaboração
futura dentro do produto.

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**

- **Identity — Autenticação** — esta feature opera sobre a conta do
  usuário autenticado. A edição do próprio perfil exige sessão válida. A
  remoção de acesso tem como consequência invalidar a sessão do usuário
  removido. Toda validação de permissão passa pela identidade do
  solicitante.
- **Identity — Links de convite** — esta feature começa onde os links
  terminam. Ela não reabre nenhuma decisão do fluxo de convites. Mas usa
  o mesmo modelo de perfis (SystemAdmin, AccountancyAdmin,
  AccountancyEmployee) e a mesma postura de preservação de histórico
  (nada é apagado fisicamente).
- **Accountancy** — a listagem, a promoção e a exclusão feitas por
  AccountancyAdmin estão limitadas à sua própria contabilidade. A
  fronteira multi-tenant é o mesmo contorno já vigente em todas as demais
  features. SystemAdmin entra no contexto de uma contabilidade específica
  para agir sobre seus usuários — não age em lote cross-tenant.

**O que esta feature habilita no futuro:**

- **Atribuição de tarefa** — a partir do momento em que a lista de
  membros da contabilidade é um dado estável e consultável, tarefas podem
  ser atribuídas a pessoas específicas. Hoje não há onde atribuir.
- **Autoria rastreável em processos** — criação de processo, execução de
  etapa, conclusão de tarefa passam a ter um "por quem" que permanece
  íntegro mesmo quando o usuário sai da contabilidade. A presença some,
  o histórico fica.
- **Dashboard por responsável** — análise operacional filtrada por
  pessoa (quantos processos cada admin conduz, quais funcionários têm
  mais tarefas em andamento) depende de uma lista de membros estável.
- **Menção em comentário e colaboração interna** — qualquer feature
  futura que dependa de referenciar outro membro da contabilidade
  pressupõe que a lista de membros seja consultável por todos.
- **Permissões granulares dentro da contabilidade** — quando o produto
  evoluir para perfis intermediários (gap conhecido), o caminho de
  promoção e edição de papel já estará estruturado. Adicionar um novo
  perfil vira uma extensão do que já existe, não uma reformulação.
- **Portal do cliente** — futura colaboração externa com o empresário
  pressupõe que a contabilidade tenha sua própria lista de membros
  organizada do lado interno antes de expor qualquer coisa ao cliente
  final.

**Posição no produto:**

Feature **core do módulo Identity** e do núcleo operacional atual (P1).
Completa o ciclo de vida do usuário dentro da plataforma — os links de
convite cobrem a entrada, a gestão de usuários cobre a vida depois da
entrada. Sem ela, o módulo Identity permanece incompleto e várias
features futuras ficam bloqueadas por falta de um modelo de presença
gerenciável.

## 6. O que não é (escopo negativo)

- **Não inclui** edição do próprio e-mail. O e-mail não é editável no
  MVP. Sem fluxo de confirmação, sem revalidação por token, sem segundo
  e-mail de backup. É uma decisão explícita de escopo, não uma lacuna
  temporária. Se o usuário precisa trocar de e-mail, a solução atual é
  fora da plataforma (pedir ao admin para remover e reconvidar). O
  tratamento correto desse caso pede maturidade de produto que o MVP não
  tem — e a frequência real desse cenário não justifica o investimento
  agora.
- **Não inclui** edição de dados pessoais de outro usuário. Ninguém
  edita o nome de outra pessoa, em nenhum cenário. Dado pessoal é do
  usuário. SystemAdmin não edita nome de AccountancyAdmin. AccountancyAdmin
  não edita nome de AccountancyEmployee. Nenhuma exceção. Se o usuário
  está com o nome errado, ele mesmo corrige. Se ele não consegue acessar
  para corrigir, a alternativa é remover e reconvidar — fluxo que já
  existe.
- **Não inclui** demotion (rebaixar AccountancyAdmin para
  AccountancyEmployee). No MVP, promoção é unidirecional. Se a
  contabilidade promoveu alguém por engano ou precisa reverter, a
  solução é remover o acesso e reconvidar no perfil correto. A
  frequência esperada desse caso é baixa, e a complexidade de fluxo
  (impacto em convites já criados por essa pessoa, transferência de
  autoridade, confirmação dupla) não justifica o investimento agora.
- **Não inclui** autoexclusão. O usuário não se remove. Quem remove é o
  admin responsável. Coerente com a postura de que remoção é
  consequência de uma decisão da equipe ou da plataforma, não um ato
  individual. Autoexclusão traz complexidades de ciclo (o que acontece
  quando o último AccountancyAdmin tenta se autoexcluir?) que são
  desnecessárias no MVP.
- **Não inclui** exclusão do último AccountancyAdmin de uma
  contabilidade. Bloqueio automático. Uma contabilidade sem admin fica
  órfã — sem caminho de gestão, sem possibilidade de promover outro
  membro, sem capacidade de convidar. O bloqueio é uma proteção
  estrutural. A mensagem ao admin que tenta a ação é clara: promova
  outro membro a administrador antes de sair.
- **Não inclui** exclusão do último SystemAdmin do sistema. Bloqueio
  automático. Sistema sem SystemAdmin fica inoperável no nível de
  plataforma.
- **Não inclui** SystemAdmin promovendo AccountancyEmployee para
  AccountancyAdmin em uma contabilidade que não é dele. A promoção é
  ato interno da contabilidade, executado por quem já tem autoridade
  administrativa dentro dela. SystemAdmin pode entrar numa contabilidade,
  ver os membros e até excluir (poder operacional de última instância),
  mas não promove — a autoridade administrativa da contabilidade nasce
  de dentro dela, não de fora. Essa separação preserva o modelo onde o
  SystemAdmin observa e intervém em emergências, mas não governa a
  estrutura interna do tenant.
- **Não inclui** exclusão física de usuário (hard delete). Todo usuário
  removido é marcado como inativo; sua identidade persiste para
  preservar autoria em processos, etapas, tarefas, comentários e
  convites que ele criou no passado. Coerente com a postura do fluxo de
  convites ("histórico intocável") e com o valor de rastreabilidade do
  produto. Excluir fisicamente quebraria a resposta para perguntas como
  "quem criou esse processo?" ou "quem gerou esse link de convite?".
- **Não inclui** notificação por e-mail de eventos de gestão (promoção,
  remoção). No MVP, ações são imediatas e visíveis na interface. O
  usuário promovido descobre ao próximo login. O usuário removido vê a
  sessão cair. Envios de e-mail entram quando o módulo de notificações
  existir — e essa decisão é do módulo de notificações, não desta
  feature.
- **Não inclui** histórico/auditoria visível ao usuário na interface.
  O sistema registra internamente quem promoveu quem, quem removeu quem
  e quando (necessário para rastreabilidade e suporte), mas o MVP não
  expõe essa trilha em uma tela dedicada. Visualização de auditoria
  entra em uma evolução posterior.
- **Não inclui** gestão granular de permissões dentro da contabilidade.
  Continua valendo o modelo de três perfis (SystemAdmin,
  AccountancyAdmin, AccountancyEmployee). Perfis intermediários, escopo
  por processo, permissão por tipo de tarefa — tudo fora do MVP. Gap
  conhecido, documentado, e intencionalmente não endereçado agora.
- **Não inclui** reativação de usuário removido. Remoção é terminal.
  Se a pessoa precisa voltar, o admin gera um novo link de convite e
  ela entra como um novo usuário (com novo registro, mantendo o histórico
  antigo como histórico de alguém que saiu). Coerente com a postura dos
  links de convite (revogação é terminal) e simplifica o fluxo — sem
  modal de confirmação "tem certeza que quer reativar este usuário?",
  sem preocupação com colisão de e-mail entre registro antigo e novo.
- **Não inclui** troca de contabilidade (transferir usuário de uma
  contabilidade para outra). Um usuário pertence a uma única
  contabilidade. Mudar isso exige sair de uma contabilidade e entrar em
  outra por convite novo. Fluxo de transferência interna não existe no
  MVP.

## 7. Custos adicionais

**Nenhum custo externo identificado.**

A feature opera inteiramente sobre dados de usuário já existentes na
plataforma. Não depende de provedor de e-mail (no MVP não há disparo de
notificação de gestão), não depende de SMS, WhatsApp, captcha, gateway
externo ou API de terceiros. O armazenamento adicional é irrelevante — o
que cresce é o número de campos atualizáveis por registro e o estado de
atividade do usuário, ambos absorvidos pela estrutura atual.

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Posição no produto | Feature core do módulo Identity, complementar ao fluxo de links de convite. Começa onde os convites terminam. |
| Edição do próprio perfil | Permitida. O único campo editável no MVP é `Name`. |
| Edição de e-mail próprio | Não. Fora do escopo do MVP. Sem confirmação, sem revalidação, sem token. |
| Edição de dados pessoais de outro usuário | Proibida para todos os perfis, sem exceção. Nome é dado do usuário — só ele altera. |
| Promoção AccountancyEmployee → AccountancyAdmin | Permitida ao AccountancyAdmin, restrita à própria contabilidade. |
| Promoção por SystemAdmin | Não. Autoridade administrativa interna da contabilidade nasce de dentro dela. SystemAdmin observa e intervém em emergências via exclusão, mas não promove. |
| Demotion (AA → AE) | Não no MVP. Promoção é unidirecional. Para reverter, remover e reconvidar. |
| Autoexclusão | Não no MVP. O admin responsável remove. |
| Exclusão por SystemAdmin | Pode excluir qualquer perfil — SystemAdmin, AccountancyAdmin, AccountancyEmployee. |
| Exclusão por AccountancyAdmin | Pode excluir AccountancyAdmin e AccountancyEmployee, restrita à própria contabilidade. |
| Exclusão do último AccountancyAdmin de uma contabilidade | Bloqueada. Mensagem orienta a promover outro membro antes. |
| Exclusão do último SystemAdmin do sistema | Bloqueada. Sistema sem SystemAdmin fica inoperável. |
| Exclusão de si mesmo | Proibida. Um admin não se remove — outro admin remove. Se for o último, o bloqueio anterior já cobre. |
| Soft delete vs hard delete | Soft delete. Usuário removido fica marcado como inativo; identidade persiste para preservar autoria em processos, etapas, tarefas, comentários e convites passados. |
| Reativação de usuário removido | Não existe. Remoção é terminal. Para voltar, novo convite gera novo usuário. |
| Comportamento da sessão ativa do usuário removido | Invalidada imediatamente. O usuário removido perde acesso no ato, sem esperar próximo login. |
| Histórico de autoria em processos/etapas/tarefas/comentários/convites criados por usuário removido | Preservado. A referência ao criador permanece e a interface mostra o nome preservado mesmo após a remoção. Coerente com "histórico intocável" do fluxo de convites. |
| Convites ativos criados por usuário removido | Permanecem ativos até expirar, esgotar ou serem revogados por outro admin da contabilidade. A revogação automática em cascata não é parte do MVP — outros admins do mesmo tenant já enxergam e podem revogar, conforme fluxo de convites. |
| Notificação por e-mail de promoção ou exclusão | Não no MVP. Descoberta ao próximo login (promoção) ou na perda de sessão (exclusão). Notificações entram com o módulo de notificações. |
| Trilha de auditoria visível ao usuário | Não no MVP. Sistema registra internamente para suporte e rastreabilidade, mas não há tela dedicada. |
| Listagem vista pelo SystemAdmin | Painel administrativo lista todos os SystemAdmins. Ao entrar no contexto de uma contabilidade, lista AccountancyAdmins e AccountancyEmployees daquela contabilidade. |
| Listagem vista pelo AccountancyAdmin | Lista todos os AccountancyAdmins e AccountancyEmployees da própria contabilidade. |
| Listagem vista pelo AccountancyEmployee | Permitida, em visão somente leitura, restrita à própria contabilidade. A contabilidade é um time; saber com quem se trabalha é operação, não privilégio administrativo. Habilita features futuras de colaboração sem reformulação. |
| Dados visíveis na listagem — AccountancyAdmin e SystemAdmin | Nome, e-mail, perfil, data de entrada, referência ao link de convite pelo qual entrou (quando houver). Dados completos para quem gerencia. |
| Dados visíveis na listagem — AccountancyEmployee | Nome e perfil. Sem e-mail, sem data de entrada, sem referência de convite. Visão minimalista, suficiente para identificar colegas, sem expor dados sensíveis aos não-gestores. |
| Separação entre usuários ativos e inativos na listagem | Sim. Visão default mostra apenas usuários ativos — pergunta operacional do dia a dia ("quem está na equipe agora?"). Usuários removidos (soft-deleted) ficam em visão separada, acessível a quem gerencia, preservando rastreabilidade sem poluir a tela de trabalho. Coerente com o padrão adotado no fluxo de links de convite. |
| Fronteira multi-tenant | Inviolável. AccountancyAdmin age apenas dentro da própria contabilidade. SystemAdmin entra no contexto de uma contabilidade para agir sobre seus usuários — não age em lote cross-tenant. |
| Fonte de verdade de permissões | Backend. Toda validação de quem pode editar, promover, excluir e listar é feita no servidor. A interface apenas reflete o estado permitido. UI nunca é a única barreira. |
| Confirmação obrigatória | Promoção e exclusão exigem confirmação explícita antes de executar. Edição do próprio nome não exige — é reversível pelo próprio usuário a qualquer momento. |
| Permissões efetivas após promoção | Aplicadas imediatamente. O usuário promovido ganha acesso a ações de AccountancyAdmin no próximo carregamento de tela ou no próximo login, o que vier antes. |

## 9. Perguntas abertas

Nenhuma. Todas as decisões de produto estão fechadas. Os pontos que
haviam sido sinalizados no input (SystemAdmin promover em tenant alheio,
demotion no MVP, autoexclusão, tratamento do último admin, soft vs hard
delete, listagem para AccountancyEmployee, dados visíveis na listagem,
sessões ativas após exclusão, autoria em histórico) estão resolvidos na
seção 8.