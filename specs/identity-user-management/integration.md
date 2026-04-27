# Contrato de Integração — Gestão de Usuários

## 1. Identificação da feature

- **Nome da feature**: Gestão de Usuários (identity-user-management)
- **Módulo backend responsável**: Identity (write + read)
- **Status atual**: `existente`
- **Data da última atualização**: 2026-04-27
- **Responsável técnico**: não identificado no código — confirmar com o tech lead do Identity

A feature está totalmente implementada na branch `master` de `api.societiza`.
Estende o aggregate `User` existente com edição do próprio nome, promoção
`AccountancyEmployee → AccountancyAdmin` e soft-delete com invalidação imediata
de sessão. Adiciona endpoints de leitura para perfil próprio, get-by-id,
listagem de SystemAdmins e listagem de membros de uma contabilidade.

---

## 2. Objetivo de negócio da feature

Esta feature cobre o ciclo de vida do usuário **depois da entrada na
plataforma**. Os links de convite cobrem a entrada; aqui começa a vida do
membro depois que ele já tem conta. Ela existe para resolver quatro dores
operacionais: (i) **promoção paralisada** — sem caminho para promover um
`AccountancyEmployee` que assumiu responsabilidades administrativas, a
contabilidade depende eternamente da ordem em que cada um entrou; (ii)
**saída sem caminho** — usuários que deixaram a contabilidade não tinham
mecanismo de remoção, criando dores de segurança e compliance; (iii)
**operação sem lista** — admins não conseguiam responder "quem está na
equipe agora?"; (iv) **autogestão travada** — usuários não podiam corrigir o
próprio nome sem pedir ajuda fora da plataforma.

A feature beneficia diretamente o `AccountancyAdmin` (maior consumidor — opera
a feature todo dia), o `SystemAdmin` (visibilidade da plataforma e poder de
intervenção em emergências) e qualquer usuário (autogestão de nome). Entra na
jornada **logo após o primeiro login** e segue ativa até o desligamento. É o
backbone para toda colaboração futura: atribuição de tarefa, autoria
rastreável em processos, dashboard por responsável, menção em comentário e
permissões granulares dependem dessa lista de membros estável.

Conexão com o produto: consome `Identity — Autenticação` (sessão válida,
invalidação de sessão), começa onde `Identity — Links de convite` termina e
respeita a fronteira multi-tenant definida em `Accountancy`.

---

## 3. Escopo atual versus escopo futuro

**O que já está implementado no backend hoje:**

- `GET v{version}/identity/users` — perfil próprio do usuário autenticado
  (`GetMyProfileEndpoint` + `GetMyProfileHandler`).
- `GET v{version}/identity/users/{userId}` — get user por id, com matriz de
  acesso por perfil (`GetUserByIdEndpoint` + `GetUserByIdHandler`).
- `PUT v{version}/identity/users` — edição do próprio nome
  (`UpdateUserEndpoint` → `UpdateUserHandler` → `User.Update`).
- `POST v{version}/identity/accountancies/{accountancyId}/members/{userId}/promote`
  — promoção `AccountancyEmployee → AccountancyAdmin`
  (`PromoteUserEndpoint` → `PromoteUserHandler` → `User.Promote`).
- `DELETE v{version}/identity/users/{userId}` — soft-delete com invalidação
  imediata de sessão (`DeleteUserEndpoint` → `DeleteUserHandler` →
  `User.Delete`).
- `GET v{version}/identity/system-admins` — listagem paginada de SystemAdmins
  (`ListSystemAdminsEndpoint`).
- `GET v{version}/identity/accountancies/{accountancyId}/members` — listagem
  paginada de membros de uma contabilidade, com payload diferente por perfil
  (`ListAccountancyMembersEndpoint`).
- 3 eventos de domínio novos: `UserUpdated`, `UserPromoted`, `UserDeleted`,
  com filas MassTransit configuradas em `ConsumerConfiguration` e o handler
  `ProjectionUserEventHandler` atualizado.
- Soft-delete: usa flag `IsDeleted` herdada da base `Entity`/`IProjectionModel`
  (sem schema novo).
- Invalidação de sessão: `User.Delete` chama `RevokeAllRefreshTokens` +
  `RegenerateSecurityStamp` na mesma operação. O middleware JWT
  (`OnTokenValidated` em `Program.cs`) compara o `security_stamp` do token
  contra o `SecurityStamp` atual do aggregate em **toda request autenticada**
  e falha se diferente.

**O que ainda não está implementado:**

- Notificação por e-mail de promoção e remoção (decisão de spec — depende do
  módulo de notificações).
- Trilha de auditoria visível na interface (decisão de spec — fora do MVP).
- Reativação de usuário removido (decisão de spec — remoção é terminal).
- Demoção `AccountancyAdmin → AccountancyEmployee` (decisão de spec —
  promoção é unidirecional).
- Edição do próprio e-mail (decisão de spec — fora do MVP).
- Edição de dados pessoais de outro usuário (decisão de spec — proibida em
  qualquer cenário).

**O que existe no domínio mas não está exposto por endpoint:**

- A flag `IsDeleted` na projection `ProjectionModel.User` permite distinguir
  ativos e inativos. Os endpoints de listagem expõem isso via query param
  `OnlyActive` (default `true`). Não há endpoint dedicado a uma "lista de
  deletados" — divergência intencional da spec, ver §16.

**O que é apenas visão futura de produto:**

- Atribuição de tarefa, autoria rastreável em processos, dashboard por
  responsável, menção em comentário, permissões granulares e portal do
  cliente — todos dependem desta feature mas não são parte dela.

**O que ainda está em discussão:**

- Nada identificado no código. Todas as decisões da spec §8 estão
  refletidas. Gaps explícitos da implementação estão listados em §16.

