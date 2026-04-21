---
status: approved
spec: identity-auth
domain: identity
created_at: 2026-04-20
---

# Autenticação — Lógica de Negócio

## 1. Fluxo por perfil

Os cinco fluxos desta feature (login, logout, renovação silenciosa, forgot
password, reset password) são idênticos em comportamento para os três perfis
da plataforma. O que distingue perfis é o que cada um faz depois de
autenticado — não como autentica. Esta seção descreve o fluxo do ponto de
vista de cada perfil, destacando apenas as diferenças de dados carregados
pela sessão.

### SystemAdmin

O SystemAdmin opera sem vínculo a nenhuma contabilidade.

**Login:**

1. Acessa a tela de login e informa e-mail e senha.
2. O sistema valida as credenciais.
3. Se corretas, a sessão é estabelecida e o SystemAdmin é direcionado à área
   geral da plataforma. A sessão carrega o perfil `SystemAdmin` e não carrega
   `accountancyId`.
4. Se incorretas, recebe mensagem genérica de credenciais inválidas. Nada é
   revelado sobre se o e-mail existe ou não.

**Logout:**

1. Clica em sair a partir de qualquer tela autenticada.
2. A sessão atual é encerrada imediatamente — o refresh em curso deixa de
   ser aceito no mesmo instante.
3. Volta à tela de login.

**Renovação silenciosa:** acontece em segundo plano enquanto o SystemAdmin
está usando a plataforma. Não há interação visível — a sessão simplesmente
se mantém viva.

**Forgot password e reset password:** idênticos aos demais perfis. Ver
abaixo.

### AccountancyAdmin

O AccountancyAdmin opera sempre dentro da sua própria contabilidade.

**Login:**

1. Acessa a tela de login e informa e-mail e senha.
2. O sistema valida as credenciais.
3. Se corretas, a sessão é estabelecida e o AccountancyAdmin é direcionado à
   área da sua contabilidade. A sessão carrega o perfil `AccountancyAdmin` e
   o `accountancyId` do tenant ao qual ele pertence.
4. Se incorretas, recebe mensagem genérica de credenciais inválidas.

**Logout, renovação silenciosa, forgot e reset:** idênticos aos demais
perfis.

### AccountancyEmployee

O AccountancyEmployee opera dentro da contabilidade à qual foi convidado.

**Login:**

1. Acessa a tela de login e informa e-mail e senha.
2. O sistema valida as credenciais.
3. Se corretas, a sessão é estabelecida e o AccountancyEmployee é direcionado
   à área operacional da sua contabilidade. A sessão carrega o perfil
   `AccountancyEmployee` e o `accountancyId` do tenant ao qual ele pertence.
4. Se incorretas, recebe mensagem genérica de credenciais inválidas.

**Logout, renovação silenciosa, forgot e reset:** idênticos aos demais
perfis.

### Fluxos comuns (aplicáveis aos três perfis)

**Renovação silenciosa da sessão:**

1. Enquanto o usuário está usando a plataforma, o token de acesso curto
   aproxima-se do fim de validade.
2. O frontend percebe a expiração (direta ou por receber uma negativa de
   acesso em uma requisição) e aciona a renovação usando a credencial de
   sessão persistente que o navegador mantém.
3. O sistema valida a credencial de sessão persistente. Se válida, emite um
   novo token de acesso e um novo marcador de sessão persistente, invalidando
   o anterior.
4. O usuário não percebe nada. Continua a ação que estava realizando.
5. Se a credencial de sessão persistente estiver inválida, expirada ou já
   tiver sido consumida, o sistema encerra a sessão e o frontend redireciona
   para a tela de login.

**Forgot password (solicitação do link):**

1. Na tela de login, o usuário clica em "esqueci minha senha".
2. Informa o e-mail associado à conta.
3. O sistema sempre retorna uma confirmação genérica de que, se o e-mail
   existir, o link foi enviado. A resposta é a mesma independente de o
   e-mail existir ou não.
