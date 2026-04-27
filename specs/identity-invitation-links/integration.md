---
status: approved
spec: identity-invitation-links
domain: identity
created_at: 2026-04-23
approved_at: 2026-04-23
---

# Contrato de Integração Backend → Frontend — Identity Invitation Links

## 1. Identificação da feature

- **Nome da feature**: Links de Convite (Identity Invitation Links)
- **Módulo backend responsável**: `Identity` (write + read). Dependência de leitura cross-module em `Accountancy` via `IAccountancyDataService`.
- **Status atual**: `existente` — MVP implementado conforme plano técnico aprovado. Sem revogação manual (fora do escopo MVP).
- **Data da última atualização**: 2026-04-23
- **Responsável técnico**: a definir (tech-lead de backend)

O frontend está integrando um MVP funcional. Todos os endpoints previstos no plano técnico foram expostos. O estado terminal `Revoked` e o endpoint de revogação manual não existem nesta versão — ciclos de vida encerram apenas por `Expired` (tempo) ou `Exhausted` (usos).

---

## 2. Objetivo de negócio da feature

Links de Convite é a forma canônica de entrada por convite na Societiza. Substitui o fluxo anterior de disparo de e-mail individual: quem tem permissão (SystemAdmin, AccountancyAdmin) gera um link que carrega em si o perfil de acesso do futuro usuário e, quando aplicável, a contabilidade de destino. O admin compartilha o link pelo canal que preferir (WhatsApp, e-mail, mensagem direta) e os convidados entram no seu próprio ritmo.

Existe para eliminar o atrito do onboarding em lote: um AccountancyAdmin que precise trazer uma equipe inteira não depende mais de coletar e-mails de cada pessoa, digitá-los certos no sistema, acompanhar convites pendentes de 72 horas e reenviar quando expiram. O link é gerado uma vez, o sistema cuida da contagem de usos, do prazo de validade de 7 dias e do bloqueio automático do link quando o limite é atingido ou quando vence.

Atinge diretamente os papéis que já têm autoridade administrativa no sistema — SystemAdmin e AccountancyAdmin — e indiretamente os AccountancyEmployees, que passam a entrar de forma mais fluida. Entra na jornada em todo momento de **onboarding de equipe**: nascimento de uma nova contabilidade na plataforma, crescimento de equipe existente, substituição de pessoa, convite de novo admin dentro da mesma contabilidade.

No produto, ela se conecta ao Identity (autenticação, criação de conta, perfis de acesso) e ao Accountancy (contabilidade de destino em links SA→AA e links dentro do tenant). Também serve como base para futuras métricas de onboarding (cada usuário carrega referência do link pelo qual entrou) e para perfis mais granulares dentro da contabilidade.

---

## 3. Escopo atual versus escopo futuro

**O que já está implementado no backend hoje:**

- Criação de link com 4 combinações de perfil: SA→SA, SA→AA, AA→AA, AA→AE.
- Limite fixo de 1 uso para SA→SA, SA→AA e AA→AA. Limite configurável (≥1) apenas para AA→AE.
- Validade fixa de 7 dias contados da criação.
- Checagem soft de unicidade de link ativo por escopo (`TargetRole + AccountancyId`) antes da criação, com resposta 409 quando já existe.
- Listagem paginada (`PageNumber`/`PageSize`) com filtro padrão `OnlyActive = true` e ordenação fixa (ativo primeiro, depois `CreatedAt` desc).
- Detalhe por ID com a lista embutida de usuários que entraram por aquele link (`Users`).
- Registro público via token: valida link, cria conta, consome 1 uso do link.
- Transição automática para `Exhausted` no ato do último uso (dentro do aggregate).
- Job recorrente `expire-pending-invitation-links` (cron `0 * * * *`, execução horária) que transiciona para `Expired` os links vencidos ainda ativos.
- Isolamento multi-tenant: AccountancyAdmin só vê e cria links da própria contabilidade; SystemAdmin sem `AccountancyId` lista só SA→SA; com `AccountancyId` lista os links daquele tenant.
- Propagação de `InvitationLinkId` na projection de `User` e no evento `UserRegistrationCompleted`.

**O que ainda não está implementado:**

- Revogação manual (`InvitationLinkRevoked`, `RevokeInvitationLinkCommand`, endpoint de revoke). Não existe nesta versão. Estado `Revoked` não existe em `InvitationLinkStatus`.
- Endpoint público de validação de token antes do cadastro (o convidado só descobre se o link é válido ao submeter o formulário).
- Query / endpoint separado para listar usuários de um link (usuários vêm embutidos no `GetInvitationLinkById`).
- Índice parcial único em MongoDB para garantia atômica de unicidade por escopo. A garantia é feita só via checagem soft.

**O que existe no domínio mas não está exposto por endpoint:**

- Método `InvitationLink.Expire()` no aggregate é invocado apenas pelo job de expiração; não há endpoint de expiração manual.

**O que é apenas visão futura de produto:**