---

## 4. Contexto de produto e semântica da feature

O usuário percebe esta feature em três momentos:

1. **Menu de perfil** — qualquer usuário abre o próprio perfil
   (`GET /identity/users`), edita o nome (`PUT /identity/users`) e salva. Sem
   confirmação adicional. A ação é reversível pelo próprio usuário.
2. **Área de Equipe da contabilidade** — `AccountancyAdmin` e
   `AccountancyEmployee` veem a lista de membros
   (`GET /identity/accountancies/{accountancyId}/members`). O admin tem CTAs
   de "Promover a administrador" e "Remover acesso" em cada linha. O
   employee vê apenas nome e perfil.
3. **Painel administrativo (SystemAdmin)** — `GET /identity/system-admins`
   lista os SystemAdmins da plataforma. Ao entrar no contexto de uma
   contabilidade específica, o SystemAdmin acessa
   `GET /identity/accountancies/{accountancyId}/members` com dados completos.

**Sucesso ao usar:**

- Edição de nome retorna 200; o frontend deve atualizar imediatamente o nome
  exibido no menu/cabeçalho/listagens.
- Promoção retorna 200 com `UserDetailResponse` contendo o novo `Role`. O
  frontend deve refletir o perfil atualizado sem aguardar logout.
- Remoção retorna 200; o frontend que disparou a ação atualiza a lista. O
  usuário removido tem o `SecurityStamp` regenerado e os refresh tokens
  revogados — qualquer request seguinte com o token antigo retorna 401, e o
  frontend dele deve tratar como sessão expirada.

**Erro funcional (não confundir com 4xx genérico):**

- Tentar promover quem já é `AccountancyAdmin` → 409 `User.AlreadyAdmin`.
- Tentar remover o último admin de uma contabilidade → 409
  `User.LastAccountancyAdmin`.
- Tentar remover o último SystemAdmin → 409 `User.LastSystemAdmin`.
- Tentar remover a si mesmo → 409 `User.CannotRemoveSelf`.
- `SystemAdmin` tentar promover qualquer usuário → 403
  `User.ForbiddenForSystemAdmin`.
- `AccountancyAdmin` agir fora da própria contabilidade → 403
  `User.ForbiddenTenant`.

**Estados/transições que importam:**

- Membro: `Ativo ↔ Inativo` (soft-delete; transição reversa não existe).
- Perfil: `AccountancyEmployee → AccountancyAdmin` (unidirecional; sem
  demoção).
- Nome: `Atual → Atualizado` (livre, autosserviço, sem confirmação).

---

## 5. Dependências e relações com outras features

**De quais outras features esta feature depende:**

- **Identity — Autenticação** — toda rota exige `[Authorize]` (JWT bearer). A
  invalidação de sessão depende de `RevokeAllRefreshTokens` e
  `RegenerateSecurityStamp` (mecanismo já existente). O middleware de auth
  (`OnTokenValidated`) compara `security_stamp` do JWT com o `SecurityStamp`
  do aggregate em toda request.
- **Identity — Links de convite** — herda o modelo de perfis (`SystemAdmin`,
  `AccountancyAdmin`, `AccountancyEmployee`) e a postura de preservação de
  histórico. A projection `User` carrega `InvitationLinkId` opcional para
  registros criados via convite — campo retornado nas listagens admin.
- **Accountancy** — fronteira multi-tenant via `accountancyId`. Toda ação de
  `AccountancyAdmin` é restrita ao próprio `accountancyId`; `SystemAdmin`
  passa o `accountancyId` explícito quando age dentro de uma contabilidade.

**Quais features consomem esta feature:**

- Hoje, nenhuma. É feature core do Identity.
- Futuras: atribuição de tarefa, autoria rastreável, dashboard por
  responsável, menção em comentário, portal do cliente — todas pressupõem
  uma lista de membros estável e a presença gerenciável.

**Posição no produto**: feature **central do módulo Identity** (P1).
Complementa o ciclo iniciado pelos links de convite.

**Dependências de dados de outro módulo**: `accountancyId` é uma `Guid` —
não há cross-call para o módulo Accountancy nesta feature; o backend confia
no claim `accountancy_id` do JWT para resolver o tenant do solicitante.

---

## 6. Rotas públicas

Base path real: `/api/v{version}/identity/...`. Todas em `ApiVersion("1.0")`.

| Método | Path | Tipo | Finalidade | Params de rota | Pronta para frontend? |
|---|---|---|---|---|---|
| GET | `/api/v1/identity/users` | Query | Perfil do próprio usuário autenticado | nenhum | sim |
| GET | `/api/v1/identity/users/{userId}` | Query | Get user por id (com matriz de acesso por perfil) | `userId: Guid` | sim |
| PUT | `/api/v1/identity/users` | Command | Atualizar nome do próprio usuário | nenhum | sim |
| DELETE | `/api/v1/identity/users/{userId}` | Command | Soft-delete de usuário com invalidação imediata de sessão | `userId: Guid`; query `accountancyId?: Guid` | sim |
| POST | `/api/v1/identity/accountancies/{accountancyId}/members/{userId}/promote` | Command | Promover `AccountancyEmployee` a `AccountancyAdmin` | `accountancyId: Guid`, `userId: Guid` | sim |
| GET | `/api/v1/identity/system-admins` | Query (paginada) | Listar SystemAdmins da plataforma | nenhum; query `OnlyActive`, `PageNumber`, `PageSize` | sim |
| GET | `/api/v1/identity/accountancies/{accountancyId}/members` | Query (paginada) | Listar membros de uma contabilidade (payload por perfil) | `accountancyId: Guid`; query `OnlyActive`, `PageNumber`, `PageSize` | sim |

