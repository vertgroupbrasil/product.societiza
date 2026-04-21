---
status: approved
domain: identity
created_at: 2026-04-20
---

# Autenticação

## 1. Visão da feature

Autenticação é o portão de entrada da Societiza. Depois que uma pessoa
cria sua conta — seja por auto-registro, seja por um link de convite —
ela precisa de uma forma confiável de entrar todos os dias, permanecer
dentro do sistema enquanto trabalha e recuperar o acesso se esquecer a
senha. Esta spec cobre essa camada inteira: login, logout, renovação
silenciosa da sessão, e o fluxo de recuperação de senha (solicitação
por e-mail e redefinição via link).

Do ponto de vista do usuário, a experiência é quase invisível quando
funciona. Ele entra uma vez com e-mail e senha, e a partir dali o
sistema mantém a sessão viva em segundo plano — ele não é deslogado no
meio de um processo, não precisa re-autenticar entre abas, não tem que
lembrar "quando foi a última vez que logou?". Quando quer sair, clica em
sair e a sessão é encerrada imediatamente, em todos os lugares. Quando
esquece a senha, pede um link, recebe no e-mail, cria uma senha nova e
volta a operar. O objetivo é que o contador nunca sinta que a plataforma
está no caminho dele — a sessão é um utilitário silencioso, não um
obstáculo.

## 2. Problema que resolve

Sem um fluxo de autenticação que cubra os cinco casos juntos, a
plataforma é funcionalmente inacessível depois do cadastro. A lista de
dores concretas que esta spec elimina:

- **Sem login, não há acesso.** Uma conta criada por auto-registro ou
  por link de convite só tem valor se a pessoa puder entrar todos os
  dias. É o caminho de volta ao produto.
- **Sessões curtas sem renovação silenciosa quebram o trabalho.** Um
  contador que fica 30 minutos trabalhando em um processo e é deslogado
  no meio perde contexto, recomeça ações, e desconfia da confiabilidade
  da plataforma. Sem renovação automática, o token curto de acesso vira
  uma fonte constante de fricção.
- **Sem logout explícito, não há controle sobre a sessão.** Um contador
  que usa uma máquina compartilhada, ou que termina o expediente e quer
  encerrar a presença no sistema, precisa de um botão que diga "saí".
  Depender só da expiração de token não é uma resposta para quem quer
  ação deliberada.
- **Sem recuperação de senha, esquecer é perder acesso.** Contadores do
  ICP operam com dezenas de sistemas — portais de prefeitura,
  e-Social, ERPs, Receita Federal. Esquecer uma senha é rotina, não
  exceção. Sem um caminho claro de recuperação por e-mail, a única
  saída é contato humano com a Societiza — e isso não escala.

O workaround atual, na ausência desta camada, seria forçar o usuário a
logar em cada aba nova, aceitar sessões que morrem a cada 15 minutos,
ou pedir suporte manual para cada senha esquecida. Nenhum desses é
aceitável para um produto B2B que quer ser o sistema operacional do
contador.

## 3. Para quem é

**Beneficiários diretos:**

- **Todo usuário autenticado da Societiza** — SystemAdmin,
  AccountancyAdmin e AccountancyEmployee. Os cinco fluxos (login,
  logout, renovação silenciosa, forgot e reset) são idênticos em
  comportamento para os três perfis. O que muda entre perfis é o que
  eles fazem *depois* de autenticados, não como autenticam.

**Beneficiários indiretos:**

- **Toda feature protegida da plataforma** — societário, accountancy,
  convites, dashboard futuro. Nenhuma dessas tem como existir sem o
  perímetro de autenticação resolvido. Esta spec é o chão sobre o qual
  o resto do produto pisa.
- **A equipe da contabilidade como unidade** — quando a autenticação é
  confiável e silenciosa, a equipe opera sem se preocupar com acesso. A
  plataforma desaparece como obstáculo e vira ferramenta.

**Momento da jornada em que a feature entra:** **todo acesso à
plataforma depois do cadastro inicial**. É o fluxo mais executado do
produto — várias vezes por dia, por cada usuário ativo.

## 4. Como melhora a experiência atual

**Antes (sem esta camada):**

Não existe estado "antes" aplicável dentro da Societiza — esta é a
camada fundacional. Mas o contraste com o que o contador enfrenta hoje
em sistemas fragmentados é direto:

- Em muitos portais que o contador usa no dia a dia, a sessão morre sem
  aviso no meio do trabalho, e a tela volta para o login sem preservar
  o que estava sendo feito.
- Em vários sistemas, o logout é ambíguo: a aba fechou, mas a sessão
  ficou viva em outra máquina.
- Recuperação de senha costuma ser um labirinto — formulários que não
  confirmam envio, links que não chegam, redefinições que não aplicam.

**Depois (com esta camada):**

- **Login direto.** Entra com e-mail e senha. Se as credenciais estão
  corretas, está dentro. Se estão erradas, recebe uma mensagem clara.
- **Sessão que se mantém.** Enquanto o contador está usando a
  plataforma — ou ela está aberta em uma aba — a sessão se renova
  automaticamente. Ele não percebe nada. Não é interrompido.
- **Logout imediato.** Clica em sair e a sessão acaba ali. Se ele
  estava logado em outra aba, aquela aba também perde acesso na próxima
  requisição.
- **Recuperação de senha previsível.** Clica em "esqueci minha senha",
  informa o e-mail, recebe confirmação de que o pedido foi processado
  (sem revelar se o e-mail existe), abre o e-mail, clica no link,
  define uma senha nova, e está pronto para logar.

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**

- **Identity como perímetro** — esta feature é o coração do módulo
  Identity. Ela materializa o que `identity-context.md` descreve em
  alto nível: login com e-mail e senha, recuperação por e-mail com
  link de 1 hora, logout imediato, confirmação genérica para proteger
  privacidade.
- **Provedor de e-mail transacional** — forgot password depende de
  envio de e-mail confiável para entregar o link de redefinição. O
  mesmo canal é usado (ou será usado) pelas features de convite, e no
  futuro pelas notificações operacionais.
- **Accountancy** — toda sessão autenticada carrega o contexto de
  `accountancyId` do usuário (quando aplicável — SystemAdmin não tem
  contabilidade). Isso é o que permite que as features downstream
  filtrem dados por tenant.

**O que esta feature habilita no futuro:**

- **Toda feature protegida da plataforma.** Societário, convites,
  dashboard, notificações — nenhuma delas existe sem este perímetro.
- **Métodos de autenticação adicionais.** O documento de identity
  projeta autenticação via Google como método adicional. A base de
  sessão, renovação silenciosa e logout estabelecida aqui é compatível
  com qualquer método de entrada que venha depois.
- **Sessão compartilhada com clientes finais.** Quando o portal do
  cliente chegar, os mesmos mecanismos de sessão e recuperação de
  senha se aplicam — o contrato de autenticação é o mesmo para todos
  os perfis.
- **Políticas de segurança mais fortes.** MFA, SSO e OAuth futuros se
  plugam sobre o fluxo base. Esta spec não os inclui, mas também não
  fecha porta para eles.

**Posição no produto:**

Esta é uma feature **fundacional do módulo Identity**. Não é
complementar, não é derivada. É pré-requisito de tudo que já foi
entregue e de tudo que virá. A qualidade dela define a percepção de
confiabilidade da plataforma inteira.

## 6. O que não é (escopo negativo)

- **Não inclui** registro de conta. Auto-registro público está fora
  desta spec e será endereçado em spec separada. Registro via link de
  convite já está coberto pela spec `identity-invitation-links`
  aprovada.
- **Não inclui** MFA (segundo fator de autenticação). É horizonte
  futuro e não compromete o MVP.
- **Não inclui** SSO corporativo ou login via provedores externos
  (Google, Microsoft). A direção futura registrada em
  `identity-context.md` é permitir Google como método adicional,
  nunca substituto de e-mail e senha — mas isso é spec separada.
- **Não inclui** gestão de sessões ativas pelo usuário (listar
  dispositivos logados, encerrar outra sessão remotamente). O logout
  encerra a sessão atual; a expiração cuida do resto.
- **Não inclui** política de senha forte configurável por
  contabilidade. A regra de força mínima de senha é a mesma para
  todos os usuários da plataforma.