- Link dedicado (vinculado previamente a um e-mail específico).
- QR code / geração de links em lote.
- Mudança de perfil pós-entrada.
- Dashboards de onboarding.
- Integração com tiers de cobrança.
- Conta existente em contabilidade diferente recebendo convite de outra contabilidade (multi-contabilidade por usuário) — fora do escopo.

**O que ainda está em discussão:**

- Nenhum ponto em aberto identificado no código. Divergência explícita entre spec e código: a spec menciona "revogação" em várias decisões, o MVP não implementa. Documentado nos gaps (seção 16).

---

## 4. Contexto de produto e semântica da feature

O usuário percebe a feature como dois fluxos distintos. Do lado do admin: uma área onde ele gera um link, copia o token/URL e compartilha. O link aparece em uma lista default de "ativos" e transita silenciosamente para histórico quando esgota ou expira. Do lado do convidado: um link que ele recebe por fora do sistema e que o leva à tela de cadastro — sem precisar de e-mail pré-cadastrado, só completa o formulário.

Sucesso ao usar, para o admin, significa conseguir criar o link (rota responde 200) e ver o link aparecer na listagem default com status `Active`. Para o convidado, sucesso é concluir o `Register` com 200 e `IdentifierResponse`. Para o sistema, sucesso é a cadeia inteira: link criado (`InvitationLinkCreated` projetada) → convidado registrado (`UserRegistrationCompleted` projetada) → link incrementado (`InvitationLinkConsumed` projetada) → transição para `Exhausted` quando `Uses == MaxUses`.

Erro funcional relevante é qualquer bloqueio que impeça uma dessas transições: 409 no create significa que já existe link ativo no mesmo escopo; 404 no register significa que o token não corresponde a nenhum link; erro `InvitationLink.Expired`/`InvitationLink.Exhausted`/`InvitationLink.NotUsable` no register significa que o link foi encontrado mas não está usável (o código Http continua 400 — Bad Request —, mas o `error.code` distingue o motivo). Um 409 no register significa conta pré-existente com o mesmo e-mail.

Estados relevantes: o aggregate `InvitationLink` existe em três status (`Active`, `Exhausted`, `Expired`). Não há `Revoked` no MVP. A transição é unidirecional — nenhum estado terminal retorna a `Active`. O campo `Uses` cresce monotonicamente dentro de `MaxUses`, nunca decrementa.

---

## 5. Dependências e relações com outras features

- **Depende de:**
  - `Identity` — módulo de origem da feature. Reusa `UserRole` (SystemAdmin, AccountancyAdmin, AccountancyEmployee), autenticação, projection de `User`.
  - `Accountancy` — consultado via `IAccountancyDataService.ExistsAsync(Guid accountancyId, …)` quando `SystemAdmin` cria link SA→AA. Implementação em `Web/Services/CrossModule/AccountancyDataService.cs` delega ao handler `GetAccountancyDetailByIdQuery` do módulo Accountancy.
- **Consumida por:**
  - Qualquer feature futura que precise rastrear origem de cadastro — `ProjectionModel.User.InvitationLinkId` é a chave.
  - Dashboards / métricas de onboarding (futuro).
- **Centralidade no produto:** central no módulo Identity. É a nova forma canônica de entrada por convite.
- **Dependências de dados de outro módulo:** apenas leitura de existência de contabilidade, não há replicação de dados de Accountancy dentro do Identity.

---

## 6. Rotas públicas

| Método | Path | Tipo | Finalidade | Params de rota | Pronta para frontend? |
|---|---|---|---|---|---|
| POST | `/api/v1/identity/invitation-link` | command | Criar link de convite. Retorna `id`, `token`, `url` de cadastro e `expiresAt`. | — | sim |
| GET | `/api/v1/identity/invitation-link` | query | Listar links (paginada). Filtros por `onlyActive` (default true) e `accountancyId` (somente SA). | — | sim |
| GET | `/api/v1/identity/invitation-link/{invitationLinkId}` | query | Detalhe do link + usuários registrados por ele. | `invitationLinkId: guid` | sim |
| POST | `/api/v1/identity/invitation-link/register` | command | Cadastro público do convidado usando o token bruto. | — | sim |

Observações:

- Base path é `/api` via `UsePathBase("/api")`. Versionamento em `/v1/...` na rota.
- As três primeiras rotas exigem autenticação com policy `CanManageInvites` (SystemAdmin ou AccountancyAdmin). A quarta (`Register`) é pública — sem `[Authorize]`.
- Criar e Listar compartilham o mesmo path, distinguidos por verbo HTTP.

---

## 7. Contrato de request

### Request — CreateInvitationLink (`POST /api/v1/identity/invitation-link`)

Corpo (`application/json`):

- `targetRole`: string, obrigatório. Um de: `SystemAdmin`, `AccountancyAdmin`, `AccountancyEmployee`.
- `accountancyId`: `Guid?`, opcional. Regras dependem de `targetRole` e role do caller (ver abaixo).
- `maxUses`: `int?`, opcional. **Deve ser nulo** quando `targetRole` ∈ {`SystemAdmin`, `AccountancyAdmin`} — nesses casos o backend fixa em 1. **Deve ser ≥ 1** quando `targetRole == AccountancyEmployee`.

