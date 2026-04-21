---
status: approved
spec: identity-auth
domain: identity
created_at: 2026-04-20
updated_at: 2026-04-20
---

# Autenticação — Contrato de Integração Backend → Frontend

## 1. Identificação da feature

- **Nome da feature**: Autenticação (Login, Logout, Refresh, Forgot Password, Reset Password)
- **Módulo backend responsável**: `Identity` (`src/Modules/Identity/*`, endpoints em `src/Web/Endpoints/Modules/Identity/Auth/`)
- **Status atual**: `existente`
- **Data da última atualização**: 2026-04-20
- **Responsável técnico**: Gabriel Carvalho

Os cinco fluxos estão implementados e expostos em `/api/v1/identity/auth/*`. A feature é a camada fundacional do produto — toda rota protegida da plataforma depende dela.

---

## 2. Objetivo de negócio da feature

Autenticação é o portão de entrada da Societiza. Depois que uma pessoa cria sua conta (auto-registro futuro ou via link de convite já implementado), ela precisa de uma forma confiável de entrar todos os dias, permanecer dentro do sistema enquanto trabalha e recuperar o acesso se esquecer a senha. Esta feature cobre a camada inteira: login, logout, renovação silenciosa da sessão e o fluxo de recuperação de senha (solicitação por e-mail e redefinição via link).

Do ponto de vista do usuário, a experiência é quase invisível quando funciona. Ele entra uma vez com e-mail e senha e, a partir dali, o sistema mantém a sessão viva em segundo plano via rotação automática do refresh token — sem redirects para login no meio de um processo, sem reautenticação entre abas. Quando quer sair, clica em sair e a sessão é encerrada imediatamente. Quando esquece a senha, pede um link, recebe por e-mail, cria uma nova senha e volta a operar.

A feature serve todos os três perfis da plataforma (`SystemAdmin`, `AccountancyAdmin`, `AccountancyEmployee`) com comportamento idêntico — o que distingue perfis é a autorização *depois* do login, carregada pelas claims do access token. Ela entra em **todo acesso à plataforma depois do cadastro inicial** e é pré-requisito de todas as features protegidas (societário, convites, dashboard futuro, notificações).

---

## 3. Escopo atual versus escopo futuro

**O que já está implementado no backend hoje:**

- `POST /api/v1/identity/auth/login` — autenticação por e-mail e senha, emite access token (JWT, 15 min) + refresh token (JWT, 7 dias).
- `POST /api/v1/identity/auth/refresh` — rotação atômica do refresh token via cookie `refresh_token`, reemissão do access token com claims atualizadas.
- `POST /api/v1/identity/auth/logout` — revoga o refresh token atual, limpa o cookie e regenera o `SecurityStamp` do usuário.
- `POST /api/v1/identity/auth/forgot-password` — gera token de reset (raw 32 bytes base64, hash SHA-256 armazenado), enfileira envio via Hangfire + Brevo SMTP. Responde sempre 200 OK, independente de o e-mail existir.
- `POST /api/v1/identity/auth/reset-password` — consome o token (validação atômica: existe, não expirado, não usado), atualiza a senha, revoga todos os refresh tokens ativos do usuário e regenera o `SecurityStamp`.
- Cookie `refresh_token` HttpOnly, Secure, SameSite=Strict, com expiração de 7 dias, gerido pelas extensões `HttpContextExtensions.SetRefreshTokenCookie` / `DeleteRefreshTokenCookie`.
- Hash de senha via PBKDF2 SHA-256, 100 000 iterações, salt de 16 bytes, hash de 32 bytes (`PasswordService`).
- Projeções MongoDB para `RefreshToken` e `PasswordResetToken` (via `ProjectionRefreshTokenEventHandler` e `ProjectionPasswordResetTokenEventHandler`).

**O que ainda não está implementado:**

- Auto-registro público (fora do escopo desta spec; permanece futuro).
- Alteração de senha a partir do perfil do usuário autenticado (gap conhecido; entra em spec de gestão de usuário).
- MFA, SSO e login via Google/Microsoft.
- Bloqueio de conta por excesso de tentativas, captcha, rate limit ao nível de feature.
- Gestão de sessões ativas pelo usuário (listar dispositivos, encerrar sessão remota).

**O que existe no domínio mas não está exposto por endpoint:**

- `User.RevokeAllRefreshTokens()` e `User.RegenerateSecurityStamp()` — invocados apenas internamente pelos handlers de `Logout` e `ResetPassword`.
- Eventos `UserPasswordUpdated` e `UserSecurityStampRegenerated` — persistidos no event store, mas não há read model público que os exponha.

**O que é apenas visão futura de produto:**

- Login via Google como método adicional.
- Sessão compartilhada com portal do cliente final.
- Políticas de senha configuráveis por contabilidade.