**Notas:**

- `GET /users` (sem id) é especificamente o "meu perfil"; não recebe id e
  não substitui o `GET /users/{userId}`.
- `DELETE` aceita um `accountancyId` por query string, mas o backend só
  considera esse valor quando o solicitante é `SystemAdmin`. Para
  `AccountancyAdmin`, o tenant do claim sempre prevalece (o query é ignorado
  na decisão; ver §11).

---

## 7. Contrato de request

### Request — GetMyProfile

`GET /api/v1/identity/users`

Sem body, sem path params, sem query params. Apenas header `Authorization:
Bearer {token}`.

### Request — GetUserById

`GET /api/v1/identity/users/{userId}`

- `userId`: `Guid`, path, obrigatório, identificador do usuário alvo.

Validação relevante:

- `userId` não pode ser `Guid.Empty` (validator `GetUserByIdValidator`).

### Request — UpdateUser

`PUT /api/v1/identity/users`

Corpo (JSON):

- `firstName`: `string`, obrigatório, até 80 caracteres.
- `lastName`: `string`, obrigatório, até 80 caracteres.

Transformações obrigatórias no frontend: nenhuma — strings vão como digitadas.

Validações relevantes (`UpdateUserValidator`):

- `FirstName` não vazio; `MaximumLength(80)`.
- `LastName` não vazio; `MaximumLength(80)`.

### Request — PromoteUser

`POST /api/v1/identity/accountancies/{accountancyId}/members/{userId}/promote`

- `accountancyId`: `Guid`, path, obrigatório.
- `userId`: `Guid`, path, obrigatório.
- Sem body.

Validações relevantes (`PromoteUserValidator`):

- `AccountancyId` não vazio.
- `UserId` não vazio.

### Request — DeleteUser

`DELETE /api/v1/identity/users/{userId}?accountancyId=...`

- `userId`: `Guid`, path, obrigatório.
- `accountancyId`: `Guid?`, query, opcional. Usado **apenas** quando o
  solicitante é `SystemAdmin`; ignorado quando o solicitante é
  `AccountancyAdmin` (que usa o claim).

Validações relevantes (`DeleteUserValidator`):

- `UserId` não vazio.

### Request — ListSystemAdmins

`GET /api/v1/identity/system-admins?OnlyActive={bool}&PageNumber={int}&PageSize={int}`

- `OnlyActive`: `bool`, query, default `true`. Quando `true`, retorna apenas
  usuários ativos (`IsDeleted == false`).
- `PageNumber`: `int`, query, default `1`. Deve ser `>= 1`.
- `PageSize`: `int`, query, default `10`. Deve estar entre `1` e `100`
  (validador `PagedQueryValidator`).

### Request — ListAccountancyMembers

`GET /api/v1/identity/accountancies/{accountancyId}/members?OnlyActive=...&PageNumber=...&PageSize=...`

- `accountancyId`: `Guid`, path, obrigatório.
- `OnlyActive`: `bool`, query, default `true`.
- `PageNumber`: `int`, query, default `1`.
- `PageSize`: `int`, query, default `10`. Mesma faixa do `ListSystemAdmins`.

Validações relevantes (`ListAccountancyMembersValidator`):

- `AccountancyId` não vazio.
- Paginação validada via `PagedQueryValidator`.

---

## 8. Contrato de response

### Response — GetMyProfile

`UserProfileResponse`:

- `id`: `Guid` — identificador estável do usuário.
- `firstName`: `string` — nome próprio.
- `lastName`: `string` — sobrenome.
- `email`: `string` — e-mail (não editável neste MVP).
- `role`: `string` — `"SystemAdmin"`, `"AccountancyAdmin"` ou
  `"AccountancyEmployee"` (nome do `SmartEnum UserRole`).
- `accountancyId`: `Guid?` — `null` para `SystemAdmin`, preenchido para os
  demais.
- `createdAt`: `DateTimeOffset` (ISO 8601) — quando o usuário foi criado.
- `updatedAt`: `DateTimeOffset?` — última alteração no aggregate; `null`
  enquanto não houver edição.

### Response — GetUserById

Mesmo `UserProfileResponse` da rota `/users`. Campos idênticos. Sucesso 200;
404 se o alvo não existir ou estiver deletado; 403 se o solicitante não
tiver acesso conforme matriz de §11.

### Response — UpdateUser

Sem body. `200 OK` na sucesso. O frontend que disparou a ação deve refletir
o novo nome localmente (não há response body para reconciliar).

### Response — PromoteUser

`UserDetailResponse`:

- `id`: `Guid` — identificador estável.
- `firstName`: `string`.
- `lastName`: `string`.
- `email`: `string`.
- `role`: `string` — após a promoção, sempre `"AccountancyAdmin"`.
- `joinedAt`: `DateTimeOffset` — mapeia `CreatedAt` do aggregate (data em
  que o usuário foi criado).
- `invitationLinkId`: `Guid?` — id do convite que originou o usuário, lido
  da projection (`null` se não houver registro de convite).

### Response — DeleteUser

Sem body. `200 OK` na sucesso. Após esta resposta, qualquer request
subsequente do usuário removido com o token antigo recebe 401 (sessão
invalidada — ver §11).

### Response — ListSystemAdmins

`IPagedResult<object>` — o `object` é sempre `UserDetailResponse`:

- `items`: `UserDetailResponse[]` — até `pageSize` registros, ordenados por
  `createdAt` decrescente.
  - Cada item tem os mesmos campos de `UserDetailResponse` acima.
- `page`: objeto de paginação (`Page`):
  - `pageNumber`: `int`.
  - `pageSize`: `int`.
  - `hasNextPage`: `bool`.
  - `hasPreviousPage`: `bool`.
  - `totalCount`: `int`.
  - `totalPages`: `int`.