Regras efetivas de `accountancyId` (aplicadas no handler, não no validator):

- Se `targetRole == SystemAdmin`: `accountancyId` **precisa ser nulo**. Presença rejeitada com erro `InvitationLink.ValidationError`.
- Se caller é SystemAdmin e `targetRole == AccountancyAdmin`: `accountancyId` **é obrigatório** e deve existir no módulo Accountancy (verificado via `ExistsAsync`). Ausência → `InvitationLink.ValidationError`. Contabilidade inexistente → `Accountancy.NotFound` (404).
- Se caller é AccountancyAdmin: `accountancyId` enviado pelo frontend **é ignorado**. O backend força a contabilidade do usuário autenticado (`currentUser.AccountancyId`). Isolamento multi-tenant é inviolável.

Matriz de autorização (em cima da policy `CanManageInvites`, aplicada dentro do handler):

- SystemAdmin → pode criar: SA→SA, SA→AA.
- AccountancyAdmin → pode criar: AA→AA, AA→AE.
- Qualquer outra combinação → `InvitationLink.Forbidden` (403).

Transformações obrigatórias no frontend: nenhuma específica — o payload é primitivo.

Validações relevantes (FluentValidation em `CreateInvitationLinkValidator`):

- `targetRole` não vazio + valor válido em `UserRole`.
- `accountancyId`, se enviado: `!= Guid.Empty`.
- `maxUses` quando `targetRole == AccountancyEmployee`: `NotNull` + `>= 1`.
- `maxUses` quando `targetRole ∈ {SystemAdmin, AccountancyAdmin}`: deve ser `Null`.

### Request — ListInvitationLinks (`GET /api/v1/identity/invitation-link`)

Query string:

- `onlyActive`: `bool`, default `true`. Quando `true`, filtra apenas `Status == Active && ExpiresAt > now` (avaliado em tempo real, independente do último snapshot projetado).
- `accountancyId`: `Guid?`, opcional. **Ignorado** quando caller é AccountancyAdmin (backend força o próprio tenant). Relevante só para SystemAdmin: se ausente, retorna links SA→SA; se presente, retorna todos os links daquela contabilidade.
- `pageNumber`: `int`, default `1`. Validado pelo `PagedQueryValidator` (>= 1).
- `pageSize`: `int`, default `10`. Validado pelo `PagedQueryValidator` (limite documentado 1–100 na descrição do endpoint).

Validações relevantes: `PagedQueryValidator<ListInvitationLinksQuery>` — regras padrão de paginação.

### Request — GetInvitationLinkById (`GET /api/v1/identity/invitation-link/{invitationLinkId}`)

Rota:

- `invitationLinkId`: `Guid`, obrigatório. Constraint `:guid` na rota — URL malformada retorna 404 do próprio roteamento.

Autorização adicional no handler: SystemAdmin pode ver qualquer link; AccountancyAdmin só se `link.AccountancyId == currentUser.AccountancyId`.

### Request — RegisterUser (`POST /api/v1/identity/invitation-link/register`)

Corpo (`application/json`):

- `token`: string, obrigatório. É o **token bruto** gerado no create (`Base64(RandomNumberGenerator.GetBytes(32))`). O backend recalcula o SHA256 e procura pela projection usando `TokenHash`.
- `firstName`: string, obrigatório. `MaxLength 100`.
- `lastName`: string, obrigatório. `MaxLength 100`.
- `email`: string, obrigatório. `EmailAddress`, `MaxLength 150`.
- `password`: string, obrigatório. `MinLength 8`, `MaxLength 100`, regex exigindo letra maiúscula, letra minúscula e dígito.
- `confirmPassword`: string, obrigatório. Deve ser `Equal(password)`.

Transformações obrigatórias no frontend:

- Enviar o **token bruto** como chegou via URL — não aplicar URL-decode adicional se o Next.js já decodificou. O raw token vem em Base64 padrão (pode conter `+`, `/`, `=`) — confirmar se o link compartilhado é transportado intacto.
- Não aplicar máscara ao `email`.

Validações relevantes: `RegisterUserValidator`.

---

## 8. Contrato de response

### Response — CreateInvitationLink (`200`)

```
{
  "id": Guid — identificador do link criado,
  "token": string — token bruto (Base64, 32 bytes de aleatoriedade),
  "url": string — URL completa pronta para compartilhar, formato "{FrontendOptions.BaseUrl}/auth/join?token={token}",
  "expiresAt": DateTimeOffset — ISO 8601 com offset, 7 dias após criação
}
```

- `url` é construída pelo backend a partir de `FrontendOptions.BaseUrl` configurado no módulo Web. O frontend pode usar diretamente ou reconstruir a partir de `token`.
- `token` retornado é apenas o raw (não o hash). O hash (`TokenHash`) nunca sai da API.

### Response — ListInvitationLinks (`200`)

Envelope paginado `IPagedResult<InvitationLinkDetailResponse>`:

- `items`: array de `InvitationLinkDetailResponse`.
- Metadados de paginação (`page`, `pageSize`, `totalItems`, `hasNextPage`, `hasPreviousPage`) — formato do `PagedResult<T>` padrão do projeto.