**O que ainda está em discussão:**

- Não identificado no código. A spec registra "nenhuma pergunta aberta" (seção 9 da spec).

---

## 4. Contexto de produto e semântica da feature

O usuário percebe a autenticação como um utilitário silencioso: entra uma vez e fica dentro. O backend reforça essa percepção por três mecanismos combinados, todos observáveis no código:

1. **Access token curto (15 min)** entregue no corpo da resposta de `POST /login` e `POST /refresh` — o frontend o injeta no header `Authorization: Bearer <token>` de cada requisição protegida.
2. **Refresh token longo (7 dias)** entregue exclusivamente via cookie HttpOnly `refresh_token` — o frontend não lê nem envia manualmente; o browser apenas o transporta automaticamente em requisições ao mesmo domínio.
3. **Rotação atômica** em cada refresh: o token antigo é revogado no mesmo aggregate commit em que o novo é emitido, e o `SecurityStamp` do usuário é regenerado — invalida quaisquer access tokens em circulação cuja claim `security_stamp` não bata com a atual.

**Sucesso** = `200 OK` com `accessToken` no corpo + cookie `refresh_token` renovado (login, refresh) ou apenas `200 OK` com cookie limpo (logout, reset, forgot).

**Erro funcional:**
- Credencial inválida no login → `400 Bad Request`, `Auth.InvalidCredentials`, mensagem `Invalid email or password` (idêntica para "e-mail inexistente" e "senha errada" — anti-enumeração).
- Refresh token ausente, inválido, expirado ou já consumido → `400 Bad Request`, `Auth.InvalidToken`, mensagem `Refresh token is invalid or expired`.
- Reset token inválido/expirado/usado → `404 Not Found`, `Auth.InvalidToken`, mensagem `Password reset token is invalid or expired`.
- Erro de validação de campo → `400 Bad Request` via `ValidationException`.

**Estados relevantes** (observáveis nas projeções MongoDB):
- Refresh token: `ativo` (não revogado, não expirado) → `revogado` (logout, reset, rotação por refresh) ou `expirado` (7 dias).
- Password reset token: `válido` (não usado, não expirado) → `usado` ou `expirado` (1 hora).

---

## 5. Dependências e relações com outras features

**Features das quais esta depende:**

- `FrontendOptions.BaseUrl` configurado em `appsettings`/`.env.local` — usado pelo `ForgotPasswordHandler` para montar o link `{BaseUrl}/auth/reset-password?token={rawToken}` enviado por e-mail.
- `JwtOptions` (`Key`, `Issuer`, `Audience`) — `src/Modules/Identity/Identity.Infrastructure/Options/JwtOptions.cs` — assinatura dos JWTs.
- `IEmailService` (Core) + Hangfire — envio assíncrono do e-mail de reset via Brevo SMTP.
- `identity-invitation-links` — cria o usuário que esta feature autentica (via `POST /invitation-link/register`).

**Features que consomem esta feature:**

- Toda rota protegida da plataforma: `Accountancy`, `WorkflowTemplate`, `Identity.Invitation` (exceto `GetInvitationLinkByToken` público).
- `HasRoleHandler` (`Identity.Application/Authorization/HasRoleHandler.cs`) — lê a claim `role` do access token para autorizar por papel.
- Qualquer handler que leia `accountancy_id` do `ClaimsPrincipal` para filtrar por tenant.

**Posição no produto:**

Central. É a camada fundacional do módulo Identity — nenhuma feature multi-tenant da Societiza existe sem ela.

**Dependências de dados de outro módulo:**

Nenhuma direta. O backend não consulta o módulo Accountancy no fluxo de login; o `accountancyId` é armazenado na projeção `User` do Identity (denormalizado no registro via convite) e entregue como claim do JWT.

**Ligação com dashboard, workflow, accountancy, notificações:**

- Dashboard, workflow, accountancy — consumem o access token emitido aqui.
- Notificações — compartilham o canal de e-mail transacional (Brevo SMTP) com o fluxo de forgot password, mas são features independentes.

---

## 6. Rotas públicas

| Método | Path | Tipo | Finalidade | Params de rota | Pronta para frontend? |
|---|---|---|---|---|---|
| POST | `/api/v1/identity/auth/login` | command | Autenticar por e-mail e senha; retorna `accessToken` + cookie `refresh_token` | — | sim |
| POST | `/api/v1/identity/auth/logout` | command | Revogar refresh token atual e limpar cookie | — | sim (requer `Authorization: Bearer`) |
| POST | `/api/v1/identity/auth/refresh` | command | Rotacionar par access/refresh usando cookie `refresh_token` | — | sim |
| POST | `/api/v1/identity/auth/forgot-password` | command | Disparar e-mail de redefinição (sempre 200 OK) | — | sim |
| POST | `/api/v1/identity/auth/reset-password` | command | Consumir token de reset e definir nova senha | — | sim |