- **Não inclui** alteração de senha a partir do perfil (usuário
  logado trocando de senha voluntariamente). Isso é parte de gestão
  de perfil de usuário, e `identity-context.md` registra que
  "atualização de perfil do usuário" é um gap conhecido — entra em
  spec de gestão de usuário.
- **Não inclui** bloqueio de conta por excesso de tentativas de
  login. Mitigações de força bruta podem existir no nível de
  infraestrutura, mas não são expostas como feature de produto nesta
  versão.
- **Não inclui** captcha ou desafio anti-bot no fluxo de login ou
  forgot password. Se for necessário por volume de abuso, entra como
  refinamento futuro.
- **Não inclui** reset de senha por outro canal que não e-mail. SMS,
  WhatsApp e chamada telefônica estão fora.
- **Não inclui** reuso de link de reset de senha. Uma vez que a senha
  é redefinida por aquele link, o link não pode ser reaplicado.
- **Não inclui** um fluxo de "lembrar de mim" opcional distinto da
  renovação silenciosa padrão. A renovação automática é o
  comportamento default.

## 7. Custos adicionais

**Provedor de e-mail transacional.**

A feature de forgot password depende de envio confiável de e-mail com
link de redefinição. Esse mesmo canal é compartilhado com a feature de
convites (que também envia e-mails transacionais) e futuramente com
notificações operacionais. Não é um custo exclusivo desta spec — é uma
dependência de infraestrutura que a Societiza já assume por necessidade
do módulo Identity como um todo.

Implicações:

- Custo recorrente proporcional ao volume de e-mails transacionais
  (logins esquecidos, convites, notificações futuras).
- Provedor precisa ter taxa de entrega confiável — e-mail de
  redefinição que cai em spam ou não chega vira chamado de suporte
  humano e mina a confiança na plataforma.
- Monitoramento de falha de entrega é responsabilidade de operação,
  não algo que o usuário final deva lidar.

Fora isso, **nenhum custo externo adicional identificado**. Login,
logout e renovação silenciosa rodam inteiramente dentro da
infraestrutura da própria Societiza.

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Escopo da spec | Cobre 5 fluxos em um único documento: login, logout, renovação silenciosa, forgot password, reset password. Registro via convite fica fora (coberto por `identity-invitation-links`). Auto-registro público fica fora (spec separada futura). |
| Método de autenticação no MVP | E-mail e senha, apenas. Google, MFA, SSO e OAuth são horizonte futuro. |
| Perfis suportados | Todos os três (SystemAdmin, AccountancyAdmin, AccountancyEmployee) usam o mesmo fluxo de autenticação. O que muda entre perfis é autorização pós-login, não autenticação. |
| Renovação de sessão | Silenciosa e automática. O usuário não interage com o processo de renovação. Não há "lembrar de mim" como opção separada — renovação silenciosa é o default. |
| Logout | Encerramento imediato da sessão atual. Sessão encerrada não permite continuar usando a plataforma. |
| Efeito de reset de senha sobre sessões | Redefinir a senha desconecta o usuário de todos os dispositivos automaticamente, conforme já registrado em `identity-context.md`. |
| Expiração do link de reset | 1 hora. Depois desse prazo, o usuário precisa solicitar um link novo. |
| Resposta do forgot password | Confirmação genérica, independente do e-mail existir na base. Evita que o formulário vire ferramenta de enumeração de contas. |
| Uso único do link de reset | Sim. Uma vez que a senha é redefinida por aquele link, o link se torna inválido. Se o usuário precisa redefinir de novo, solicita um novo link. |
| Canal de recuperação | Somente e-mail. SMS, WhatsApp e outros canais estão fora. |
| Fonte de verdade | Backend. Toda validação de credenciais, tokens e expiração é feita no servidor. O frontend apenas reflete o estado da sessão. |
| Status da implementação | Backend já implementado em `api.societiza`. Esta spec documenta o que foi construído para servir de base para o logic.md e o integration.md subsequentes. |

## 9. Perguntas abertas

Nenhuma. Todas as decisões de produto estão fechadas. O comportamento
dos cinco fluxos foi definido consistentemente com o que já está
registrado em `identity-context.md` e com o padrão estabelecido pela
spec `identity-invitation-links`. As decisões remanescentes são
internas à camada de lógica (contratos de tokens, campos expostos,
códigos de erro) e pertencem ao `logic.md` — não à spec de produto.