Cada `InvitationLinkDetailResponse`:

- `id`: Guid.
- `targetRole`: string. Valores canônicos: `SystemAdmin`, `AccountancyAdmin`, `AccountancyEmployee`.
- `status`: string. Valores canônicos: `Active`, `Expired`, `Exhausted`. Nota: filtro `onlyActive` pode esconder `Expired`/`Exhausted`, mas eles aparecem quando `onlyActive=false`.
- `accountancyId`: `Guid?`. Null para links SA→SA; preenchido para SA→AA, AA→AA, AA→AE.
- `createdByUserId`: Guid.
- `maxUses`: int. Sempre 1 para SA→SA / SA→AA / AA→AA; valor informado pelo admin para AA→AE.
- `uses`: int. Contador atual.
- `createdAt`: DateTimeOffset (ISO 8601).
- `expiresAt`: DateTimeOffset (ISO 8601). Sempre `createdAt + 7 dias`.
- `updatedAt`: `DateTimeOffset?`. Null até haver primeiro consumo ou transição de status.
- `url`: string. Reconstruída a partir de `token` da projection: `"{BaseUrl}/auth/join?token={token}"`.

Ordenação: links ativos (`Status == Active && ExpiresAt > now`) primeiro, depois por `CreatedAt` decrescente. **Ordem é autoritativa** — frontend não deve reordenar localmente.

### Response — GetInvitationLinkById (`200`)

`InvitationLinkByIdResponse`:

- Todos os campos de `InvitationLinkDetailResponse` **exceto** a ordem difere ligeiramente — o record canônico do backend traz: `id`, `targetRole`, `status`, `accountancyId`, `createdByUserId`, `maxUses`, `uses`, `expiresAt`, `createdAt`, `updatedAt`, `url`.
- `users`: `List<InvitationLinkUserResponse>` — lista embutida de usuários registrados via este link, ordenada por `CreatedAt` asc (ordem de entrada).

Cada `InvitationLinkUserResponse`:

- `userId`: Guid.
- `firstName`: string.
- `lastName`: string.
- `email`: string.
- `joinedAt`: DateTimeOffset — corresponde a `User.CreatedAt`.

### Response — RegisterUser (`200`)

`IdentifierResponse`:

- `id`: Guid — id do usuário recém-criado.

Nenhum token de sessão é retornado. O frontend deve encaminhar o usuário ao fluxo de login após o cadastro.

### 8.1. Formato canônico versus formato de exibição

- `targetRole` e `status` chegam **sem tradução** (canônico: nomes internos em PascalCase). Se a UI for pt-BR, o frontend mapeia.
- Datas (`createdAt`, `updatedAt`, `expiresAt`, `joinedAt`) chegam em ISO 8601 com offset (`DateTimeOffset`) — formato canônico, pronto para parsing JS (`new Date(...)`).
- `token` chega **bruto** (Base64). Se o frontend for embutir em URL, pode precisar de URL-encode dependendo do transporte. A `url` retornada pelo backend já vem concatenada crua — confirmar que `FrontendOptions.BaseUrl` não aplica encoding.
- `email` chega como gravado pelo usuário (sem normalização aparente no código).
- `url` chega pronta para exibição/cópia.

### 8.2. Nullabilidade e ausência de campo

- `accountancyId`: null em links SA→SA. `null` semanticamente significa "sem contabilidade", não "não preenchido".
- `updatedAt`: null até o primeiro consumo ou transição de status. `null` significa "nunca atualizado desde a criação".
- Não há campos omitidos do JSON por padrão — todos saem sempre, mesmo quando null (comportamento default do System.Text.Json / Newtonsoft na solução; não identificado `NullValueHandling.Ignore` no código).

---

## 9. Estados de negócio e transições

Estados do aggregate `InvitationLink` (enum `InvitationLinkStatus` em `Identity.Domain.Enumerations`):

- `Active` (value 0) — estado inicial. O link é usável (condições: `Uses < MaxUses && ExpiresAt > UtcNow`).
- `Expired` (value 1) — terminal. Disparado pelo job horário ou rejeitado em tempo real pelo `IsUsable()` quando `ExpiresAt <= UtcNow`.
- `Exhausted` (value 3) — terminal. Disparado no mesmo `Consume()` que leva `Uses` a `MaxUses`.

Observação importante: `InvitationLinkStatus` tem value 0 (Active), 1 (Expired), **3** (Exhausted). O valor 2 não é utilizado — ficou reservado mas não há `Revoked` no MVP.

Transições possíveis:

- `Active → Exhausted` — no ato de `Consume()` dentro do aggregate, quando `Uses + 1 == MaxUses`. Dois eventos disparados em sequência: `InvitationLinkConsumed` e `InvitationLinkExhausted`.
- `Active → Expired` — via job `expire-pending-invitation-links`. O job carrega o aggregate, chama `Expire()`, persiste `InvitationLinkExpired`.
- Nenhum estado terminal sai. Sem reativação, sem reversão.