4. Se o e-mail existe, o sistema gera um link de redefinição com prazo de
   1 hora e o envia ao e-mail informado. O envio do e-mail acontece de
   forma assíncrona — a resposta ao usuário não espera pela entrega.
5. O usuário recebe o e-mail com o link de redefinição.

**Reset password (consumo do link):**

1. O usuário abre o e-mail e clica no link de redefinição.
2. O link abre a tela de redefinição no frontend, carregando o token.
3. O usuário informa a nova senha e a confirmação.
4. O frontend envia token + nova senha ao sistema.
5. O sistema valida o token (existe, não foi usado, não expirou) e os
   requisitos mínimos da senha.
6. Se tudo válido, a senha é atualizada, o token é marcado como usado, e
   todas as sessões ativas do usuário em qualquer dispositivo são
   encerradas.
7. O usuário é redirecionado para a tela de login. Precisa entrar com a
   nova senha.
8. Se o token for inválido, expirado ou já utilizado, o sistema rejeita com
   mensagem clara. O usuário precisa solicitar um novo link de redefinição.

### Ações proibidas (comuns aos três perfis)

- Acessar qualquer área protegida sem sessão ativa: o sistema rejeita com
  negativa de acesso e o frontend redireciona para a tela de login.
- Renovar uma sessão encerrada (por logout, reset de senha ou expiração
  da credencial de sessão persistente): o sistema rejeita. O frontend
  redireciona para login.
- Reutilizar um link de redefinição de senha já consumido: o sistema
  rejeita com mensagem de link inválido.
- Reutilizar um link de redefinição de senha expirado (mais de 1 hora após
  a solicitação): o sistema rejeita com mensagem de link inválido.

---

## 2. Estados e transições

### Sessão do usuário

Uma sessão de usuário existe em um de três estados em linguagem de produto:

**Ativa**
O usuário está autenticado. Possui um token de acesso válido (curto) e uma
credencial de sessão persistente válida (mais longa, mantida no navegador).
Pode operar na plataforma normalmente.

**Expirada silenciosamente**
O token de acesso venceu, mas a credencial de sessão persistente ainda é
válida. Transitório — dura apenas o tempo necessário para a renovação
silenciosa ocorrer. Do ponto de vista do usuário, é invisível.

**Encerrada**
A sessão não existe mais. Nem token de acesso, nem credencial de sessão
persistente são aceitos. Estado terminal para aquela sessão específica —
o usuário precisa fazer login novamente para abrir uma nova sessão.

**Diagrama de transições:**

```
[Login bem-sucedido]    → Ativa
Ativa                   → Expirada silenciosamente   (prazo curto do token de acesso vencido)
Expirada silenciosamente → Ativa                      (renovação silenciosa bem-sucedida)
Ativa                   → Encerrada                  (logout manual do usuário)
Ativa                   → Encerrada                  (reset de senha concluído pelo próprio ou de outra sessão do mesmo usuário)
Expirada silenciosamente → Encerrada                  (credencial de sessão persistente inválida, expirada ou já consumida)
```

Uma sessão encerrada não retorna a nenhum outro estado. Login cria uma sessão
nova — não reativa a anterior.

### Credencial de sessão persistente (refresh)

A credencial de sessão persistente que mantém a sessão viva entre trocas de
token de acesso tem seus próprios estados:

**Válida**
Dentro do prazo de 7 dias da sua emissão e ainda não consumida. Pode ser
apresentada ao sistema para renovar a sessão.

**Consumida (rotacionada)**
Foi apresentada ao sistema em uma renovação bem-sucedida, e uma nova
credencial foi emitida no lugar. Estado terminal — não pode mais ser usada.

**Revogada**
Foi invalidada explicitamente pelo sistema por logout, por reset de senha do
usuário, ou como parte da rotação durante uma renovação. Estado terminal.

**Expirada**
Passaram-se mais de 7 dias desde a emissão sem que fosse usada. Estado
terminal.

**Diagrama de transições:**