### Response — ListAccountancyMembers

`IPagedResult<object>` — heterogêneo conforme o perfil do solicitante:

- Solicitante `AccountancyAdmin` ou `SystemAdmin`: `items` é
  `UserDetailResponse[]` (com `email`, `joinedAt`, `invitationLinkId`).
- Solicitante `AccountancyEmployee`: `items` é `UserSummaryResponse[]`:
  - `id`: `Guid`.
  - `firstName`: `string`.
  - `lastName`: `string`.
  - `role`: `string`.
- Ordenação: `Role` ascendente (admins primeiro, conforme valores do
  `SmartEnum UserRole` — SystemAdmin=0, AccountancyAdmin=1,
  AccountancyEmployee=2) e depois `createdAt` ascendente.
- O envelope `page` é idêntico ao de `ListSystemAdmins`.

**Observação importante:** o tipo do `items` em
`ListAccountancyMembers` é runtime-polimórfico (`object[]`). O frontend
deve discriminar pelos campos presentes no objeto (a presença de `email`
indica `UserDetailResponse`; a ausência indica `UserSummaryResponse`),
ou pelo perfil do solicitante (já conhecido na sessão).

### 8.1. Formato canônico versus formato de exibição

- `email` chega no formato canônico (lower-case por convenção do registro,
  sem máscara). Exibir como vem.
- `createdAt`, `updatedAt`, `joinedAt`: `DateTimeOffset` ISO 8601 com offset
  (ex.: `2026-04-27T14:30:00.0000000+00:00`). Frontend formata localmente.
- `id`, `accountancyId`, `invitationLinkId`, `userId`: `Guid` em formato
  hífen padrão (`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`).
- `role`: nome do enum em PascalCase (`SystemAdmin`, `AccountancyAdmin`,
  `AccountancyEmployee`). Não há tradução server-side.

### 8.2. Nullabilidade e ausência de campo

- `accountancyId` (em `UserProfileResponse`): `null` para `SystemAdmin`;
  preenchido para `AccountancyAdmin` e `AccountancyEmployee`.
- `updatedAt`: `null` enquanto não houver edição no aggregate (estado
  inicial após criação).
- `invitationLinkId` (em `UserDetailResponse`): `null` quando o registro do
  usuário não tem convite associado na projection. Pode ocorrer em
  registros antigos ou em usuários criados fora do fluxo de convite.
- O backend **sempre** retorna o campo (não omite); usa `null` para
  ausência.

---

## 9. Estados de negócio e transições

### Estado de presença do usuário

| Estado   | Descrição |
|----------|-----------|
| `Ativo`  | `IsDeleted == false`. Aparece em listagens com `OnlyActive=true`. |
| `Inativo`| `IsDeleted == true`. Sessão invalidada, refresh tokens revogados. Aparece nas listagens apenas quando `OnlyActive=false`. |

**Transição `Ativo → Inativo`** disparada por `DELETE /users/{userId}`:

- Bloqueada para auto-remoção (409 `User.CannotRemoveSelf`).
- Bloqueada quando o alvo é o último `AccountancyAdmin` ativo da
  contabilidade (409 `User.LastAccountancyAdmin`). A contagem é feita na
  projection com filtro `Role == AccountancyAdmin && AccountancyId == ... && !IsDeleted`.
- Bloqueada quando o alvo é o último `SystemAdmin` ativo (409
  `User.LastSystemAdmin`).
- Bloqueada quando `AccountancyAdmin` tenta agir fora do próprio tenant
  (403 `User.ForbiddenTenant`).
- Bloqueada para `AccountancyEmployee` em qualquer cenário (403
  `User.Forbidden`).

**Transição `Inativo → Ativo`**: não existe. Soft-delete é terminal. Para
voltar, o admin gera novo convite (cria um novo usuário, não reativa o
antigo).

### Estado de perfil

| Estado | Descrição |
|--------|-----------|
| `AccountancyEmployee` | Membro operacional, sem autoridade administrativa. |
| `AccountancyAdmin` | Membro com gestão da equipe e da contabilidade. |
| `SystemAdmin` | Operador da plataforma. Não tem `accountancyId`. |

**Transição `AccountancyEmployee → AccountancyAdmin`** disparada por
`POST /accountancies/{id}/members/{userId}/promote`:

- Solicitante deve ser `AccountancyAdmin` da mesma contabilidade. Caso
  contrário: 403 (`User.ForbiddenForSystemAdmin` para SystemAdmin,
  `User.Forbidden` para outros, `User.ForbiddenTenant` para
  cross-tenant).
- Alvo deve ser `AccountancyEmployee` no momento da operação. Se já for
  `AccountancyAdmin`: 409 `User.AlreadyAdmin`.

**Demoção `AccountancyAdmin → AccountancyEmployee`**: não implementada
(decisão de spec).

### Estado de nome

Edição livre, autosserviço, sem bloqueio além da autenticação válida e dos
limites do validator (`NotEmpty`, `MaximumLength(80)`).

### 9.1. Ordenação, filtros e agrupamentos com semântica de negócio

- **`ListSystemAdmins`**: ordenado por `CreatedAt` decrescente (mais
  recentes primeiro). Filtro implícito `Role == SystemAdmin`. Filtro
  `OnlyActive` controla a inclusão de inativos.
- **`ListAccountancyMembers`**: ordenado por `Role` ascendente (admins
  primeiro, via `SmartEnum.Value` — SystemAdmin=0, AccountancyAdmin=1,
  AccountancyEmployee=2) e depois por `CreatedAt` ascendente. Filtro
  implícito por `AccountancyId` da rota. Filtro `OnlyActive` idem.