Validação em tempo real (`Register`): independentemente do status projetado, o handler recarrega o aggregate do event store e chama `IsUsable()`, que reavalia as três condições. Um link que venceu há 5 minutos mas ainda não foi visto pelo job é rejeitado no ato do registro.

O que bloqueia transições:

- Criação: violação de unicidade de escopo (`InvitationLink.AlreadyActiveInScope`, 409).
- Consumo: link em estado terminal → erro `InvitationLink.Expired` / `InvitationLink.Exhausted` / `InvitationLink.NotUsable` (400 via `HandleFailure`). E-mail duplicado → `User.AlreadyExists` (409).

CTAs / telas dependentes:

- Botão "criar novo link" para o escopo X deve ficar desabilitado quando já existe link ativo no escopo X — o frontend descobre via listagem default (`onlyActive=true`).
- Telas de histórico consomem `onlyActive=false`.
- Detalhe do link mostra a lista `users` — se for status terminal, é o histórico final; se ativo, vai crescendo até `MaxUses`.

### 9.1. Ordenação, filtros e agrupamentos com semântica de negócio

- **Ordenação da listagem é autoritativa**: `Status==Active && ExpiresAt>now` primeiro, depois `CreatedAt` desc. O frontend não deve reordenar.
- **Ordenação dos usuários** (dentro de `GetById`): `CreatedAt` ascendente. É a ordem real de entrada no sistema.
- **Filtro `onlyActive` é computado em tempo real**: usa `ExpiresAt > now`, não só `Status == Active`. Isso significa que links com `Status==Active` no projected mas já vencidos (job ainda não rodou) são filtrados fora da lista de ativos e **não** aparecem também em `onlyActive=false` a menos que o job já tenha transicionado. Até lá, ficam em um limbo onde não aparecem em nenhuma visão de listagem — documentado como gap na seção 16.
- **Filtros não suportados hoje**: não há filtro por `targetRole`, por `createdByUserId`, por intervalo de datas ou por status específico (`Expired` apenas, `Exhausted` apenas). Listagem é sempre "todos os visíveis + flag onlyActive".
- **Agrupamento por status (ativos vs histórico)**: o frontend deve fazer duas chamadas distintas (`onlyActive=true` e `onlyActive=false`) — não há endpoint unificado com agrupamento.

---

## 10. Erros esperados e edge cases

### Create

- **`400 Bad Request` (validator)**: payload inválido — `targetRole` fora de `UserRole`, `maxUses` inválido para o `targetRole`. Frontend deve apresentar inline no campo.
- **`400 Bad Request` (handler, `InvitationLink.ValidationError`)**: `accountancyId` presente em SA→SA ou ausente em SA→AA. Mensagem do backend é específica — frontend pode exibir.
- **`401 Unauthorized`**: token de acesso ausente/inválido. Frontend redireciona para login.
- **`403 Forbidden` (`InvitationLink.Forbidden`)**: caller autenticado mas sem combinação válida de role → targetRole. Mensagem do backend é genérica; o frontend pode mostrar "Você não tem permissão para criar esse tipo de link."
- **`404 Not Found` (`Accountancy.NotFound`)**: em SA→AA, a `accountancyId` não existe. Frontend deve informar "Contabilidade não encontrada" — provavelmente dado stale em lista.
- **`409 Conflict` (`InvitationLink.AlreadyActiveInScope`)**: já existe link ativo no mesmo escopo. **Bloqueio esperado** — frontend pode desabilitar o botão antes (consultando listagem) e, em caso de 409, mostrar mensagem explicativa orientando a aguardar expiração/esgotamento do link atual (revogação não existe no MVP).

### List / GetById

- **`400 Bad Request`**: paginação inválida (`pageNumber < 1`, `pageSize < 1`).
- **`401 Unauthorized`**: sem auth.
- **`403 Forbidden`**: role sem `CanManageInvites` (na prática, qualquer coisa que não seja SystemAdmin ou AccountancyAdmin). No GetById, também dispara se AccountancyAdmin tentar acessar link de outro tenant.
- **`404 Not Found` (GetById)**: id inexistente ou soft-deleted.

### Register (público)

- **`400 Bad Request` (validator)**: payload inválido — senha fraca, senhas não coincidem, e-mail malformado. Feedback inline.
- **`400 Bad Request` (handler)**: link encontrado mas não usável. Três códigos de error distintos — o frontend **deve** inspecionar o `error.code` para decidir mensagem:
  - `InvitationLink.Expired` → "Este link expirou. Peça um novo ao administrador."
  - `InvitationLink.Exhausted` → "Este link atingiu o número máximo de usos."
  - `InvitationLink.NotUsable` → fallback genérico.
- **`404 Not Found` (`InvitationLink.NotFound`)**: token não bateu com nenhum link (hash não encontrado). Frontend deve exibir "Link inválido" — sem revelar detalhe (pode ser token quebrado ou link nunca existente).
- **`409 Conflict` (`User.AlreadyExists`)**: conta com esse e-mail já existe. Frontend deve orientar "Já existe uma conta com este e-mail — faça login."

Envelope de erro padrão é `ProblemDetails` com:

- `title` — categoria (Validation Error, Not Found, Conflict, Forbidden, Bad Request).
- `type` — `error.code` (ex: `InvitationLink.NotFound`).
- `detail` — mensagem legível em inglês (em `InvitationLink.*` / `User.AlreadyExists`).
- `status` — código HTTP.
- `extensions.errors` — lista de `Error` quando for erro de validação.

---

## 11. Semântica de autenticação e autorização

- **Register é público**. Sem `[Authorize]`, sem policy.
- **Create / List / GetById** exigem auth + policy `CanManageInvites` (definida em `IdentityPolicies.Register`).
  - A policy aceita `UserRole.SystemAdmin` ou `UserRole.AccountancyAdmin`.
  - É implementada por um `HasRoleRequirement` + `HasRoleHandler` customizados no módulo Identity (não usa `RoleClaim` padrão).
- **Autorização adicional no handler** (regra de produto, além da policy):
  - Create aplica matriz de permissões (SA→{SA,AA}; AA→{AA,AE}). Fora disso, 403 `InvitationLink.Forbidden`.
  - GetById aplica regra de tenant (AA só do próprio tenant).
  - List aplica regra de visibilidade (AA vê só do próprio tenant; SA vê só SA→SA se não passar `accountancyId`).
- **Contexto de usuário** via `ICurrentUserService` (`UserId`, `Role`, `AccountancyId`). O `Role` é uma string — convertida no handler via `UserRole.TryFromName`. O `AccountancyId` é `Guid?` — pode ser null (SystemAdmin).

**Gap auth frontend ↔ backend**: o backend espera um JWT já contendo `role` e `accountancyId` (detalhes de emissão do token ficam fora desta feature). Confirmar com o time de auth se esses claims estão emitidos. O `Register` não emite token nem faz login automático — o frontend precisa redirecionar para a tela de login após sucesso.

---

## 12. Impacto de UX que o frontend precisa saber

- **Create retorna id novo** (`Guid`). O frontend pode seguir direto ao detalhe (`GetById`) ou apenas adicionar otimisticamente à lista.
- **Create retorna `url` pronta para compartilhar**. UX crítica é o botão de copiar — após `200`, o foco deve ir para "copiar link".
- **Transição para `Exhausted` é instantânea** (parte do mesmo commit do `Register`). O frontend precisa invalidar a listagem após sucesso de register ou, em cenários de admin em outra tela, aceitar que um poll periódico / refetch on focus é suficiente para pegar transições feitas por outras sessões.
- **Transição para `Expired` é por job horário**. Pode haver atraso de até ~1h entre o link vencer e o status virar `Expired` na projection. O filtro `onlyActive` compensa parcialmente (recalcula em tempo real), mas a visão histórica pode ficar vazia nessa janela. Documentado em gaps.
- **Ordem da listagem deve ser preservada**. Reordenação local quebra semântica.
- **A lista de `users` em `GetById` é embutida** — não há paginação própria. Para links com `MaxUses` grande, o payload pode crescer. MVP aceita.
- **Não há redirecionamento ou navegação automática em nenhum endpoint** — todos retornam payload, o frontend decide UX.
- **Máscaras**: nenhum campo precisa de máscara visual no que diz respeito à integração. Formatação de data fica a cargo do frontend.
- **Códigos de erro de Register precisam virar feedback no formulário**, não toast genérico. `InvitationLink.Expired`/`Exhausted`/`NotFound`/`NotUsable` são mensagens fundamentalmente diferentes para o convidado.

---

## 13. Impacto de dados no frontend

- Após **Create**: invalidar listagem (`ListInvitationLinks` do escopo correspondente) e invalidar caches de "existe link ativo no escopo X?".
- Após **Register** (admin consultando outra sessão): se houver subscrição em `GetById`, `uses` e possivelmente `status` mudaram. Refetch necessário. Listagem do admin também: contador atualizado.
- **Consistência eventualmente consistente** — backend usa event sourcing + projection MongoDB. O commit do evento é síncrono com MassTransit entregando ao handler de projection. Na prática o atraso é de ms, mas teoricamente a projection pode ficar atrás por janela curta após o commit.
- **UI otimista com segurança**: 
  - Create: aceitável otimizar com o id retornado (já gerado pelo backend).
  - Register: não aplicar otimismo — depende de múltiplas validações.
  - Revogação: não existe.
- **Stale data risks**:
  - Link `Active` na projection mas já vencido até o próximo job: a listagem com `onlyActive=true` filtra corretamente (tempo real); com `onlyActive=false` ele aparece como `Active` até o job rodar.
  - Contador de `Uses` atualiza por evento separado (`InvitationLinkConsumed`) — pode chegar em poucos ms após o Register, mas não é instantaneamente legível no mesmo commit.

### 13.1. Identificadores estáveis e chaves de reconciliação

- **Estáveis**: `id` do link (`Guid`, imutável), `userId` (imutável).
- **Pode mudar sem quebrar identidade**: `status`, `uses`, `updatedAt`.
- **Nunca usar como chave de reconciliação**: `token`, `url`, `expiresAt`, `createdAt` (imutáveis mas não são identidade), `email` (único, mas pode ser normalizado no futuro).
- O `token` é único por link ativo. Após a expiração/esgotamento, o token continua na projection mas não é mais usável — pode ser usado como chave visual ("qual link tem este token?"), não como identidade.