```
[Emitida no login]     → Válida
[Emitida na renovação] → Válida
Válida                 → Consumida   (usada em renovação bem-sucedida; nova credencial emitida)
Válida                 → Revogada    (logout manual, reset de senha, ou rotação)
Válida                 → Expirada    (7 dias passados sem uso)
```

Credenciais em estado terminal nunca voltam a ser válidas. Uma nova
credencial é criada do zero no próximo login ou na próxima renovação.

### Token de redefinição de senha

**Válido**
Dentro do prazo de 1 hora desde a solicitação e ainda não consumido. Pode
ser apresentado para redefinir a senha.

**Consumido**
Foi usado com sucesso em uma redefinição. Estado terminal.

**Expirado**
Passou-se mais de 1 hora desde a solicitação sem que fosse usado. Estado
terminal.

**Diagrama de transições:**

```
[Solicitação de forgot] → Válido
Válido                  → Consumido   (usuário redefiniu a senha com sucesso)
Válido                  → Expirado    (1 hora passada sem uso)
```

Tokens em estado terminal não voltam a ser válidos. Se o usuário precisa
redefinir novamente, solicita um novo link por forgot password.

---

## 3. Regras de negócio

### Login
- E-mail e senha são obrigatórios.
- Validação de credenciais é feita no servidor. O frontend nunca verifica
  senha localmente.
- Credenciais inválidas (e-mail inexistente ou senha errada) retornam a
  mesma mensagem genérica — o sistema não distingue os dois casos para o
  usuário final.
- Login bem-sucedido emite um token de acesso curto e uma credencial de
  sessão persistente, ambos específicos daquela sessão.
- A credencial de sessão persistente é armazenada em cookie gerenciado pelo
  navegador (o frontend não tem acesso direto ao seu conteúdo).
- O token de acesso é entregue no corpo da resposta para uso no cabeçalho
  de autorização das requisições subsequentes.
- A sessão carrega o perfil do usuário e, se aplicável, o `accountancyId` do
  tenant ao qual ele pertence. SystemAdmin não carrega `accountancyId`.

### Logout
- Logout encerra a sessão atual imediatamente.
- A credencial de sessão persistente usada por aquela sessão é revogada no
  mesmo instante — não pode mais ser aceita em nenhuma renovação futura.
- O cookie da credencial de sessão persistente é removido do navegador.
- O token de acesso já emitido permanece válido até seu prazo curto vencer
  naturalmente — não há revogação imediata do lado do servidor, e isso é
  aceito porque o prazo é curto e o frontend descarta o token em memória
  ao sair.
- Logout requer sessão autenticada. Tentar fazer logout sem sessão ativa é
  rejeitado.

### Renovação silenciosa
- A renovação usa exclusivamente a credencial de sessão persistente
  enviada automaticamente pelo navegador via cookie. Não requer cabeçalho
  de autorização.
- Toda renovação bem-sucedida **rotaciona** a credencial: a antiga é
  revogada e uma nova é emitida.
- Uma credencial de sessão persistente só pode ser usada uma vez em
  renovação — apresentá-la novamente depois da rotação é rejeitado.
- A renovação também reemite o token de acesso com os dados atualizados do
  usuário (perfil, `accountancyId`, marcador de segurança).
- A renovação regenera o marcador de segurança do usuário a cada rotação,
  invalidando qualquer token de acesso emitido antes daquela rotação que
  ainda esteja em circulação.
- Se a credencial de sessão persistente está inválida (revogada,
  consumida ou expirada), a renovação falha e a sessão é considerada
  encerrada.

### Forgot password
- O e-mail é o único dado solicitado.
- A resposta ao frontend é a mesma independente de o e-mail existir na
  base — confirmação genérica sempre. Isso impede que o formulário seja
  usado para descobrir quais e-mails estão cadastrados.
- Se o e-mail existe, um token de redefinição é gerado com prazo de 1 hora
  e o link é enviado por e-mail.
- O envio do e-mail é assíncrono — a resposta ao frontend não espera pela
  entrega efetiva.