- A ordem retornada é a oficial — o frontend não deve reordenar.

---

## 10. Erros esperados e edge cases

Erros vêm no formato `ProblemDetails` do ASP.NET Core (não no envelope
`{statusCode, message, path, traceId, ...}` mencionado no global do
template — ver §16):

```json
{
  "title": "...",
  "type": "User.Code",
  "status": 4xx,
  "detail": "...",
  "errors": null
}
```

| Cenário | HTTP | Code | Mensagem | O que o frontend deve fazer |
|---|---|---|---|---|
| Validação falhou (FirstName vazio etc) | 400 | `Validation.Error` | varia por regra | mostrar erros inline; bloquear botão Salvar até correção |
| Usuário alvo não encontrado / deletado | 404 | `User.NotFound` | "User not found." | toast/mensagem; remover linha da lista local |
| Caller é AccountancyEmployee ou outro perfil sem permissão | 403 | `User.Forbidden` | "You do not have permission to remove users." / "Only an accountancy administrator can promote members." / "You do not have permission to view this user." | esconder CTA na UI; tratar 403 como bug se chegou |
| Caller é SystemAdmin tentando promover | 403 | `User.ForbiddenForSystemAdmin` | "System administrators cannot promote members." | não exibir CTA "Promover" para SA |
| Caller (AccountancyAdmin) age fora do próprio tenant | 403 | `User.ForbiddenTenant` | "You can only manage members of your own accountancy." / "You can only list members of your own accountancy." | bug — não deveria acontecer com routing correto |
| Tentativa de auto-remoção | 409 | `User.CannotRemoveSelf` | "Cannot remove yourself." | desabilitar CTA "Remover" na própria linha do solicitante |
| Tentativa de remover o último AccountancyAdmin | 409 | `User.LastAccountancyAdmin` | "Cannot remove the last administrator of the accountancy." | exibir mensagem orientando a promover outro membro antes |
| Tentativa de remover o último SystemAdmin | 409 | `User.LastSystemAdmin` | "Cannot remove the last system administrator." | exibir mensagem; desabilitar CTA quando lista tem 1 |
| Promover usuário que já é admin | 409 | `User.AlreadyAdmin` | "User is already an accountancy administrator." | atualizar lista local (perfil já era admin) |
| Token inválido / sessão invalidada | 401 | (corpo `{"error":"..."}`) | "Token has been invalidated." | tratar como sessão expirada; redirecionar ao login |

**Edge cases relevantes (todos cobertos no código):**

- **Dois admins se removendo simultaneamente**: a contagem do "último
  admin" é feita na projection MongoDB antes do `AppendEventsAsync`. Há
  uma janela teórica de race entre dois admins clicando ao mesmo tempo em
  uma contabilidade com exatamente 2 admins (eventual consistency da
  projection). O write usa concorrência otimista por `(AggregateId,
  Version)` no event store, mas a checagem de "último" depende da
  projection — ver §16. Em produção real, frequência esperada baixíssima.
- **Usuário deletado tentando agir**: o `OnTokenValidated` no JWT bearer
  carrega o aggregate em toda request. Aggregate deletado é detectado
  via `LoadAggregateAsync` (que já falha em estado deletado) ou via
  `SecurityStamp` divergente após o `RegenerateSecurityStamp`. Resultado:
  401 imediato.
- **Promoção idempotente**: se o caller chamar `promote` duas vezes em
  sequência, a segunda recebe 409 `User.AlreadyAdmin` (estado já é
  AccountancyAdmin).
- **Soft-delete idempotente**: tentar deletar um usuário já deletado
  retorna o erro de `LoadAggregateAsync` (provavelmente `ConflictError` ou
  `NotFoundError`, ver §16) — o frontend deve tratar como já-removido.
- **AccountancyEmployee tenta acessar dados completos**: o backend filtra
  o payload por perfil em `ListAccountancyMembersHandler` independentemente
  do que o frontend solicitou. O AE recebe `UserSummaryResponse[]` mesmo
  que tente forçar via query.

---

## 11. Semântica de autenticação e autorização

Toda rota da feature exige autenticação. JWT Bearer via header
`Authorization`.

**Pipeline de auth no JWT Bearer (`Program.cs`):**

1. Validação padrão do token (issuer, audience, signing key, lifetime).
2. `OnTokenValidated`: extrai `sub` (userId) e `security_stamp` do token,
   carrega o aggregate `User` e compara `SecurityStamp` atual com o do
   token. Se diferente → `context.Fail("Token has been invalidated.")`.

Esse é o mecanismo que garante invalidação **imediata** após
`User.Delete` ou qualquer chamada de `RegenerateSecurityStamp`.

**Policies aplicadas (`IdentityPolicies`):**

| Endpoint | Atributo | Policy/Authorize |
|---|---|---|
| `GetMyProfile` | `[Authorize]` | qualquer usuário autenticado |
| `GetUserById` | `[Authorize]` | qualquer usuário autenticado (matriz de acesso fina no handler) |
| `UpdateUser` | `[Authorize]` | qualquer usuário autenticado |
| `PromoteUser` | `[Authorize(Policy = CanPromoteUser)]` | apenas `AccountancyAdmin` (handler também rejeita SA defensivamente) |
| `DeleteUser` | `[Authorize(Policy = CanManageUsers)]` | `SystemAdmin` ou `AccountancyAdmin` (handler aplica regras adicionais) |
| `ListSystemAdmins` | `[Authorize(Policy = CanManageAccountancies)]` | apenas `SystemAdmin` |
| `ListAccountancyMembers` | `[Authorize]` | qualquer usuário autenticado (handler filtra payload e cross-tenant) |

**Matriz de acesso — `GetUserById` (no handler):**