Nenhuma operação é `query`. Todas são comandos MediatR com lado efeito (emit/revoke token, enfileirar job, atualizar senha).

- `Login`, `Refresh`, `ForgotPassword`, `ResetPassword` — anônimas.
- `Logout` — requer `[Authorize]` (access token válido no header).

Rotas reais vivem sob `UsePathBase("/api")` + `/v1/identity/auth/...`.

---

## 7. Contrato de request

### Request — Login (`POST /api/v1/identity/auth/login`)

Corpo (`LoginRequest`):
- `email`: `string`, obrigatório, formato de e-mail válido. Validado em `LoginValidator`.
- `password`: `string`, obrigatório, não-vazio. Não há exigência de tamanho mínimo no login (a validação de complexidade só aplica em `RegisterUser` e `ResetPassword`).

Transformações obrigatórias no frontend:
- Nenhuma. Campos são strings planas.

Validações relevantes:
- `EmailAddress()` do FluentValidation — e-mail em formato padrão.
- Erro de validação → `400 Bad Request` com envelope de ValidationProblem.

### Request — Logout (`POST /api/v1/identity/auth/logout`)

Corpo: **nenhum** (endpoint `.WithoutRequest`).

O refresh token é lido exclusivamente do cookie `refresh_token` pelo endpoint e repassado ao `LogoutCommand`. O frontend precisa garantir que a requisição é feita com `credentials: 'include'` (ou equivalente) para o browser enviar o cookie.

Header obrigatório:
- `Authorization: Bearer <accessToken>` — endpoint tem `[Authorize]`.

Validações relevantes:
- `LogoutValidator` exige `RefreshToken` não-vazio — se o cookie estiver ausente, retorna `400 Bad Request` por validação, **antes** de chegar no `[Authorize]` apenas quando o access token também está ausente/inválido. Ordem observada no código: `[Authorize]` atua primeiro; cookie ausente com access token válido dispara validação.

### Request — Refresh (`POST /api/v1/identity/auth/refresh`)

Corpo: **nenhum** (endpoint `.WithoutRequest`).

O refresh token é lido do cookie `refresh_token`. **Não requer** `Authorization` header.

Transformações obrigatórias no frontend:
- Enviar a requisição com `credentials: 'include'` (ou `withCredentials: true` em XHR/Axios) para o browser transportar o cookie.

Validações relevantes:
- `RefreshValidator` exige `RefreshToken` não-vazio — cookie ausente → `400 Bad Request`.

### Request — ForgotPassword (`POST /api/v1/identity/auth/forgot-password`)

Corpo (`ForgotPasswordRequest`):
- `email`: `string`, obrigatório, formato de e-mail válido.

Transformações obrigatórias no frontend:
- Nenhuma.

Validações relevantes:
- `ForgotPasswordValidator` exige `email` não-vazio e válido.
- Se o e-mail é válido mas não existe na base → o handler retorna `Success` silenciosamente (anti-enumeração). O frontend recebe o mesmo `200 OK` que receberia se o e-mail existisse.

### Request — ResetPassword (`POST /api/v1/identity/auth/reset-password`)

Corpo (`ResetPasswordRequest`):
- `token`: `string`, obrigatório. É o `rawToken` gerado no forgot (`Convert.ToBase64String(RandomNumberGenerator.GetBytes(32))`). O frontend extrai do query string `?token=` do link do e-mail.
- `newPassword`: `string`, obrigatório. Requisitos mínimos implementados no `ResetPasswordValidator`: mínimo 8 caracteres, máximo 100 caracteres, ao menos uma letra maiúscula, ao menos uma letra minúscula, ao menos um dígito.
- `confirmPassword`: `string`, obrigatório, deve ser igual a `newPassword`.

Transformações obrigatórias no frontend:
- Nenhuma. Token é base64 URL-safe de 32 bytes — pode conter `+`, `/`, `=`. O backend aceita como veio do e-mail.

Validações relevantes:
- `Equal(x => x.NewPassword)` — FluentValidation compara confirmação com a nova senha.
- Validação do próprio token (existência, expiração, uso) é feita no handler, não no validator — retorna `404 Not Found`.

---

## 8. Contrato de response

### Response — Login (`200 OK`)

Corpo (objeto anônimo montado no endpoint — **não** é o `LoginResponse` bruto):
- `accessToken`: `string` — JWT assinado HS256. Claims: `sub` (userId, Guid), `email`, `role` (`SystemAdmin` | `AccountancyAdmin` | `AccountancyEmployee`), `security_stamp` (Guid), `accountancy_id` (Guid, **omitido para SystemAdmin**). Expiração: 15 minutos.