- Cada solicitação de forgot password gera um novo token independente. Não
  há invalidação automática de tokens anteriores pendentes — basta que
  apenas um deles seja consumido para invalidar o uso subsequente de
  qualquer outro, por efeito de reset de senha.

### Reset password
- Token, nova senha e confirmação de senha são obrigatórios.
- Nova senha e confirmação devem ser iguais — o frontend pode validar essa
  equivalência antes do envio; o servidor valida novamente.
- A nova senha deve ter pelo menos 8 caracteres. Requisitos de complexidade
  adicional (letra maiúscula, minúscula, dígito) são política de produto a
  ser aplicada consistentemente no frontend e no servidor.
- O token deve existir, estar dentro do prazo de 1 hora e ainda não ter
  sido usado. As três condições são verificadas no momento do consumo.
- Token inválido (inexistente, expirado ou já usado) é rejeitado com a
  mesma categoria de erro — o sistema não distingue publicamente a causa
  para o usuário final.
- Reset de senha bem-sucedido:
  - Atualiza a senha do usuário.
  - Marca o token como consumido.
  - **Revoga todas as credenciais de sessão persistente do usuário em
    todos os dispositivos.** Qualquer sessão aberta perde a capacidade
    de renovar — na próxima tentativa, a sessão é encerrada.
  - Regenera o marcador de segurança do usuário, invalidando tokens de
    acesso em circulação.
- O usuário precisa fazer login novamente com a nova senha após o reset —
  não é autologado.

### Invariantes
- Sessão encerrada não pode ser reaberta sem novo login.
- Credencial de sessão persistente, uma vez consumida, revogada ou
  expirada, nunca volta a ser válida.
- Token de redefinição de senha, uma vez consumido ou expirado, nunca
  volta a ser válido.
- Reset de senha bem-sucedido sempre desconecta o usuário de todos os
  dispositivos — é efeito colateral garantido pelo sistema, não uma
  opção do usuário.
- Validação de credenciais, tokens e expiração é sempre autoritativa no
  servidor. O frontend apenas reflete o estado informado pelo servidor.
- Toda sessão autenticada carrega o perfil do usuário. Sessões de
  AccountancyAdmin e AccountancyEmployee carregam adicionalmente o
  `accountancyId` — que é o contorno multi-tenant de tudo que vem depois
  (societário, convites, dashboard).

---

## 4. Pontos de integração