### 13.2. Compatibilidade e evolução de contrato

- Esta feature **substitui** o fluxo anterior de "convite por e-mail" (se existia endpoint legado, foi removido — não identificado no código como endpoint ativo).
- Contratos novos não coexistem com antigos.
- Evoluções futuras prováveis que exigirão atenção do frontend:
  - Adição de `Revoked` em `InvitationLinkStatus`.
  - Adição de campo `revokedAt` / `revokedBy` nos responses.
  - Possível migração de `token` no response do create para esquema one-time retrieval (token não volta em listagens).
  - Filtros adicionais (`targetRole`, `dateRange`).

---

## 14. Observabilidade e suporte

- **`traceId` no response de erro**: não identificado no `CreateProblemDetails` do `EndpointBaseExtensions` — o `ProblemDetails` criado não inclui `traceId` explicitamente. O template canônico menciona `traceId` no envelope global, mas a implementação atual do `HandleFailure` não seta. Confirmar se middleware global adiciona (presente `TraceIdentifierFilter` no EventBus, mas não no pipeline HTTP). Documentado como gap.
- **Logs relevantes**:
  - Projection handlers logam `LogError`/`LogWarning` em falhas (ver `ProjectionInvitationLinkEventHandler`).
  - Job `ExpireInvitationLinksHandler` loga `LogWarning` quando falha ao carregar aggregate específico.
  - Criação / consumo / expiração geram events no event store (auditoria natural via `StoreEvents`).
- **Diagnóstico de erro útil**: o frontend deveria registrar `error.code`, `error.detail`, HTTP status, e correlacionar por timestamp + userId. `token` **nunca** deve ir para log de cliente.

---

## 15. Exemplos reais de payload

```json
// Request válido — CreateInvitationLink (AA → AE)
{
  "targetRole": "AccountancyEmployee",
  "maxUses": 5
}
```

```json
// Request válido — CreateInvitationLink (SA → AA)
{
  "targetRole": "AccountancyAdmin",
  "accountancyId": "9b3c9f7a-2b6d-4a1e-8f2c-3d1a2b4c5d6e"
}
```

```json
// Response válida — CreateInvitationLink
{
  "id": "0f8f5c3a-4b1d-4e2a-9f7a-2b6c3d4e5f6a",
  "token": "k9Z1p8X2qR3tV4wY5aB6cD7eF8gH0iJ1kL2mN3oP4qR=",
  "url": "https://app.societiza.com.br/auth/join?token=k9Z1p8X2qR3tV4wY5aB6cD7eF8gH0iJ1kL2mN3oP4qR=",
  "expiresAt": "2026-04-30T12:34:56.789+00:00"
}
```

```json
// Request válido — RegisterUser
{
  "token": "k9Z1p8X2qR3tV4wY5aB6cD7eF8gH0iJ1kL2mN3oP4qR=",
  "firstName": "Joana",
  "lastName": "Silva",
  "email": "joana.silva@cliente.com.br",
  "password": "MinhaSenha123",
  "confirmPassword": "MinhaSenha123"
}
```

```json
// Response válida — RegisterUser
{
  "id": "7a3b1c2d-8e4f-4a5b-9c0d-1e2f3a4b5c6d"
}
```

```json
// Response válida — GetInvitationLinkById
{
  "id": "0f8f5c3a-4b1d-4e2a-9f7a-2b6c3d4e5f6a",
  "targetRole": "AccountancyEmployee",
  "status": "Active",
  "accountancyId": "9b3c9f7a-2b6d-4a1e-8f2c-3d1a2b4c5d6e",
  "createdByUserId": "1a2b3c4d-5e6f-7890-abcd-ef1234567890",
  "maxUses": 5,
  "uses": 2,
  "expiresAt": "2026-04-30T12:34:56.789+00:00",
  "createdAt": "2026-04-23T12:34:56.789+00:00",
  "updatedAt": "2026-04-23T15:01:00.000+00:00",
  "url": "https://app.societiza.com.br/auth/join?token=k9Z1p8X2qR3tV4wY5aB6cD7eF8gH0iJ1kL2mN3oP4qR=",
  "users": [
    {
      "userId": "7a3b1c2d-8e4f-4a5b-9c0d-1e2f3a4b5c6d",
      "firstName": "Joana",
      "lastName": "Silva",
      "email": "joana.silva@cliente.com.br",
      "joinedAt": "2026-04-23T14:50:00.000+00:00"
    },
    {
      "userId": "8b4c2d3e-9f5a-4b6c-ad1e-2f3a4b5c6d7e",
      "firstName": "Pedro",
      "lastName": "Souza",
      "email": "pedro.souza@cliente.com.br",
      "joinedAt": "2026-04-23T15:01:00.000+00:00"
    }
  ]
}
```