- `SystemAdmin` → vê qualquer alvo (`SystemAdmin`, `AccountancyAdmin`,
  `AccountancyEmployee`).
- `AccountancyAdmin` → vê apenas `AccountancyAdmin` e
  `AccountancyEmployee` da própria contabilidade.
- `AccountancyEmployee` → vê apenas o próprio usuário.

**Resolução de tenant em `DeleteUser`:**

- Se o caller é `SystemAdmin`: o backend confia no `accountancyId` que
  vem por query param (quando há) e no que está no aggregate alvo. Não há
  cross-check obrigatório com o claim do SA (que não tem `accountancyId`).
- Se o caller é `AccountancyAdmin`: o backend ignora o query param e usa
  o `currentUser.AccountancyId` (claim). O alvo deve ter o mesmo
  `accountancyId`; caso contrário, 403 `User.ForbiddenTenant`.

**Resolução de tenant em `ListAccountancyMembers`:**

- `SystemAdmin`: pode listar qualquer contabilidade.
- Outros perfis: `currentUser.AccountancyId` deve ser igual a
  `request.AccountancyId`; caso contrário 403 `User.ForbiddenTenant`.

**Gap auth real vs auth esperada**: nenhum identificado. A spec exige que
toda autorização passe pelo backend; a implementação respeita isso (UI é
o reflexo, não a barreira).

---

## 12. Impacto de UX que o frontend precisa saber

- **`UpdateUser`**: não retorna body. Frontend deve atualizar o estado
  local com `firstName`/`lastName` enviados (e idealmente fazer um
  `GET /users` em seguida para reconciliar `updatedAt`). Sem
  redirect.
- **`PromoteUser`**: retorna `UserDetailResponse` com o novo `role`. O
  frontend deve atualizar a linha na lista e refletir as novas permissões
  do usuário promovido (que pode ser ele mesmo se for caso de teste).
  Sem redirect.
- **`DeleteUser`**: não retorna body. Frontend que disparou (admin) deve
  remover a linha da lista ativa. Se o admin removeu a si mesmo (bloqueio
  via 409), nada acontece. Se outro admin removeu o usuário logado, a
  próxima request dele falha em 401 — tratar como sessão expirada e
  redirect para login.
- **`Promote` retorna id novo?** Não. O `id` permanece o mesmo (o usuário
  é o mesmo, apenas o `role` mudou).
- **Ordem importa visualmente?** Sim. Backend retorna ordem oficial de
  exibição; não reordenar localmente.
- **Listas têm paginação real**: sempre passar `pageNumber`/`pageSize` ou
  aceitar defaults. Usar `page.hasNextPage` para paginar.
- **Mensagens de erro de negócio (409, 403 de policy de domínio)** devem
  ser exibidas como toast/banner com o `detail` do `ProblemDetails`. Os
  textos do backend já estão em inglês e em linguagem de UI razoável.
- **Confirmação obrigatória em UI** (decisão de spec, frontend deve
  implementar): promoção e remoção exigem modal de confirmação antes da
  chamada. Edição de nome não exige.
- **Campo que precisa de máscara**: nenhum. Não há CPF/CNPJ/telefone
  nesta feature.

---

## 13. Impacto de dados no frontend

| Mutation | Leituras invalidadas |
|---|---|
| `UpdateUser` | `GetMyProfile`, `GetUserById` (do próprio), `ListAccountancyMembers` (linha do usuário com novo nome) |
| `PromoteUser` | `GetUserById` (do alvo), `ListAccountancyMembers` (linha do alvo com novo `role` e mudança de ordenação porque ordena por Role) |
| `DeleteUser` | `ListAccountancyMembers`, `ListSystemAdmins`, `GetUserById` (alvo), `GetMyProfile` (se foi auto, já retornará 401), e quaisquer leituras que mostrem o usuário em listagens admin |

**Stale data**: as listagens leem da projection MongoDB, que é eventual
consistency. Após uma mutation que atualiza a projection, há uma janela
mínima até o evento ser consumido pelo `ProjectionUserConsumer` e o
documento ser atualizado. Mitigar com refetch após sucesso da mutation
e/ou pequena espera antes de relistar.

**UI otimista**: aceitável para `UpdateUser` (alteração de nome é
reversível e local). Para `PromoteUser` e `DeleteUser`, **evitar** UI
otimista — reflita apenas após resposta de sucesso, dado o conjunto de
bloqueios 409 possíveis.

### 13.1. Identificadores estáveis e chaves de reconciliação

- `id` (`Guid`) do usuário: chave estável, nunca muda. Usar como key em
  listas e como chave de reconciliação.
- `email`: estável neste MVP (não é editável), mas **não usar como chave**
  — o futuro inclui possível edição.
- `firstName`, `lastName`, `role`, `accountancyId`, `updatedAt`,
  `isDeleted`: voláteis. Nunca usar como chave de reconciliação.
- `invitationLinkId`: pode ser `null`; usar apenas como referência
  informativa.

### 13.2. Compatibilidade e evolução de contrato

- Esta feature **adiciona** rotas; não substitui nenhum contrato anterior
  do Identity. Coexiste pacificamente com `/auth/*` e `/invitation-link/*`.
- Eventos `UserUpdated`, `UserPromoted`, `UserDeleted` são novos —
  consumidores futuros podem se inscrever.
- Mudanças que exigiriam ajuste obrigatório no frontend (futuro):
  - se a edição de e-mail for habilitada (fora do MVP) — `UpdateUser`
    pode ganhar campo `email` opcional;
  - se a demoção for habilitada — possível novo endpoint `POST .../demote`;
  - se a reativação for habilitada — novo endpoint para reverter
    soft-delete.

---