Cookie (side-effect):
- `refresh_token`: JWT assinado HS256, claims `sub` + `jti`, expiração 7 dias. HttpOnly, Secure, SameSite=Strict. O frontend **não** deve tentar ler.

Observação: o `LoginResponse` da camada Shared (`record LoginResponse(string AccessToken, string RefreshToken)`) **não** é serializado inteiro para o cliente — o endpoint faz `Ok(new { accessToken = response.AccessToken })` e move o `RefreshToken` para o cookie. O frontend só vê `accessToken`.

### Response — Refresh (`200 OK`)

Idêntico ao login: corpo `{ accessToken: string }` + cookie `refresh_token` rotacionado.

### Response — Logout (`200 OK`)

Corpo: vazio. O cookie `refresh_token` é removido via `Set-Cookie` com `Max-Age=0`.

### Response — ForgotPassword (`200 OK`)

Corpo: vazio. Retornado sempre, independente do e-mail existir.

### Response — ResetPassword (`200 OK`)

Corpo: vazio. A senha foi atualizada, o token marcado como usado, todos os refresh tokens ativos revogados e o security stamp regenerado. O usuário precisa logar novamente.

### Response — Erros (todos os endpoints)

Envelope `ProblemDetails` (gerado por `EndpointBaseExtensions.HandleFailure`):
- `title`: `"Validation Error"` | `"Bad Request"` | `"Not Found"` | `"Conflict"` | `"Forbidden"`.
- `type`: o `Error.Code` — ex: `"Auth.InvalidCredentials"`, `"Auth.InvalidToken"`, `"Validation.Error"`.
- `detail`: mensagem legível — ex: `"Invalid email or password"`, `"Refresh token is invalid or expired"`, `"Password reset token is invalid or expired"`.
- `status`: número HTTP.
- `errors`: `Error[]` preenchido apenas em erro de validação (FluentValidation).

### 8.1. Formato canônico versus formato de exibição

- `accessToken` e o conteúdo do cookie `refresh_token` chegam em formato JWT (string base64-url compacta, 3 segmentos `header.payload.signature`). O frontend **não decodifica** o refresh (é opaco). O access pode ser decodificado para leitura das claims — recomendado decodificar apenas para UI (role, accountancyId); validação continua sendo do backend.
- Claims de data dentro do JWT (`exp`, `iat`, `nbf`) são Unix timestamps — convenção padrão do JWT, não há customização no código.
- Mensagens de erro (`detail`) estão em **inglês** no backend (`"Invalid email or password"`, `"Refresh token is invalid or expired"`). O frontend deve mapear para textos em português ao exibir ao usuário.

### 8.2. Nullabilidade e ausência de campo

- `accountancy_id` no access token é **omitido** (claim não incluída) para usuários `SystemAdmin`. Não vem como `null` — vem ausente. O frontend precisa tratar `undefined`/chave ausente, não `null`.
- `role` é sempre presente (string enum: `SystemAdmin` | `AccountancyAdmin` | `AccountancyEmployee`).
- Em erros, `errors` só aparece em erro de validação FluentValidation — pode ser `null`/ausente nos demais.

---

## 9. Estados de negócio e transições

### Sessão do usuário

Mapeamento entre a linguagem da spec e os artefatos do código:

| Estado (spec) | Artefato no código |
|---|---|
| Ativa | access token JWT não-expirado + `RefreshToken` (entity) com `IsRevoked=false` e `ExpiresAt > UtcNow` |
| Expirada silenciosamente | access token JWT expirado + `RefreshToken` ainda válido — estado transitório durante o ciclo de refresh |
| Encerrada | `RefreshToken.IsRevoked=true` ou `ExpiresAt <= UtcNow` |

Transições implementadas:

- Login bem-sucedido → **Ativa** (evento `UserRefreshTokenAdded`).
- Ativa → Encerrada (logout) → evento `UserRefreshTokenRevoked` + `UserSecurityStampRegenerated`.
- Ativa → Expirada silenciosamente → Ativa (refresh) → 2 eventos `UserRefreshTokenRevoked` (antigo) + `UserRefreshTokenAdded` (novo) + `UserSecurityStampRegenerated`, todos no mesmo commit.
- Ativa → Encerrada (reset de senha) → múltiplos `UserRefreshTokenRevoked` (todos os tokens válidos do usuário) + `UserPasswordUpdated` + `UserPasswordResetTokenUsed` + `UserSecurityStampRegenerated`.
- Expirada silenciosamente → Encerrada → ocorre quando o `RefreshHandler` encontra `existingToken is null || !existingToken.IsValid()`.

### Refresh token

| Estado (spec) | Código |
|---|---|
| Válida | `IsRevoked=false && ExpiresAt > UtcNow` (método `IsValid()` em `RefreshToken.cs`) |
| Consumida (rotacionada) | `IsRevoked=true` após rotação em `RefreshHandler` |
| Revogada | `IsRevoked=true` com `RevokedAt` preenchido (logout ou reset) |
| Expirada | `ExpiresAt <= UtcNow` — não há job que marque expirados; é uma checagem dinâmica |