```
Frontend precisa saber:
  - Se o login foi bem-sucedido ou falhou, com a categoria do erro (credencial inválida vs. erro de validação de formato) — para apresentar a mensagem adequada
  - O token de acesso recebido no login e em cada renovação — o frontend o usa no cabeçalho de autorização das requisições subsequentes
  - O perfil do usuário autenticado (SystemAdmin, AccountancyAdmin, AccountancyEmployee) — para decidir a área inicial após login e o que mostrar na navegação
  - O `accountancyId` do usuário autenticado, quando aplicável — para contextualizar chamadas de features multi-tenant
  - Quando um token de acesso está prestes a expirar ou recebeu negativa de acesso de uma requisição protegida — para disparar a renovação silenciosa
  - O resultado da renovação: sucesso (novo token recebido) ou falha (sessão precisa ser encerrada e usuário redirecionado ao login)
  - Que o cookie de sessão persistente é gerenciado automaticamente pelo navegador — o frontend não precisa ler seu conteúdo, apenas garantir que requisições de renovação e logout sejam feitas de forma a enviar cookies
  - Que forgot password sempre confirma envio, independente do e-mail existir — a mensagem exibida ao usuário é a mesma em todos os casos
  - Que reset password pode falhar por token inválido, expirado ou já usado, e que todas essas falhas apresentam a mesma categoria de mensagem ao usuário (convite para solicitar novo link)
  - Os requisitos mínimos da nova senha no fluxo de reset — para feedback imediato no formulário (o servidor valida novamente)
  - Que após reset bem-sucedido, o usuário precisa fazer login com a nova senha — não é autologado

Backend precisa garantir:
  - Validação atômica de credenciais no login — senha é verificada contra o hash armazenado
  - Mensagem de erro idêntica para "e-mail não existe" e "senha errada" no login — impede enumeração de contas
  - Emissão do token de acesso e da credencial de sessão persistente de forma atômica no login — se um dos dois falhar, nenhum é emitido
  - Entrega da credencial de sessão persistente exclusivamente via cookie HttpOnly, Secure, SameSite=Strict, com prazo de 7 dias — o frontend não tem acesso ao conteúdo
  - Aceitação da credencial de sessão persistente exclusivamente via cookie nas requisições de renovação e logout — nunca via cabeçalho ou corpo
  - Rotação atômica da credencial na renovação: revogação da antiga + emissão da nova acontecem na mesma operação, impedindo que a antiga continue válida após o uso
  - Regeneração do marcador de segurança a cada rotação — invalida tokens de acesso antigos em circulação
  - Resposta idêntica em forgot password independente de o e-mail existir — o frontend não distingue
  - Enfileiramento do envio de e-mail de redefinição de forma assíncrona — a resposta ao frontend é imediata
  - Entrega do link no e-mail apontando para a URL correta do frontend, com o token como parâmetro
  - Prazo exato de 1 hora para o token de redefinição, validado no momento do consumo — não depende exclusivamente de job de varredura
  - Validação das três condições de uso do token de reset (existe, não expirado, não usado) em uma única operação atômica — evita race condition de dois cliques simultâneos consumindo o mesmo token
  - Marcação atômica do token como consumido + atualização da senha + revogação de todas as credenciais de sessão persistente + regeneração do marcador de segurança em reset bem-sucedido — tudo ou nada
  - Revogação explícita da credencial de sessão persistente no logout, e remoção do cookie do navegador
  - Rejeição de logout sem sessão autenticada, com categoria de erro distinguível pelo frontend
  - Rejeição de renovação quando a credencial de sessão persistente é inválida, expirada ou já consumida, com categoria de erro distinguível pelo frontend — para que o frontend saiba que precisa redirecionar ao login
  - Isolamento do contexto de autenticação por usuário: nenhum dado de sessão de um usuário é jamais exposto em resposta a requisição de outro usuário
  - Categorização estável dos erros retornados (credencial inválida, token inválido/expirado/usado, validação de formato) — para que o frontend possa mapear cada caso a uma mensagem adequada sem depender de texto livre
```

---

## 5. Casos de borda

### Token de acesso expirado no meio de uma ação
O usuário está em uma operação longa e, no envio da requisição final, o
token de acesso já venceu. **Decisão:** o frontend recebe negativa de
acesso, dispara a renovação silenciosa usando a credencial de sessão
persistente, recebe um novo token e reenvia a requisição original. O
usuário não percebe. Se a renovação falhar, a sessão é encerrada e o
usuário é redirecionado ao login — a ação em curso é perdida.

### Tentativa de renovação após logout
O cookie de credencial de sessão persistente foi removido, mas uma
requisição antiga tenta renovar usando uma credencial já revogada.
**Decisão:** o sistema rejeita a renovação. O frontend trata como sessão
encerrada e redireciona ao login.

### Renovação simultânea em múltiplas abas
O usuário tem a plataforma aberta em duas abas. Ambas percebem que o
token de acesso está próximo do vencimento e disparam renovação ao mesmo
tempo. **Decisão:** apenas a primeira sucede (consome a credencial
atual e recebe a nova). A segunda recebe negativa porque a credencial já
foi consumida — o frontend da segunda aba trata como sessão encerrada e
pode: (a) tentar ler o novo token da primeira aba via mecanismo próprio
do frontend (ex: storage compartilhado), ou (b) redirecionar ao login.
O backend não arbitra entre abas — apenas garante a rotação atômica.
O frontend decide como coordenar abas; essa coordenação é detalhe de
implementação do cliente e não altera o contrato do servidor.