```json
// Erro de validação (exemplo: targetRole inválido)
{
  "title": "Validation Error",
  "type": "Validation.General",
  "detail": "One or more validation failures have occurred.",
  "status": 400,
  "errors": [
    { "code": "TargetRole", "message": "TargetRole must be one of: SystemAdmin, AccountancyAdmin, AccountancyEmployee" }
  ]
}
```

```json
// Conflito — já existe link ativo no mesmo escopo
{
  "title": "Conflict",
  "type": "InvitationLink.AlreadyActiveInScope",
  "detail": "An active link already exists for this scope — revoke the current one before creating another.",
  "status": 409
}
```

```json
// Link não usável no Register
{
  "title": "Bad Request",
  "type": "InvitationLink.Expired",
  "detail": "This invitation link has expired.",
  "status": 400
}
```

> **Nota sobre os exemplos**: os valores de `token`, `Guid`, e-mails e URL são ilustrativos — o contrato real do backend respeita os formatos descritos nas seções 7 e 8. A mensagem de `AlreadyActiveInScope` menciona "revoke" no texto porque está hard-coded no handler, mas **revogação não existe no MVP** — o admin precisa aguardar transição natural (ver gaps).

---

## 16. Gaps, limitações e decisões abertas

1. **Revogação manual não existe no MVP** — a spec prevê revogação em várias decisões e a mensagem do erro `AlreadyActiveInScope` menciona "revoke", mas o backend **não** implementa `InvitationLinkRevoked`, nem comando/endpoint de revogação, nem status `Revoked`. O admin cuja unidade de escopo já tem link ativo só pode aguardar expiração (7 dias) ou esgotamento para criar outro. **Divergência spec × código assumida no plan.md.**

2. **Mensagem do backend orienta ação inexistente** — o `detail` de `InvitationLink.AlreadyActiveInScope` diz "revoke the current one before creating another". Como revogar não é possível, o frontend **não deve repetir literalmente a mensagem** — traduza para "Aguarde o link atual esgotar ou expirar antes de criar outro."

3. **`Revoked` reservado em valor — value 2 do SmartEnum não é usado, `Exhausted` recebe value 3**. Documentação informativa; o frontend consome apenas os `Name` via JSON.

4. **Race condition na unicidade de escopo** — checagem soft via projection. Dois creates simultâneos podem gerar dois links ativos no mesmo escopo. Risco aceito no MVP (volume baixo).

5. **Dupla transação no Register** — `AppendEventsAsync(user)` + `AppendEventsAsync(link)` rodam em transações separadas. Se a segunda falhar, o usuário foi criado mas o contador do link não incrementou. O efeito para o convidado: cadastro aparente de sucesso. Para o admin: link com `uses` desatualizado. **Nenhum retry automático.** Risco aceito no MVP.

6. **Projection `Active` em limbo quando prazo venceu mas job ainda não rodou** — `onlyActive=true` filtra fora (condição em tempo real). `onlyActive=false` mostra como `Active` (condição só por status). Ou seja, o link "some" da UI até o job atualizar (até 1h). Frontend pode compensar aplicando filtro adicional `expiresAt > now` ao exibir a visão histórica — mas essa não é responsabilidade documentada pelo produto.

7. **`Token` armazenado na projection** — aceita como decisão MVP para permitir reconstrução de URL sem roundtrip ao event store. Em produção, pode-se mover para esquema one-time retrieval (token volta só na criação). Frontend deve considerar `token` dado sensível e **não logar**.

8. **`traceId` no envelope de erro não identificado no código** — `EndpointBaseExtensions.CreateProblemDetails` não adiciona `traceId`. O template canônico sugere esse campo. Se existir middleware global que injete, não foi varrido. Frontend deve consultar time de backend antes de tentar exibir `traceId`.

9. **Não há endpoint público de validação de token antes do cadastro** — o convidado só descobre se o link está válido submetendo o formulário. UX pode sofrer em links já esgotados/expirados (usuário preenche a tela inteira antes do erro). Frontend pode mitigar orientando o admin a compartilhar rapidamente, mas a solução real é endpoint de validação — fora do escopo MVP.

10. **Ausência de `User.AlreadyExists` cross-tenant clarification** — o Register rejeita e-mail duplicado **global** (projection de User). Se o usuário já pertence a outra contabilidade, o sistema não cria conta, nem associa. Mensagem retornada é genérica (`409 User.AlreadyExists`) — o frontend não distingue "é você mesmo" vs. "é outra pessoa com mesmo e-mail". Decisão de produto descrita na spec; mensagem simples no frontend.

11. **Validator `ListInvitationLinks` delega 100% ao `PagedQueryValidator`** — não há validação de combinação `callerRole x accountancyId` no validator; essa lógica vive dentro do handler. Frontend precisa tratar 403 de listagem quando envia combinação inválida.

12. **Sem índice MongoDB de unicidade por escopo** — plan.md confirma. Consequência: item 4 acima.

13. **Sem normalização de `email`** — o código grava `request.Email` sem `ToLower()` / `Trim()`. Dois cadastros com `Joana@x.com` e `joana@x.com` podem coexistir (a projection lookup é case-sensitive em MongoDB a menos que haja collation). Frontend deve normalizar antes de enviar para evitar variações acidentais.