### Token de redefinição de senha

| Estado (spec) | Código |
|---|---|
| Válido | `IsUsed=false && ExpiresAt > UtcNow` (método `IsValid()` em `PasswordResetToken.cs`) |
| Consumido | `IsUsed=true && UsedAt != null` após `ResetPasswordHandler` |
| Expirado | `ExpiresAt <= UtcNow` — checagem dinâmica, sem job de varredura |

### 9.1. Ordenação, filtros e agrupamentos com semântica de negócio

Não aplicável — nenhuma listagem é exposta por esta feature. Operações são todas comandos unitários. Sem filtros implícitos, sem paginação, sem ordenação.

---

## 10. Erros esperados e edge cases

| Situação | Quando acontece | Código HTTP | `type` | Comportamento esperado no frontend |
|---|---|---|---|---|
| E-mail ou senha errados no login | `POST /login` com credenciais que não batem | `400 Bad Request` | `Auth.InvalidCredentials` | Exibir mensagem genérica: "E-mail ou senha inválidos." Não distinguir se o e-mail existe. |
| E-mail malformado no login/forgot | `email` vazio ou formato inválido | `400 Bad Request` | `Validation.Error` | Marcar o campo no formulário; usar `errors[]` se necessário para mensagem inline. |
| Senha vazia no login | `password` vazio | `400 Bad Request` | `Validation.Error` | Marcar o campo; inline. |
| Refresh token ausente (cookie não enviado) | `POST /refresh` sem cookie | `400 Bad Request` | `Validation.Error` (via `RefreshValidator`) | Redirecionar imediatamente para login. |
| Refresh token inválido/expirado/revogado | `POST /refresh` ou `POST /logout` com cookie que falha na validação JWT, ou não existe no aggregate, ou `IsRevoked=true`, ou `ExpiresAt <= UtcNow` | `400 Bad Request` | `Auth.InvalidToken` | Encerrar sessão no cliente e redirecionar para login. |
| Access token ausente/inválido em rota protegida (ex: logout) | `[Authorize]` falha | `401 Unauthorized` | — (pipeline do ASP.NET, sem `ProblemDetails` customizado) | Disparar refresh; se refresh falhar, redirecionar para login. |
| Reset token inválido/expirado/já usado | `POST /reset-password` | `404 Not Found` | `Auth.InvalidToken` | Mensagem: "Link inválido ou expirado." + CTA para `forgot-password` novamente. |
| Nova senha < 8 caracteres ou sem complexidade | `POST /reset-password` ou `POST /invitation-link/register` | `400 Bad Request` | `Validation.Error` | Marcar o campo; exibir as regras: mínimo 8 caracteres, ao menos uma maiúscula, uma minúscula e um dígito. |
| Confirmação de senha não bate | `POST /reset-password` | `400 Bad Request` | `Validation.Error` | Marcar os dois campos; mensagem inline. |
| Forgot com e-mail inexistente | `POST /forgot-password` | `200 OK` | — | **Exibir mensagem de sucesso genérica** mesmo quando o e-mail não existe. Nunca expor "e-mail não cadastrado." |

Detalhe importante sobre o fluxo de renovação: o `RefreshHandler` retorna `400 Bad Request` (não `401`) quando o refresh token é inválido. O frontend precisa tratar **qualquer falha** em `POST /refresh` como "sessão encerrada, redirecionar para login" — independente do código.

---

## 11. Semântica de autenticação e autorização

| Endpoint | Requer auth? | Contexto de usuário | Autorização |
|---|---|---|---|
| `POST /login` | não | público | público |
| `POST /refresh` | não (usa cookie) | `sub` extraído do JWT do refresh | a validação é do próprio refresh token |
| `POST /logout` | **sim** (`[Authorize]`) | access token válido + cookie | qualquer usuário autenticado |
| `POST /forgot-password` | não | público | público |
| `POST /reset-password` | não | `sub` extraído do próprio reset token (via `ProjectionModel.PasswordResetToken.UserId`) | validação é do próprio token |

Claims do access token entregues ao frontend (decodificáveis para uso de UI, **não** para validação de segurança):
- `sub` — `userId` (Guid).
- `email` — string.
- `role` — `"SystemAdmin"` | `"AccountancyAdmin"` | `"AccountancyEmployee"` (valores do `UserRole` SmartEnum).
- `security_stamp` — Guid; regenerado a cada refresh e a cada reset de senha.
- `accountancy_id` — Guid; **omitido para SystemAdmin**.

---

## 12. Impacto de UX que o frontend precisa saber