### Reset de senha enquanto outra sessão está ativa
O usuário redefine a senha a partir do link de reset, e tem uma sessão
ativa em outro dispositivo (ou aba) no mesmo momento. **Decisão:** todas
as credenciais de sessão persistente do usuário são revogadas no reset.
O outro dispositivo continua funcionando até o próximo token de acesso
vencer — e aí a renovação falha e a sessão é encerrada. Aceito: o delay
de no máximo 15 minutos é o prazo curto do token de acesso.

### Dois pedidos de forgot password em sequência
O usuário solicita forgot password, não consegue acessar o e-mail
rapidamente, solicita de novo. Dois tokens válidos coexistem por até 1
hora. **Decisão:** aceito. Qualquer um dos tokens pode ser consumido. No
momento em que um deles é consumido, o reset revoga todas as sessões e
regenera o marcador de segurança — os tokens pendentes de reset não são
invalidados explicitamente, mas perdem qualquer utilidade porque o
próximo reset do mesmo usuário seguirá o mesmo caminho. Se o usuário
tentar usar o token antigo depois de já ter resetado a senha, ele será
aceito (se ainda dentro do prazo) e redefinirá a senha novamente —
aceito nesta versão porque significa apenas uma nova troca de senha
iniciada por quem tem acesso ao e-mail.

### Link de reset clicado após expirar
O usuário recebe o e-mail, demora mais de 1 hora para clicar, abre o
link. **Decisão:** o sistema rejeita no consumo com mensagem clara de
link inválido ou expirado. O frontend apresenta a mensagem e oferece
atalho para solicitar novo link via forgot password.

### Link de reset clicado depois de já ter sido usado
O usuário redefine a senha com sucesso, e mais tarde clica de novo no
mesmo link (por distração ou porque deixou o e-mail aberto). **Decisão:**
o sistema rejeita no consumo. O frontend apresenta mensagem de link
inválido e oferece atalho para solicitar novo link via forgot password.

### Tentativa de login com conta recém-convidada que não finalizou cadastro
O convite foi enviado, mas o convidado ainda não acessou o link nem criou
a conta. Alguém tenta login com esse e-mail. **Decisão:** login falha com
credencial inválida — a conta ainda não existe. Nenhuma distinção é feita
entre "conta inexistente" e "conta existente com senha errada".

### Credencial de sessão persistente no limite do prazo de 7 dias
O usuário volta à plataforma depois de exatamente 7 dias de inatividade.
**Decisão:** a credencial está expirada. A renovação falha. O frontend
redireciona ao login. O usuário reentra com e-mail e senha e uma sessão
nova é aberta.

### Troca de perfil ou de contabilidade durante sessão ativa
O usuário é promovido (ex: de AccountancyEmployee para AccountancyAdmin)
ou migrado entre contabilidades enquanto tem uma sessão ativa. **Decisão:**
gestão de usuário que altera perfil ou `accountancyId` está fora do
escopo desta spec. Quando essa feature existir, o mecanismo natural é
regenerar o marcador de segurança do usuário, o que força a renovação
silenciosa a reemitir o token de acesso com os dados atualizados na
próxima rotação. Nenhum tratamento especial é necessário nesta feature.

### Logout em rede instável
O usuário clica em sair, mas a requisição de logout falha por perda de
conexão antes de chegar ao servidor. **Decisão:** o frontend, diante da
falha, ainda assim deve (a) remover o token de acesso em memória e (b)
redirecionar à tela de login. O cookie da credencial de sessão
persistente pode permanecer válido no navegador até ser usado em uma
renovação (ou até expirar em 7 dias). O usuário, ao voltar, precisa
logar novamente — a sessão antiga não é automaticamente retomada. Aceito
nesta versão: o risco residual é baixo e o comportamento de defesa é do
lado do cliente.

### Mensagens idênticas em falhas de reset
Token inválido, expirado e já usado produzem mensagens de categoria
equivalente ao usuário final. **Decisão:** aceito. O usuário não precisa
distinguir entre as três causas — a ação corretiva é a mesma (solicitar
novo link). A distinção existe internamente no servidor para
observabilidade, mas não é exposta publicamente.