## 14. Observabilidade e suporte

- O endpoint **não** retorna `traceId` no envelope de erro (o template
  global menciona `traceId` mas o `EndpointBaseExtensions` atual não
  injeta esse campo no `ProblemDetails`). Gap — ver §16.
- Logs relevantes (Serilog + structured logging):
  - Cada `Handle` do `ProjectionUserEventHandler` faz `logger.LogError`
    com `@event.UserId` em falhas de projeção. Um warning é emitido
    quando o aggregate não é encontrado para projeção
    (`UserUpdated`, `UserPromoted`, `UserDeleted`).
  - O `JwtBearerEvents.OnTokenValidated` falha com mensagens curtas
    ("Token has been invalidated.", "Invalid token claims.") que viram
    body 401.
  - Serilog de request é configurado em `Program.cs` com `RequestHost`,
    `RequestScheme`, `UserAgent`, `TenantId` (header `X-Tenant-ID` quando
    presente).
- Para suporte: o frontend pode logar `userId` (do token), `accountancyId`
  (do contexto) e o `code` retornado no `ProblemDetails.Type` (ex.:
  `User.LastAccountancyAdmin`) em casos de erro de negócio.

---

## 15. Exemplos reais de payload

```json
// Request válido — UpdateUser  (PUT /api/v1/identity/users)
{
  "firstName": "Marina",
  "lastName": "Costa"
}

// Response — UpdateUser
// 200 OK, sem body

// Response — GetMyProfile  (GET /api/v1/identity/users)
{
  "id": "8f5f1d44-2a7d-4c1d-9b3a-9c0a2e1f4d11",
  "firstName": "Marina",
  "lastName": "Costa",
  "email": "marina@example.com",
  "role": "AccountancyAdmin",
  "accountancyId": "1f02c4f8-4d1a-49e2-9a72-2b3c8d77a55b",
  "createdAt": "2026-04-10T18:23:11.0000000+00:00",
  "updatedAt": "2026-04-27T14:30:00.0000000+00:00"
}

// Response — GetUserById  (GET /api/v1/identity/users/{userId})
{
  "id": "3a2bf2e1-77b5-4e9d-8e1a-0c1f5e7d0a01",
  "firstName": "Pedro",
  "lastName": "Almeida",
  "email": "pedro@example.com",
  "role": "AccountancyEmployee",
  "accountancyId": "1f02c4f8-4d1a-49e2-9a72-2b3c8d77a55b",
  "createdAt": "2026-04-12T09:00:00.0000000+00:00",
  "updatedAt": null
}

// Response — PromoteUser
// (POST /api/v1/identity/accountancies/{accountancyId}/members/{userId}/promote)
{
  "id": "3a2bf2e1-77b5-4e9d-8e1a-0c1f5e7d0a01",
  "firstName": "Pedro",
  "lastName": "Almeida",
  "email": "pedro@example.com",
  "role": "AccountancyAdmin",
  "joinedAt": "2026-04-12T09:00:00.0000000+00:00",
  "invitationLinkId": "6c0d2a1f-1234-4567-8901-abcdef012345"
}

// Response — DeleteUser
// 200 OK, sem body

// Response — ListSystemAdmins
// (GET /api/v1/identity/system-admins?OnlyActive=true&PageNumber=1&PageSize=10)
{
  "items": [
    {
      "id": "11111111-1111-1111-1111-111111111111",
      "firstName": "Ana",
      "lastName": "System",
      "email": "ana@societiza.com",
      "role": "SystemAdmin",
      "joinedAt": "2026-01-15T12:00:00.0000000+00:00",
      "invitationLinkId": null
    }
  ],
  "page": {
    "pageNumber": 1,
    "pageSize": 10,
    "hasNextPage": false,
    "hasPreviousPage": false,
    "totalCount": 1,
    "totalPages": 1
  }
}

// Response — ListAccountancyMembers (caller AccountancyAdmin)
// (GET /api/v1/identity/accountancies/{accountancyId}/members)
{
  "items": [
    {
      "id": "8f5f1d44-2a7d-4c1d-9b3a-9c0a2e1f4d11",
      "firstName": "Marina",
      "lastName": "Costa",
      "email": "marina@example.com",
      "role": "AccountancyAdmin",
      "joinedAt": "2026-04-10T18:23:11.0000000+00:00",
      "invitationLinkId": "aaaa1111-bbbb-2222-cccc-3333dddd4444"
    },
    {
      "id": "3a2bf2e1-77b5-4e9d-8e1a-0c1f5e7d0a01",
      "firstName": "Pedro",
      "lastName": "Almeida",
      "email": "pedro@example.com",
      "role": "AccountancyEmployee",
      "joinedAt": "2026-04-12T09:00:00.0000000+00:00",
      "invitationLinkId": "6c0d2a1f-1234-4567-8901-abcdef012345"
    }
  ],
  "page": {
    "pageNumber": 1,
    "pageSize": 10,
    "hasNextPage": false,
    "hasPreviousPage": false,
    "totalCount": 2,
    "totalPages": 1
  }
}

// Response — ListAccountancyMembers (caller AccountancyEmployee)
{
  "items": [
    {
      "id": "8f5f1d44-2a7d-4c1d-9b3a-9c0a2e1f4d11",
      "firstName": "Marina",
      "lastName": "Costa",
      "role": "AccountancyAdmin"
    },
    {
      "id": "3a2bf2e1-77b5-4e9d-8e1a-0c1f5e7d0a01",
      "firstName": "Pedro",
      "lastName": "Almeida",
      "role": "AccountancyEmployee"
    }
  ],
  "page": {
    "pageNumber": 1,
    "pageSize": 10,
    "hasNextPage": false,
    "hasPreviousPage": false,
    "totalCount": 2,
    "totalPages": 1
  }
}

// Erro de validação — UpdateUser com FirstName vazio
{
  "title": "Validation Error",
  "type": "Validation.Error",
  "status": 400,
  "detail": "...",
  "errors": [
    { "code": "FirstName", "message": "FirstName is required" }
  ]
}

// Conflito — DeleteUser tentando remover último admin
{
  "title": "Conflict",
  "type": "User.LastAccountancyAdmin",
  "status": 409,
  "detail": "Cannot remove the last administrator of the accountancy."
}

// Conflito — PromoteUser em alguém que já é admin
{
  "title": "Conflict",
  "type": "User.AlreadyAdmin",
  "status": 409,
  "detail": "User is already an accountancy administrator."
}

// Forbidden — SystemAdmin tentando promover
{
  "title": "Forbidden",
  "type": "User.ForbiddenForSystemAdmin",
  "status": 403,
  "detail": "System administrators cannot promote members."
}
```