- **Login retorna apenas `accessToken` no corpo.** Refresh token está no cookie e **não é acessível via JS** (HttpOnly). Não tente armazenar refresh em `localStorage`.
- **Cookie é Secure + SameSite=Strict.** Implicação: em desenvolvimento local sem HTTPS o cookie pode não ser setado pelo browser. Usar HTTPS local ou proxy apropriado.
- **Refresh é single-use.** Qualquer chamada subsequente ao `POST /refresh` com o mesmo cookie falha (o token foi revogado no primeiro uso, e um novo cookie foi emitido no response). Em múltiplas abas, só a primeira que disparar refresh terá sucesso — as outras recebem `400`. Frontend precisa coordenar abas (ex: BroadcastChannel, storage event) ou aceitar que uma aba cairá para login.
- **Access token tem 15 min de TTL.** Estratégia sugerida: interceptor em 401 dispara refresh automaticamente e retenta a requisição original. Se refresh também falhar, redireciona para login.
- **Logout não invalida access tokens já emitidos.** Eles continuam válidos até a expiração natural (15 min) ou até o `security_stamp` mudar. Frontend deve descartar `accessToken` em memória ao fazer logout.
- **Reset de senha desloga todos os dispositivos.** Qualquer sessão aberta em outra aba/dispositivo perde a capacidade de renovar; no máximo em 15 min (TTL do access token) essa sessão cai para login.
- **Forgot password sempre retorna sucesso.** Use mensagem única: "Se o e-mail existir, você receberá um link em breve." Nunca "E-mail não encontrado."
- **Link de reset aponta para `{FrontendOptions.BaseUrl}/auth/reset-password?token={rawToken}`.** Frontend precisa ter a rota `/auth/reset-password` que leia o `token` do query string e envie ao endpoint.
- **Login não retorna dados do usuário (nome, role, accountancyId) no corpo.** Essas informações estão **apenas nas claims do access token**. O frontend decodifica o JWT (sem validar assinatura localmente) para popular contexto de usuário, **ou** faz uma chamada separada `GET /me` (que **não existe atualmente** — ver §16).
- **Redirect pós-login** é decisão do frontend baseada na claim `role`: `SystemAdmin` → área geral; `AccountancyAdmin`/`AccountancyEmployee` → área da contabilidade (usando `accountancy_id` da claim).
- Após reset bem-sucedido, **não há autologin** — frontend deve redirecionar para a tela de login com mensagem de sucesso.
- Forgot password é assíncrono no envio do e-mail, mas a resposta HTTP é síncrona em ~instantânea. O frontend não precisa aguardar delivery confirmation — apenas exibir sucesso.

---

## 13. Impacto de dados no frontend

- **Após login:** o frontend tem apenas o `accessToken` e o cookie (invisível). Qualquer leitura de perfil do usuário depende de decodificar claims ou chamar um endpoint de perfil — que não existe nesta feature.
- **Após refresh:** novo `accessToken` + novo cookie. O novo access token pode ter claims diferentes (principal caso: `security_stamp` atualizado). Frontend deve relê-las ao substituir o token em memória.
- **Após logout:** frontend deve limpar estado de usuário em memória, invalidar queries de dados protegidos e redirecionar para login.
- **Após reset-password bem-sucedido:** nenhum estado autenticado sobrevive — redirecionar para login.

Stale data: não aplicável dentro desta feature (sem listagens). Para features downstream, uma mudança de `security_stamp` (refresh/reset) implica que caches in-memory vinculados ao usuário podem estar "antigos" — mas é raro importar.

UI otimista: não aplicável. Login, logout, refresh e reset são ações que **precisam** do resultado do servidor antes de transicionar UI — não há como otimizar.

### 13.1. Identificadores estáveis e chaves de reconciliação

- `sub` (userId, Guid) — identificador **estável** do usuário. Chave primária em toda lógica downstream. Frontend deve tratar como chave de reconciliação para cache de perfil.
- `accountancy_id` — estável, mas **pode mudar** no futuro se uma feature de migração entre contabilidades for implementada (ver spec §6 e logic.md §5).
- `security_stamp` — **não é chave**. Muda a cada refresh/reset. Usar apenas para detectar "meu token está stale?".
- `jti` do refresh — identifica a emissão específica do token; não é exposto ao frontend diretamente.

### 13.2. Compatibilidade e evolução de contrato

- Esta é a primeira implementação. **Não há contrato legado**.
- Quebras futuras previsíveis:
  - Adição de MFA pode introduzir um estado intermediário `pending_mfa` entre login-com-credenciais e sessão ativa, alterando o contrato de `POST /login`.
  - Inclusão de Google OAuth pode adicionar um novo endpoint sem quebrar o existente.

---

## 14. Observabilidade e suporte

