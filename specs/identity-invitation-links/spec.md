---
status: approved
domain: identity
created_at: 2026-04-17
approved_at: 2026-04-17
---

# Links de Convite

## 1. Visão da feature

Convidar pessoas para a Societiza deixa de ser um ato individual de digitar
um e-mail por vez. Em vez disso, quem tem permissão para convidar gera um
link. Esse link carrega em si o perfil de acesso que o convidado vai receber
e, quando aplicável, a contabilidade à qual ele vai pertencer. O admin
compartilha o link pelo meio que já usa — WhatsApp da equipe, e-mail em
grupo, mensagem direta — e a pessoa entra quando estiver pronta.

O admin decide na criação quantas pessoas podem entrar por aquele link. O
sistema cuida do resto: conta os usos, respeita o prazo de validade de 7
dias e bloqueia o link automaticamente assim que o limite é atingido, quando
vence, ou quando o admin decide revogar. Nada é apagado — todo link criado
permanece no histórico, junto com a lista de quem entrou por ele. Quando o
admin precisa saber "como essa pessoa chegou ao sistema?", a resposta está
ali.

A experiência é simples: dentro da área de equipe, um botão gera um link
pronto para copiar. Na área de convites, a visão default mostra apenas os
links **ativos** — é a pergunta operacional que o admin faz no dia a dia
("o que posso compartilhar agora?"). Os links **inativos** (esgotados,
expirados, revogados) permanecem acessíveis em uma visão histórica
separada, preservando a rastreabilidade sem poluir a tela de trabalho.
Nada é apagado — todo link criado permanece consultável, com sua contagem
de uso e a lista de quem entrou por ele. Dentro de uma mesma contabilidade,
todos os AccountancyAdmins enxergam e podem revogar os links criados por
qualquer outro admin da mesma contabilidade — o tenant é transparente para
quem tem autoridade administrativa dentro dele.

**Modelo de autoridade dentro da contabilidade.** Uma contabilidade pode
ter N AccountancyAdmins. Não há limite de 1. Um AccountancyAdmin pode
convidar outro AccountancyAdmin para a mesma contabilidade — é o mecanismo
pelo qual a contabilidade constrói sua própria camada administrativa sem
depender do SystemAdmin. Esse tipo de convite é um ato dirigido, não um
onboarding de equipe: o link nasce com limite fixo de 1 uso, igual ao
caso SystemAdmin → SystemAdmin. Admin é escolhido individualmente, não
provisionado em lote. A fronteira multi-tenant permanece inviolável: esse
convite só vale para a contabilidade do próprio admin, nunca para outra.

## 2. Problema que resolve

Hoje, cada pessoa que entra na Societiza exige que um admin digite o e-mail
dela e dispare um convite individual. Para um AccountancyAdmin que está
onboardando uma equipe de cinco funcionários, isso significa cinco ações
separadas, cinco erros possíveis de digitação, cinco convites pendentes
para acompanhar, e a obrigação de ter o e-mail correto de cada pessoa antes
mesmo de começar. Muitas vezes o admin nem sabe ainda qual e-mail o
funcionário vai usar no sistema.

Esse atrito acontece toda vez que a contabilidade cresce, toda vez que
alguém é substituído e toda vez que a Societiza onboarda uma contabilidade
nova. O workaround atual é engolir o atrito ou adiar o convite — o que
significa adiar a adoção do produto pela equipe. Para o SystemAdmin
convidando um AccountancyAdmin novo, é o mesmo problema em escala menor
mas com igual cerimônia.