---

## 16. Gaps, limitações e decisões abertas

### 16.1. Divergência intencional: ausência de rota dedicada para listar inativos

A spec §1 e logic §1 falam em "visão de inativos" como tab/seção separada
para admins. O código não expõe rota dedicada. No lugar disso, as duas
rotas de listagem (`ListSystemAdmins` e `ListAccountancyMembers`)
recebem um query param `OnlyActive` (default `true`) que, quando setado
para `false`, inclui usuários deletados na resposta. O frontend deve
implementar a "visão de inativos" passando `OnlyActive=false` na mesma
rota; não há endpoint específico. O plano técnico deixou isso explícito
como decisão da sessão. Documentado aqui para que o revisor humano valide
o alinhamento com o produto.

### 16.2. Resposta heterogênea de `ListAccountancyMembers`

A rota retorna `IPagedResult<object>`, com `items` sendo
`UserDetailResponse[]` ou `UserSummaryResponse[]` conforme o perfil do
solicitante. O frontend precisa discriminar pelos campos presentes (ou
pelo perfil do solicitante já conhecido na sessão). Considerar padronizar
no futuro com dois endpoints distintos ou um envelope discriminador
explícito (`type: "detail" | "summary"`). Hoje funciona, mas o contrato
TypeScript/Zod no frontend precisa ser uma união discriminada manual.

### 16.3. Atomicidade do "último administrador" sob concorrência

A contagem de admins ativos é feita na projection MongoDB em
`DeleteUserHandler` antes de chamar `User.Delete`. A escrita usa
concorrência otimista por `(AggregateId, Version)` no event store, mas a
projection é eventual consistency — cenário teórico: dois admins clicando
em remover o outro ao mesmo tempo, ambos passam pela contagem (ainda 2
ativos), ambos chegam ao `AppendEventsAsync` do alvo respectivo, ambos
têm sucesso. Resultado: contabilidade pode ficar órfã.

Frequência real esperada baixíssima. Mitigação atual: aceitar a janela
(coerente com o padrão do projeto para `InvitationLink`). Cabe um job de
detecção pós-fato ou um lock pessimista futuro. Frontend não pode evitar
— é responsabilidade backend. Pedir validação humana se for prioridade.

### 16.4. Envelope de erro divergente do template global

O template do `integration.md` global menciona o envelope
`{ statusCode, message, path, traceId, errors, detail }`. O backend
retorna `ProblemDetails` padrão do ASP.NET Core nesta feature
(`{ title, type, status, detail, errors }`) via
`EndpointBaseExtensions.HandleFailure`. Não há `traceId` nem `path` no
body. Frontend precisa tratar a forma real (`ProblemDetails`), não a
descrita no template. Vale alinhar com o tech lead se o envelope deveria
ser uniformizado.

### 16.5. Resposta de `UpdateUser` sem body

A rota retorna 200 sem body. A logic §4 do `logic.md` diz que "o nome
atualizado do usuário autenticado [deve refletir] na interface
imediatamente (menu de perfil, cabeçalho, listagem)". O frontend pode
fazer um `GET /users` em seguida ou apenas atualizar o estado local com
os valores enviados (sem `updatedAt` exato). Considerar futuro: retornar
`UserProfileResponse` para reconciliação direta.

### 16.6. Confirmação de exclusão idempotente

Se o caller chamar `DELETE /users/{userId}` para um usuário já deletado,
o `LoadAggregateAsync` falha — não está documentado no código com qual
tipo de erro exato (`NotFoundError` ou `ConflictError`, dependendo do
comportamento de `LoadAggregateAsync` com `IsDeleted == true`). O
frontend deve tratar 404 e 409 como "já-removido" com a mesma reação
visual.

### 16.7. Identificação do responsável técnico

Não foi possível identificar o responsável técnico no código (não há
metadado, owners file ou cabeçalho de autoria). Confirmar com o tech
lead do Identity antes de considerar o documento canônico.

### 16.8. `ListSystemAdmins` e `ListAccountancyMembers` retornam `IPagedResult<object>`

O tipo runtime é `IPagedResult<object>` mesmo quando o conteúdo é sempre
`UserDetailResponse[]` (no caso de `ListSystemAdmins`). É um detalhe de
implementação que casa com a heterogeneidade de
`ListAccountancyMembers`, mas pode confundir geração de schema/typed SDK
do lado do frontend. Considerar tipos concretos no futuro.

### 16.9. Falta endpoint para listar deletados de forma exclusiva

Conforme §16.1, não há rota tipo `GET /users/deleted`. Se a UX exigir uma
tela exclusivamente de inativos com paginação independente, isso requer
uma rota nova ou um filtro adicional explícito — hoje a rota mistura
ativos+inativos quando `OnlyActive=false`.