- **`traceId`**: o `ProblemDetails` retornado não inclui `traceId` nem `path` explicitamente — apenas `title`, `type`, `detail`, `status` e (quando aplicável) `errors`. Há um `TraceIdentifierFilter` para o event bus (`src/Web/ServiceInstallers/EventBus/PipeFilters/TraceIdentifierFilter.cs`) e `AddProblemDetails` em `src/Web/Extensions/ServiceCollectionExtensions.cs` que injeta `traceIdentifier` no body do ProblemDetails via middleware padrão do ASP.NET — ou seja, o campo aparece no response de erros produzidos pelo pipeline, mas **não** nos ProblemDetails montados manualmente pelo `HandleFailure`. Ver §16.
- **Logs do backend:**
  - `ForgotPasswordHandler` não loga sucesso/falha silenciosa — e-mail inexistente simplesmente retorna `Success`.
  - `ProjectionRefreshTokenEventHandler` e `ProjectionPasswordResetTokenEventHandler` logam `LogError` quando falham a projeção MongoDB.
  - Falhas de envio de e-mail (Brevo SMTP) são responsabilidade de `IEmailService` (camada Core) — logs e retries dependem dessa implementação, não desta feature.
- **Como o frontend pode ajudar o suporte:** ao reportar um erro, incluir o corpo do `ProblemDetails` (`type`, `detail`) e o timestamp. Se o backend expuser `traceId` (via middleware padrão), incluí-lo também.

---

## 15. Exemplos reais de payload

```json
// Request válido — Login
{
  "email": "victor.kunz@vertgroup.com.br",
  "password": "MinhaSenh@123"
}

// Response válida — Login (200 OK)
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI4M2U4Y2YxYi00ZjIxLTQ1OGUtOGRiMC1iNzkwZmEzY2Q1YTciLCJlbWFpbCI6InZpY3Rvci5rdW56QHZlcnRncm91cC5jb20uYnIiLCJyb2xlIjoiQWNjb3VudGFuY3lBZG1pbiIsInNlY3VyaXR5X3N0YW1wIjoiOTE1OTY2OGMtOTkyYy00ZGM1LWIyNTUtMzAyNGUwZmYxMjZkIiwiYWNjb3VudGFuY3lfaWQiOiJhNDZlNDRiOC0yY2RjLTQzOWQtODYwYy1kNWQ0YWQ1NzhmNDIiLCJleHAiOjE3Nzc2NTg5MDAuImlzcyI6InNvY2lldGl6YS1hcGkiLCJhdWQiOiJzb2NpZXRpemEtd2ViIn0.signature"
}
// + Set-Cookie: refresh_token=eyJhbGciOiJIUzI1NiIs...; HttpOnly; Secure; SameSite=Strict; Expires=...; Max-Age=604800

// Request válido — Refresh (corpo vazio, cookie transportado pelo browser)

// Response válida — Refresh (200 OK) — idêntica à de Login
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

// Request válido — Logout (corpo vazio, cookie + Authorization header)

// Response válida — Logout (200 OK) — corpo vazio

// Request válido — ForgotPassword
{
  "email": "victor.kunz@vertgroup.com.br"
}

// Response válida — ForgotPassword (200 OK) — corpo vazio

// Request válido — ResetPassword
{
  "token": "R9aZ7uV3yXqNbE2mK4pT6sLw8oYj1cH0dGfI5rPvA==",
  "newPassword": "NovaSenh@456",
  "confirmPassword": "NovaSenh@456"
}

// Response válida — ResetPassword (200 OK) — corpo vazio

// Erro — credenciais inválidas (400 Bad Request)
{
  "title": "Bad Request",
  "type": "Auth.InvalidCredentials",
  "detail": "Invalid email or password",
  "status": 400,
  "errors": null
}

// Erro — refresh token inválido/expirado (400 Bad Request)
{
  "title": "Bad Request",
  "type": "Auth.InvalidToken",
  "detail": "Refresh token is invalid or expired",
  "status": 400,
  "errors": null
}

// Erro — reset token inválido/expirado/usado (404 Not Found)
{
  "title": "Not Found",
  "type": "Auth.InvalidToken",
  "detail": "Password reset token is invalid or expired",
  "status": 404,
  "errors": null
}

// Erro — validação (400 Bad Request)
{
  "title": "Validation Error",
  "type": "Validation.Error",
  "detail": "One or more validation errors occurred",
  "status": 400,
  "errors": [
    { "code": "Validation.Email", "message": "Email must be valid" },
    { "code": "Validation.Password", "message": "Password must be at least 8 characters" }
  ]
}
```

Observações:
- O shape exato de `errors[]` depende de `IValidationResult.Errors` (Core) — assumido aqui como `Error[]` com `code` + `message` (o mesmo tipo `Error` usado em `HandleFailure`). O exemplo acima é ilustrativo; o frontend deve confirmar inspeccionando a resposta real do pipeline de validação.