O convite por e-mail individual também cria um modelo mental frágil: o
admin precisa acompanhar convites pendentes, reenviar quando expiram em 72
horas, lembrar quem ainda não aceitou. Tudo isso é trabalho administrativo
que não gera valor — é só fricção entre a intenção ("quero convidar minha
equipe") e o resultado ("minha equipe está dentro do sistema").

## 3. Para quem é

**Beneficiários diretos:**

- **SystemAdmin** — usa para convidar outros SystemAdmins (raro, mas
  crítico) e para onboardar novos AccountancyAdmins quando uma
  contabilidade entra na plataforma.
- **AccountancyAdmin** — usa para onboardar a equipe da sua contabilidade.
  É o caso de maior volume e o que mais se beneficia do link reutilizável.

**Beneficiários indiretos:**

- **AccountancyEmployee** — entra no sistema de forma mais fluida, no seu
  próprio tempo, sem depender de ter tido o e-mail digitado certinho pelo
  admin.
- **Toda a operação societária** — equipes inteiras passam a entrar em
  minutos, não em semanas. A Societiza deixa de ser gargalo na adoção.

O momento da jornada em que a feature entra: **onboarding de equipe**.
Seja no nascimento de uma nova contabilidade na plataforma, seja no
crescimento de uma equipe existente.

## 4. Como melhora a experiência atual

**Antes:**

O AccountancyAdmin precisa coletar os e-mails de cada pessoa da equipe
antes de começar. Abre a tela de convites, digita um e-mail, escolhe o
perfil, envia. Repete para a próxima pessoa. Um erro de digitação
significa um convite perdido e precisa ser revogado e reenviado. Cada
convite expira em 72 horas — se a pessoa demora para olhar, o admin precisa
reenviar. A caixa de entrada da pessoa pode ter filtrado como spam. O
admin precisa acompanhar quem aceitou, quem não aceitou, quem expirou.

**Depois:**

O AccountancyAdmin abre a área de equipe, clica em "gerar link de
convite", informa quantas pessoas vão entrar por aquele link (ex: 5) e
copia o link. Cola no grupo do WhatsApp da equipe. As cinco pessoas
entram no seu próprio ritmo, cada uma criando sua conta com o e-mail que
preferir. O admin vê a contagem de uso subir de 0/5 para 5/5 na área de
convites, e o link se desativa automaticamente. Se sobrar uma pessoa que
precisava entrar depois, o admin gera um link novo. Se uma vaga não for
usada, o link expira em 7 dias e some da lista de links ativos — sem ação
manual.

Para o caso SystemAdmin → SystemAdmin, a experiência é análoga mas mais
restrita: o link já nasce com limite fixo de uma única pessoa. É um
convite dirigido a alguém específico sem o peso operacional de um cadastro
por e-mail.

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**

- **Identity** — esta feature é parte do módulo de identidade. Reusa a
  noção de perfil de acesso (SystemAdmin, AccountancyAdmin,
  AccountancyEmployee) e de autenticação (criação de conta, login).
- **Accountancy** — o link que concede acesso a uma contabilidade carrega
  o `accountancyId`. Convites de AccountancyAdmin e AccountancyEmployee
  só fazem sentido no contexto de uma contabilidade existente.

**O que esta feature habilita no futuro:**

- **Métricas de onboarding** — com cada usuário carregando a referência do
  convite pelo qual entrou, fica possível medir quanto tempo uma
  contabilidade leva para colocar sua equipe na plataforma, quais links
  convertem e quais são abandonados.
- **Permissões mais granulares** — quando o produto evoluir para perfis
  intermediários dentro da contabilidade (gap conhecido de granularidade),
  a estrutura de link por perfil já estará pronta para acomodar novos
  papéis sem reformulação do fluxo.
- **Link dedicado por e-mail** — variação futura em que o admin pré-vincula
  o link a um e-mail específico. A infraestrutura atual de links com
  limite de uso e expiração já suporta essa evolução como refinamento,
  não como reformulação.
- **Onboarding em massa dentro da contabilidade** — AccountancyAdmins com
  equipes grandes podem gerar um único link AA → AE com `maxUses` alto,
  eliminando o gargalo de emitir convites um a um para cada funcionário.

**Posição no produto:**

Esta é uma feature **core do módulo Identity**. Ela substitui o fluxo
atual de convites individuais por e-mail. Não é uma camada adicional — é
a nova forma canônica de entrar na Societiza por convite. O auto-registro
continua sendo o ponto de entrada de usuários SystemAdmin seminais e de
contas que a Societiza semeia por fora; os links de convite cobrem tudo
que é *dirigido por papel* dentro da plataforma.

## 6. O que não é (escopo negativo)

- **Não inclui** disparo de e-mail para o convidado. O link é gerado e
  compartilhado pelo admin no canal que ele preferir. A Societiza não
  precisa conhecer o e-mail do convidado antes da aceitação.
- **Não inclui** link dedicado vinculado previamente a um e-mail
  específico. Todo link é aberto para qualquer pessoa dentro do seu
  limite de uso. Link com lock de e-mail fica para uma evolução futura.
- **Não inclui** QR code, geração de links em lote, ou interface de
  campanha de onboarding. A unidade de criação é um link por vez.
- **Não inclui** convite entre contabilidades diferentes. Um
  AccountancyAdmin nunca gera convite para outra contabilidade — a
  fronteira multi-tenant é inviolável. Só o SystemAdmin pode criar
  convites apontando para uma contabilidade específica que não é a dele
  (porque ele não tem contabilidade).
- **Não inclui** mudança de perfil após a entrada. O perfil é o que o
  link carrega, ponto. Promover um AccountancyEmployee a
  AccountancyAdmin, por exemplo, é fluxo separado de gestão de usuário —
  fora do escopo desta feature.
- **Não inclui** histórico visual de fluxo de onboarding (dashboard,
  gráficos, relatórios). A rastreabilidade desta versão é consulta — não
  visualização. Dashboards sobre onboarding ficam para o momento da
  feature de dashboard.
- **Não inclui** reativação de link expirado, revogado ou esgotado.
  Qualquer um desses estados é terminal. Se o admin precisa de mais
  acesso, gera um link novo.
- **Não inclui** exclusão de link. Nenhum link é apagado — nem pelo
  criador, nem pelo SystemAdmin. Todo link criado permanece no
  histórico com seu status final. O mecanismo para neutralizar um link
  é revogação, não exclusão. Excluir um link quebraria o vínculo de
  origem dos usuários que entraram por ele e apagaria a resposta para
  "como essa pessoa chegou ao sistema?".
- **Não inclui** integração com planos ou tier de cobrança. MVP é
  freemium — limites de quantas pessoas podem entrar em uma contabilidade
  pelo plano não existem como restrição desta feature.

## 7. Custos adicionais

**Nenhum custo externo identificado.**

O link é uma URL gerada e validada pelo próprio backend. Não depende de
provedor de e-mail (o e-mail atual de 72h deixa de ser disparado no novo
fluxo — o admin compartilha o link manualmente). Não depende de SMS,
WhatsApp, ou qualquer gateway externo. Não exige armazenamento adicional
relevante — é uma tabela de registros pequenos com crescimento linear e
controlado.

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Substituir fluxo atual | Sim. O fluxo de convite individual por e-mail sai. Link de convite é a forma canônica de entrada por convite na Societiza. |
| Expiração | 7 dias para todos os links, independente do perfil ou caso de uso. |
| Disparo de e-mail ao convidado | Não. O admin compartilha o link no canal que preferir. A Societiza não precisa do e-mail do convidado antes da aceitação. |
| Limite de uso — SystemAdmin → SystemAdmin | Fixo em 1. O próprio caso de uso pede entrada única e controlada. |
| Limite de uso — AccountancyAdmin → AccountancyAdmin | Fixo em 1. Admin é cargo escolhido individualmente, não onboarding de equipe. Mesma lógica e simetria do caso SA → SA. |
| Limite de uso — AccountancyAdmin → AccountancyEmployee | Definido pelo admin no momento da criação. |
| Limite de uso — SystemAdmin → AccountancyAdmin | Fixo em 1. Convite dirigido a uma pessoa específica, não onboarding de equipe. Simétrico a SA → SA e AA → AA. |
| Separação visual entre links ativos e inativos | Sim. Visão default da área de convites mostra apenas links ativos. Links inativos (esgotados, expirados, revogados) vão para uma visão histórica separada. Preserva rastreabilidade sem poluir a tela de trabalho. Decisão de produto, execução de interface. |
| Revogação | Quem criou o convite pode revogar. AccountancyAdmins podem revogar qualquer link ativo da sua própria contabilidade — inclusive os criados por outros AccountancyAdmins do mesmo tenant. SystemAdmin pode revogar apenas links SA→SA. |
| Comportamento pós-limite/expiração/revogação | Estados terminais. Link não pode ser reativado. O admin gera um novo se necessário. |
| Preservação de histórico | Nenhum convite é excluído fisicamente. Links esgotados, expirados e revogados permanecem visíveis na área de convites com status claro. |
| Rastreabilidade | Todo usuário que entra por um link carrega referência ao link pelo qual entrou. A lista de quem entrou por um link é consulta direta. Usuários semeados fora do fluxo de convite (primeiro SystemAdmin) não têm essa referência — é aceitável. |
| Fonte de verdade | Backend. Toda validação de expiração, revogação, limite de uso e permissão para criar é feita no servidor. A interface apenas reflete o estado. |
| Quantos AccountancyAdmins por contabilidade | N. Não há limite de 1. Contabilidades do ICP frequentemente têm mais de uma pessoa com autoridade administrativa real. Restrição a 1 cria bus factor. |
| AccountancyAdmin convidar outro AccountancyAdmin | Sim, dentro da própria contabilidade. A fronteira multi-tenant impede qualquer admin convidar para uma contabilidade diferente da sua. |
| Escopo multi-tenant | Inviolável. AccountancyAdmin só cria convites dentro da sua própria contabilidade. SystemAdmin é o único papel que pode apontar um convite para uma contabilidade específica que não seja a dele. |
| Exclusão de link | Não existe. Nenhum link é apagado, por nenhum papel. Revogação é o mecanismo de neutralização. Histórico é intocável — preserva rastreabilidade e a resposta a "como esse usuário entrou?". |
| Aviso visual de expiração iminente | Sim, desejado. O backend entrega `createdAt` e `expiresAt`; o frontend calcula e exibe o sinal. Decisão de produto, execução de interface. |
| Ponto de criação do link SA→AA | SystemAdmin cria o link SA→AA de dentro do contexto da contabilidade de destino (entra na contabilidade via módulo Accountancy e gera o link a partir da área de convites daquele tenant). A tela geral de convites do SystemAdmin contém apenas links SA→SA. O link SA→AA, uma vez criado, habita a contabilidade de destino — não a tela pessoal do SystemAdmin. |
| Atalho "revogar e regerar" | Não. Complexidade desnecessária para o MVP. O admin revoga e cria um link novo — dois passos simples. |
| Visibilidade cross-admin dentro da mesma contabilidade | Sim. Todos os AccountancyAdmins de uma mesma contabilidade enxergam e podem revogar os links criados por qualquer outro admin da mesma contabilidade. Transparência interna é o default dentro do tenant. |
| Convidado que já tem conta na Societiza | Não entra na organização pelo link. Mensagem explicativa no frontend; nenhuma absorção automática de conta. Sem trabalho adicional em backend ou frontend além do bloqueio. |

## 9. Perguntas abertas

Nenhuma. Todas as perguntas que estavam em aberto foram respondidas e
estão refletidas nas seções de **Decisões tomadas** e **Escopo negativo**.
Resumo do que foi fechado nesta iteração:

- Aviso visual de expiração iminente → sim, responsabilidade do frontend
  a partir dos campos `createdAt` e `expiresAt`.
- Notificação ao criador quando SystemAdmin revoga → não.
- Atalho "revogar e regerar" → não, fora do escopo MVP.
- Visibilidade cross-admin dentro da mesma contabilidade → sim, links são
  compartilhados entre todos os AccountancyAdmins do mesmo tenant.
- Convidado que já tem conta → bloqueado com mensagem explicativa, sem
  absorção de conta, sem trabalho adicional de integração.
- Exclusão de link → não existe. Revogação é o mecanismo de neutralização.
- Múltiplos AccountancyAdmins por contabilidade e admin convidando admin
  → sim para ambos, dentro da mesma contabilidade.
- Poluição da tela de convites por links inativos (soft-delete em todo o
  sistema) → resolvida por separação visual: ativos por default, histórico
  em visão separada. Dado preservado, UX focada no operacional.
- Limite de uso em AccountancyAdmin → AccountancyAdmin → fixo em 1. Admin
  é ato dirigido, não provisionamento de equipe. Simétrico ao caso SA → SA.