---

## 16. Gaps, limitações e decisões abertas

### Divergências resolvidas

1. ~~**Nomes de role (`AccountingAdmin` vs `AccountancyAdmin`)**~~ — **RESOLVIDO.** O código foi atualizado: `UserRole.cs` agora define `AccountancyAdmin` e `AccountancyEmployee`. A claim `role` do JWT carrega esses valores. Frontend deve usar `AccountancyAdmin` e `AccountancyEmployee`.

2. ~~**Status HTTP para credenciais inválidas**~~ — **RESOLVIDO.** `LoginEndpoint` agora declara apenas `ProducesResponseType(StatusCodes.Status400BadRequest)` para falha de credenciais. O Swagger e o comportamento real do `HandleFailure` estão alinhados: `Auth.InvalidCredentials` retorna `400 Bad Request`.

3. ~~**Requisitos de complexidade de senha**~~ — **RESOLVIDO.** Tanto `RegisterUserValidator` quanto `ResetPasswordValidator` agora implementam as mesmas regras: mínimo 8 caracteres, máximo 100, ao menos uma letra maiúscula, ao menos uma letra minúscula, ao menos um dígito. O contrato documentado no Swagger (`ResetPasswordEndpoint.Description`) está coerente com o validator.

### Limitações conhecidas (dentro do escopo)

4. **Endpoint de perfil do usuário autenticado (`GET /me`).** A seção 13 assume a possibilidade de um endpoint que exponha o perfil pós-login. Esse endpoint **não existe** no código atual. Alternativas para o frontend hoje: (a) decodificar o JWT localmente (não validar, só ler claims para popular UI), ou (b) aguardar spec futura de gestão de usuário. Gap conhecido já registrado em `identity-context.md`. Endpoint de perfil faz parte da feature de gestão de usuários — não entra neste escopo.

5. **`traceId` em respostas de erro.** `HandleFailure` monta `ProblemDetails` sem popular `traceId` ou `path`. O middleware padrão do ASP.NET pode injetar `traceIdentifier` via `AddProblemDetails`, mas não é garantido para ProblemDetails construídos manualmente. O envelope canônico da documentação global (`{ statusCode, message, path, traceId, errors, detail }`) não é estritamente respeitado — o backend retorna o shape do `ProblemDetails` do ASP.NET (`title`, `type`, `detail`, `status`, `extensions.errors`). Frontend precisa se adaptar ao shape real.

6. **Múltiplos tokens de reset válidos coexistindo.** A spec aceita (§5 edge cases). O código confirma: `ForgotPasswordHandler` emite um novo `PasswordResetToken` a cada chamada **sem invalidar os anteriores**. Qualquer um pode ser consumido dentro do prazo de 1 hora. Aceito, documentado.

7. **Race condition em reset.** `ResetPasswordHandler` faz (a) find no projection, (b) load do aggregate, (c) checagem de validade, (d) uso do token. Não há lock otimista explícito no handler. Duas requisições simultâneas podem passar pela checagem (c) antes de qualquer `AppendEventsAsync` tomar efeito. O event store (`Core.Persistence`) pode ou não detectar concorrência por `Version` — não verificado neste documento. Potencial problema se observado em produção.

8. **Ausência de rate limit específico.** Nenhum middleware de rate limit foi encontrado na camada Web para `Login` ou `ForgotPassword`. A spec §6 confirma que não está no escopo desta versão. Aceito; pode virar problema de abuso.

9. **Envio de e-mail assíncrono sem feedback ao frontend.** `ForgotPasswordHandler` enfileira `SendPasswordResetEmailCommand` via Hangfire e retorna `Success` imediatamente. Falhas no envio (SMTP recusado, bounce) não propagam para o usuário — só aparecem em logs de servidor. Aceito pelo fluxo de confirmação genérica; suporte precisa monitorar Brevo.

10. **Password reset token hash colidindo.** Chave de busca na projeção é `TokenHash` (SHA-256 de um randomBytes de 32). Probabilidade de colisão é efetivamente zero, mas o schema permite. O índice em MongoDB não foi verificado neste documento.

11. **Conta recém-convidada que não finalizou cadastro.** Logic.md §5 prevê: login falha com `Auth.InvalidCredentials` porque a conta ainda não existe. Confirmado: `LoginHandler` faz find pela projeção `User.Email` — se o convite ainda não consumiu, não há linha em `User`. O fluxo funciona como esperado.

### Decisões abertas

- Expor ou não `GET /me` para facilitar o frontend (item 4) — entra em spec de gestão de usuários.
- Padronizar o envelope de erro para incluir `traceId` (item 5).

Esta documentação reflete o estado do código em `master` de `api.societiza` em 2026-04-20. Qualquer alteração na implementação requer atualização deste artefato.