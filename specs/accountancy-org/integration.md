---
status: draft
spec: accountancy-org
created_at: 2026-04-21
---

# Accountancy — Contrato de Integração Backend → Frontend

## 1. Identificação da feature

- **Nome da feature**: Accountancy — Unidade Organizacional
- **Módulo backend responsável**: `Accountancy` (`src/Modules/Accountancy`)
- **Status atual**: `existente`
- **Data da última atualização**: 2026-04-21
- **Responsável técnico**: a definir pelo tech-lead do backend

O backend já expõe 5 endpoints sob `v1/accountancy` para criar, listar,
consultar por id, consultar a contabilidade da sessão ("Minha
Contabilidade") e atualizar os dados cadastrais. Toda a escrita é restrita
ao perfil `SystemAdmin`; a leitura de "Minha Contabilidade" é aberta a
qualquer usuário autenticado que tenha a claim `accountancy_id` emitida
no access token.

---

## 2. Objetivo de negócio da feature

A Accountancy é a representação digital de um escritório de contabilidade
dentro da Societiza. Cada contabilidade cadastrada é a fronteira do tenant:
processos, templates, usuários e tudo mais vivem dentro do escopo de uma
`AccountancyId`. Sem contabilidade cadastrada, nenhum outro módulo do produto
consegue separar dados por cliente.

Esta feature atende três consumidores distintos no frontend:

- O **back-office da Societiza** (`SystemAdmin`), que precisa listar,
  criar, consultar e editar os dados cadastrais de qualquer contabilidade.
- O **AccountancyAdmin**, que precisa apenas visualizar "Minha
  Contabilidade" — somente leitura.
- O **AccountancyEmployee**, que tem exatamente a mesma experiência de
  leitura do AccountancyAdmin em "Minha Contabilidade".

A entrada natural no produto é:

1. `SystemAdmin` cadastra a contabilidade via back-office.
2. O link de convite (feature de Identity) é gerado apontando para a
   `AccountancyId` recém-criada.
3. O primeiro `AccountancyAdmin` aceita o convite e, no primeiro login,
   encontra "Minha Contabilidade" como confirmação visual de que a firma
   existe dentro da Societiza.

Esta doc cobre somente o módulo `Accountancy`. A geração de convites, o
ciclo de registro de usuário e a emissão de token ficam no módulo
`Identity` e devem ser consumidos pelas specs correspondentes.

---

## 3. Escopo atual versus escopo futuro

**O que já está implementado no backend hoje:**

- `POST v1/accountancy` — criar contabilidade (SystemAdmin).
- `GET v1/accountancy` — listar contabilidades paginadas (SystemAdmin).
- `GET v1/accountancy/{accountancyId}` — consultar por id (SystemAdmin).
- `GET v1/accountancy/me` — "Minha Contabilidade" (qualquer usuário
  autenticado com claim `accountancy_id`).
- `PUT v1/accountancy/{accountancyId}` — atualizar dados cadastrais
  (SystemAdmin).
- Validação estrutural local de CNPJ, CEP e telefone no pipeline de
  FluentValidation.
- Unicidade de CNPJ validada tanto na **criação** quanto na **edição**
  (o `PUT` verifica se o CNPJ pertence a outra contabilidade antes de
  gravar e retorna `409 CNPJAlreadyExists` quando há conflito).
- Persistência de CNPJ, CEP e telefone **sem formatação** (apenas
  dígitos).
- Soft-delete flag na projeção (`IsDeleted`) — listagem filtra por
  `!IsDeleted`, mas não há endpoint que altere este valor.
- Ordenação fixa por `LegalName` e, como desempate, `CreatedAt` na
  listagem.
- Paginação com total de registros e total de páginas expostos no
  envelope da listagem.

**O que ainda não está implementado:**

- Controle de concorrência otimista na edição. Dois SystemAdmins
  editando simultaneamente: vence o último `save`.
- Endpoint para exclusão, inativação, reativação ou qualquer mudança
  de ciclo de vida — intencional, não existe na spec.

**O que existe no domínio mas não está exposto por endpoint:**

- Evento `AccountancyUpdated` com `UpdatedAt` preenchido pelo agregado
  já é publicado pela infraestrutura de event-sourcing, mas nenhum
  endpoint externo expõe o histórico de eventos.
- Flag `IsDeleted` na projeção: o domínio não emite evento de deleção
  nem o agregado expõe método para removê-la; a flag existe apenas para
  permitir filtro defensivo no read model.

**O que é apenas visão futura de produto:**

- Self-service de edição pelo `AccountancyAdmin`.
- Busca textual e filtros operacionais na listagem de `SystemAdmin`.
- Identidade visual customizável (logo, banner, cores).
- Gestão de múltiplas contabilidades por usuário (`accountancy-offices`,
  arquivada no product.societiza).

**O que ainda está em discussão:**

- Nenhuma discussão aberta relevante ao frontend identificada no código
  ou na spec de produto aprovada.

---

## 4. Contexto de produto e semântica da feature

Do ponto de vista do usuário final, a feature aparece em dois lugares
bem diferentes:

- **Para o `SystemAdmin`**, é um back-office: uma listagem de
  contabilidades, um formulário de criação e um formulário de edição.
  Sucesso é "conseguir provisionar um novo cliente rapidamente".
  Erro funcional é "criar ou editar com CNPJ duplicado" (bloqueado com
  `409 CNPJAlreadyExists` tanto no `POST` quanto no `PUT`) ou "submeter
  um formulário com campo obrigatório ausente / CNPJ estruturalmente
  inválido" (`400` com ProblemDetails).

- **Para o `AccountancyAdmin` e o `AccountancyEmployee`**, é uma tela
  identitária: "Minha Contabilidade". Sucesso é "entrar no produto e ver
  claramente em qual firma eu estou operando". A tela é estritamente
  somente leitura; não existe nenhum controle de edição para estes perfis,
  e o backend rejeita (401/403) qualquer tentativa de escrita via
  manipulação direta de requisição.

Não há ciclo de vida da entidade. Uma contabilidade está sempre no
estado **cadastrada** depois de criada — não existe inativar, suspender,
arquivar ou excluir. A única transição observável é a de dados
cadastrais atualizados, o que só altera o snapshot visível e o campo
`updatedAt` da resposta.

---

## 5. Dependências e relações com outras features

- **Identity (autenticação e claims)** — o access token emitido pelo
  `Identity` carrega as claims `sub` (userId), `email`, `role` e,
  quando aplicável, `accountancy_id`. Todos os endpoints de
  `Accountancy` exigem autenticação JWT. A tela "Minha Contabilidade"
  depende diretamente da claim `accountancy_id` estar presente no token.
- **Identity (convite de usuários)** — o fluxo de convite aponta para
  uma `accountancyId` já existente. A criação da contabilidade é
  pré-requisito do fluxo de onboarding de usuário. Este documento **não**
  cobre convites; isso pertence à spec `identity-invitation-links`.
- **Módulos futuros (Societário, Dashboard, Workflow em produção,
  Notificações)** — todos consumirão `accountancyId` como discriminante
  multi-tenant. O frontend deve assumir que `accountancyId` é a chave de
  scoping de qualquer listagem ou leitura transversal.

Posição no produto: **feature fundacional**. Sem contabilidade
cadastrada, os demais módulos não têm referência de tenant.

---

## 6. Rotas públicas

Todas as rotas vivem sob `UsePathBase("/api")` com versionamento em
segmento de URL `v1`. Caminhos completos:

| Método | Path | Tipo | Finalidade | Params de rota | Pronta para frontend? |
|---|---|---|---|---|---|
| POST   | `/api/v1/accountancy`                       | Command | Criar contabilidade (SystemAdmin). Retorna o `Id` gerado. | — | sim |
| GET    | `/api/v1/accountancy`                       | Query   | Listar contabilidades paginadas, ordenadas por `LegalName` (SystemAdmin). | — | sim |
| GET    | `/api/v1/accountancy/{accountancyId}`       | Query   | Consultar detalhe de qualquer contabilidade por id (SystemAdmin). | `accountancyId: Guid` | sim |
| GET    | `/api/v1/accountancy/me`                    | Query   | "Minha Contabilidade" — retorna a contabilidade associada ao usuário autenticado via claim `accountancy_id`. | — | sim |
| PUT    | `/api/v1/accountancy/{accountancyId}`       | Command | Atualizar dados cadastrais de uma contabilidade (SystemAdmin). Corpo completo obrigatório. | `accountancyId: Guid` | sim |

Notas operacionais:

- Versionamento: além de `/v1/` no path, o backend aceita o header
  `X-Api-Version: 1.0` e assume a versão padrão quando omitido.
- Não há suporte a `PATCH` parcial. O `PUT` é um **replace completo**
  dos campos editáveis; todos precisam ser enviados no body.
- Não há endpoint `DELETE`. Exclusão é decisão explícita de produto
  (`spec.md` §6).

---

## 7. Contrato de request

### Request — `POST /api/v1/accountancy` (criar contabilidade)

Corpo (`application/json`):

- `CNPJ`: string, obrigatório, 14 dígitos estruturalmente válidos. Aceita
  entrada com ou sem máscara — o backend roda `OnlyDigits()` antes de
  validar estrutura e persistir. Validação estrutural é local (algoritmo
  de dígitos verificadores de CNPJ); não há chamada a Receita Federal.
- `LegalName`: string, obrigatório, até 150 caracteres. É a Razão Social.
- `TradeName`: string, **opcional**. Até 150 caracteres quando enviado.
  `null` ou ausente é aceito.
- `Address`: string, obrigatório, até 200 caracteres. Texto livre.
- `City`: string, obrigatório, até 100 caracteres.
- `State`: string, obrigatório, máximo 2 caracteres (sigla UF, ex.
  `SP`). O backend valida o tamanho máximo; **não** valida se a sigla
  corresponde a uma UF real.
- `PostalCode`: string, obrigatório. Aceita com ou sem máscara; o
  backend extrai dígitos e valida que tenha 8 dígitos e não seja
  sequência repetida (ex. `00000000` é rejeitado).
- `Phone`: string, obrigatório. Aceita com ou sem máscara; o backend
  extrai dígitos e exige entre 9 e 11 dígitos, não-repetidos.
- `Email`: string, **opcional**. Quando enviado, precisa ter formato de
  e-mail válido e até 100 caracteres. `null` ou ausente é aceito.

Transformações obrigatórias no frontend:

- Nenhuma transformação é obrigatória antes de enviar. Campos com
  máscara podem ser enviados com ou sem máscara — o backend normaliza.
- Recomendado: enviar sempre em formato canônico (apenas dígitos para
  CNPJ, CEP e telefone) para evitar ambiguidade em logs.

Validações relevantes (todas aplicadas no servidor, retornam `400 Bad
Request` com `ProblemDetails` + lista de erros em `extensions.errors`):

- `CNPJ is required` — CNPJ vazio.
- `CNPJ must be valid` — CNPJ estruturalmente inválido.
- `Legal name is required` / `Legal name cannot exceed 150 characters`.
- `Trade name cannot exceed 150 characters` (se enviado).
- `Address is required` / `Address cannot exceed 200 characters`.
- `City is required` / `City cannot exceed 100 characters`.
- `State is required` / `State must have a maximum of 2 characters`.
- `Postal code is required` / `Postal code must be valid`.
- `Phone is required` / `Phone number must be valid`.
- `Email must be valid` / `Email cannot exceed 100 characters` (se
  enviado).

### Request — `GET /api/v1/accountancy` (listar)

Query string:

- `pageNumber`: inteiro, opcional, default `1`. Deve ser ≥ `1`.
- `pageSize`: inteiro, opcional, default `10`. Deve estar entre `1` e
  `100`.

Sem corpo. Sem parâmetros de busca, filtro ou ordenação — são decisões
fechadas de produto; ordenação é fixa (`LegalName` ascendente, desempate
por `CreatedAt` ascendente).

Validações (`400 Bad Request` quando falham):

- `Page number must be greater than or equal to 1`.
- `Page size must be greater than or equal to 1`.
- `Page size cannot exceed 100`.

### Request — `GET /api/v1/accountancy/{accountancyId}` (detalhe por id)

Parâmetros de rota:

- `accountancyId`: `Guid`, obrigatório. A rota já restringe via
  constraint `:guid` — `Guid` mal formado não casa a rota e retorna
  `404` do roteador antes de chegar ao endpoint.

Sem corpo. Sem query string.

### Request — `GET /api/v1/accountancy/me` ("Minha Contabilidade")

Não tem parâmetros de rota, query string ou corpo. A contabilidade é
resolvida exclusivamente pela claim `accountancy_id` do usuário
autenticado.

### Request — `PUT /api/v1/accountancy/{accountancyId}` (atualizar)

Parâmetros de rota:

- `accountancyId`: `Guid`, obrigatório.

Corpo (`application/json`): **replace completo**. Todos os campos
abaixo são obrigatórios no body — mesmo os opcionais para o domínio
precisam ser enviados (pelo menos como `null`):

- `CNPJ`: mesma regra da criação. **Unicidade é validada**: se o CNPJ
  informado pertencer a outra contabilidade (diferente de
  `accountancyId` da rota), o handler retorna `409 Conflict` com código
  `CNPJAlreadyExists`.
- `LegalName`: mesma regra.
- `TradeName`: opcional no domínio, mas **precisa estar presente no body**
  (aceita `null`).
- `Address`: mesma regra.
- `City`: mesma regra.
- `State`: mesma regra.
- `PostalCode`: mesma regra.
- `Phone`: mesma regra.
- `Email`: opcional no domínio, mas **precisa estar presente no body**
  (aceita `null`).

Transformações obrigatórias no frontend:

- Carregar os dados atuais via `GET /api/v1/accountancy/{id}` antes de
  editar, para repopular o formulário. Enviar apenas os campos alterados
  **quebra** o contrato — o backend interpreta ausência como string
  vazia e falha na validação.

Validações: idênticas às do `POST`, mais `Id is required` /
`Id must be a valid GUID` quando o Guid é vazio.

---

## 8. Contrato de response

### Response — `POST /api/v1/accountancy`

- `id`: `Guid` — identificador gerado para a nova contabilidade. Único
  campo da resposta (tipo `IdentifierResponse`).

Uso no frontend: guardar este `id` para usá-lo no próximo passo do
fluxo de onboarding (geração de link de convite em Identity). A
resposta **não** inclui os dados cadastrais de volta — para exibi-los,
faça `GET /api/v1/accountancy/{id}` em seguida.

Observação sobre consistência: o projection MongoDB é populado via
event handler disparado pelo append de eventos. A operação é síncrona
dentro do mesmo pipeline de request (não há enfileiramento), então o
`GET` imediato após o `POST` deve retornar os dados recém-gravados.

### Response — `GET /api/v1/accountancy` (lista paginada)

Envelope `IPagedResult<AccountancyDetailResponse>`:

- `items`: array de `AccountancyDetailResponse` (ver estrutura abaixo).
  Até `pageSize` elementos.
- `page`:
  - `pageNumber`: inteiro — página atual.
  - `pageSize`: inteiro — tamanho da página.
  - `hasNextPage`: boolean — se existe página seguinte.
  - `hasPreviousPage`: boolean — se existe página anterior.
  - `totalCount`: inteiro — total absoluto de contabilidades que
    satisfazem a consulta (independente de `pageSize`).
  - `totalPages`: inteiro — total de páginas calculado a partir de
    `totalCount` e `pageSize`.

Com `totalCount` e `totalPages` o frontend tem tudo o que precisa para
renderizar controles como "Página 3 de 12" ou salto direto para a
última página.

### Response — `GET /api/v1/accountancy/{accountancyId}` e `GET /api/v1/accountancy/me`

Tipo `AccountancyDetailResponse`:

- `id`: `Guid` — identificador estável da contabilidade.
- `cnpj`: string — **sem máscara**, apenas dígitos. Sempre presente.
- `legalName`: string — razão social. Sempre presente.
- `tradeName`: string | null — nome fantasia. Pode vir `null` quando não
  foi preenchido no cadastro.
- `address`: string — logradouro / endereço. Sempre presente.
- `city`: string — sempre presente.
- `state`: string (2 caracteres) — sigla UF. Sempre presente.
- `postalCode`: string — **sem máscara**, 8 dígitos. Sempre presente.
- `phone`: string — **sem máscara**, apenas dígitos (9 a 11 dígitos).
  Sempre presente.
- `email`: string | null — pode vir `null` quando não foi preenchido.
- `createdAt`: `DateTimeOffset` ISO-8601, UTC — data de criação.
- `updatedAt`: `DateTimeOffset` ISO-8601 | null — data da última
  atualização; `null` enquanto a contabilidade nunca foi editada.

### Response — `PUT /api/v1/accountancy/{accountancyId}`

Sem corpo (`200 OK` vazio). Para obter o estado atualizado, o frontend
precisa re-consultar `GET /api/v1/accountancy/{id}`.

### 8.1. Formato canônico versus formato de exibição

- `cnpj`, `postalCode`, `phone` chegam **sem máscara** (apenas dígitos).
  O frontend aplica:
  - CNPJ → `XX.XXX.XXX/XXXX-XX`
  - CEP → `XXXXX-XXX`
  - Telefone → `(XX) XXXXX-XXXX` (11 dígitos) ou `(XX) XXXX-XXXX`
    (10 dígitos); o backend também aceita 9 dígitos (sem DDD),
    embora seja o caso mais raro.
- `state` chega pronto em sigla UF maiúscula esperada; o backend não
  normaliza caixa, então se o `SystemAdmin` cadastrou `sp` minúsculo,
  o frontend recebe `sp`. Recomendado forçar `UPPER` no formulário.
- Todas as datas (`createdAt`, `updatedAt`) chegam em ISO-8601 com
  offset UTC. Formatar para a timezone do usuário é responsabilidade do
  frontend.

### 8.2. Nullabilidade e ausência de campo

- `tradeName` e `email`: podem vir **`null`** (nunca string vazia). A
  serialização JSON do backend preserva explicitamente `null` — esses
  dois campos **sempre existem na resposta**, mas podem ter valor
  `null`.
- `updatedAt`: `null` sinaliza semanticamente "nunca foi editada".
  Qualquer edição bem-sucedida preenche o campo. Se o frontend precisar
  distinguir "nunca editada" de "editada há muito tempo", confie em
  `updatedAt === null` como critério.
- Os demais campos obrigatórios (`id`, `cnpj`, `legalName`, `address`,
  `city`, `state`, `postalCode`, `phone`, `createdAt`) nunca vêm `null`
  ou omitidos em respostas de sucesso.

Regra prática para a tela "Minha Contabilidade": quando `tradeName`
ou `email` são `null`, omitir inteiramente o rótulo — não exibir
placeholder "—" ou "Não informado" (decisão de `logic.md`, não do
código, mas compatível com o contrato).

---

## 9. Estados de negócio e transições

A entidade tem um único estado operacional: **cadastrada**. Não há
ativo/inativo, suspensão, arquivamento ou exclusão. Todas as rotas
retornam e operam sobre contabilidades "cadastradas".

Transições observáveis:

- `(não existe)` → `cadastrada`: gatilho é `POST` bem-sucedido. Backend
  emite `AccountancyCreated`, persiste no event store e atualiza a
  projeção síncrona.
- `cadastrada (snapshot A)` → `cadastrada (snapshot B)`: gatilho é
  `PUT` bem-sucedido. Backend emite `AccountancyUpdated` e preenche
  `UpdatedAt`. O `CreatedAt` nunca muda.

CTAs que dependem desses estados:

- Existe contabilidade → é possível gerar link de convite (feature de
  Identity). Frontend deve habilitar o CTA somente quando o `POST`
  confirmar o id.
- Após `PUT`, a UI deve recarregar o snapshot com `GET /{id}` e
  exibir sinalização mínima de "atualizado há X" usando `updatedAt`.

### 9.1. Ordenação, filtros e agrupamentos com semântica de negócio

- Ordenação oficial do domínio na listagem: **`LegalName` ascendente,
  desempate por `CreatedAt` ascendente**. Essa é a ordem que o
  `SystemAdmin` espera ver no back-office. O frontend **não deve**
  reordenar localmente; se reordenar por outro critério, deixe isso
  explícito como ação do usuário, não como default.
- Não há suporte nativo a filtro (nem por nome, nem por UF, nem por
  intervalo de data). Qualquer filtragem hoje é client-side sobre a
  página corrente — limitação conhecida, não bug.
- Listagem filtra automaticamente `IsDeleted = false`. Como não há
  endpoint que marque `IsDeleted = true`, na prática isso é um
  no-op hoje, mas mantém o read model preparado para evolução futura.
- Agrupamentos: nenhum suportado.

---

## 10. Erros esperados e edge cases

Os endpoints convivem com **dois envelopes de erro distintos**, ambos
aplicados por camadas diferentes do pipeline. O frontend precisa tratar
os dois. A regra é simples:

- Falhas retornadas pelo pipeline `Result` / `HandleFailure` (validação,
  NotFound, Conflict, Forbidden oriundo do domínio, Bad Request
  genérico) saem como `ProblemDetails` (`application/problem+json`).
- Falhas capturadas pelo `ExceptionHandlingMiddleware` (exceções de
  autenticação/autorização do ASP.NET, NotFound via exceção e qualquer
  exceção não tratada) saem como envelope simplificado
  `{ error, message }`.

Este padrão é idêntico em Identity (Auth, Invitation, User) e em
Accountancy — não há diferença de envelope por módulo.

| Status | Quando acontece | Envelope | O que o frontend deve fazer |
|---|---|---|---|
| `400 Bad Request` | Validação FluentValidation falhou (ex. CNPJ inválido, CEP inválido, campo obrigatório ausente) **ou** paginação fora do range. Tipo `ValidationError`; `errors` traz a lista completa de códigos/mensagens. | `ProblemDetails` | Mapear erros por código (`CNPJ must be valid`, etc.) para feedback inline no formulário. Mensagens estão em inglês — o frontend traduz para pt-BR. |
| `400 Bad Request` | Regra de domínio violada (qualquer `Error` genérico que não seja NotFound/Conflict/Forbidden/Validation). `type` traz o código. | `ProblemDetails` | Exibir `detail` do backend como fallback e logar `X-Correlation-ID` do header. |
| `401 Unauthorized` | Access token ausente, expirado ou com `security_stamp` inválido. | `{ error: "Unauthorized", message }` | Redirecionar para login. Não tentar retry. |
| `403 Forbidden` (via pipeline Result) | Falha `ForbiddenError` retornada pelo handler (ex. `GET /me` sem claim `accountancy_id` → código `AccountancyClaimMissing`). | `ProblemDetails` | Ler `type` para identificar o motivo (`AccountancyClaimMissing` etc.) e agir conforme o caso. |
| `403 Forbidden` (via middleware) | Policy do ASP.NET rejeitou o acesso (ex. AccountancyAdmin chamando rota de SystemAdmin). | `{ error: "Forbidden", message }` | Esconder a UI proibida e, se chegar aqui, mostrar erro genérico de permissão. |
| `404 Not Found` (via pipeline Result) | `GET /{id}` ou `GET /me` quando o id/claim aponta para contabilidade inexistente (código `AccountancyNotFound`). | `ProblemDetails` | Exibir estado "não encontrada" e oferecer retorno à listagem. |
| `404 Not Found` (via router / middleware) | Rota inexistente ou `accountancyId` que não casa a constraint `:guid`; `NotFoundException` lançada. | vazio (router) ou `{ error: "NotFound", message }` (middleware) | Rota errada — sinalizar como 404 genérico. |
| `409 Conflict` | `POST` ou `PUT` quando o CNPJ já existe em outra contabilidade (código `CNPJAlreadyExists`). | `ProblemDetails` | Sinalizar o campo CNPJ no formulário com mensagem "CNPJ já cadastrado". |
| `500 Internal Server Error` | Exceção não tratada. | `{ error: "InternalServerError", message: "An unexpected error occurred." }` | Exibir mensagem genérica e instruir o usuário a tentar novamente. |

Estrutura exata dos envelopes — ver exemplos em §15.

Edge cases relevantes:

- **Formulário de edição enviado sem `tradeName`/`email`**: como o body
  é replace completo e esses campos são opcionais no domínio, enviar
  `null` é a forma correta de apagar os valores anteriores.
- **Múltiplos SystemAdmins editando a mesma contabilidade**: o último
  `save` vence. Não há ETag, `If-Match` nem versão otimista exposta.
  Decisão aceita em `logic.md` §5; frontend não precisa mostrar aviso.
- **Listagem vazia**: `items = []`, `page.totalCount = 0`,
  `page.totalPages = 0`, `page.hasNextPage = false`,
  `page.hasPreviousPage = false`. Frontend renderiza estado vazio.
- **`GET /me` para SystemAdmin**: SystemAdmin não tem claim
  `accountancy_id`. O endpoint retorna `403` com código
  `AccountancyClaimMissing` — não `404`. Não exibir essa rota no menu
  desse perfil.
- **Edição para CNPJ de outra contabilidade**: bloqueada no `PUT`
  exatamente como no `POST` — `409 CNPJAlreadyExists`. O frontend deve
  tratar o 409 igual no formulário de criação e no de edição.

---

## 11. Semântica de autenticação e autorização

- **Todas as rotas exigem access token JWT válido** (Bearer). O esquema
  padrão valida assinatura, issuer, audience, lifetime e confere
  `security_stamp` do usuário contra o banco — se o stamp foi invalidado
  (ex. logout global, troca de senha), o request recebe `401`.
- **Claim obrigatórias no access token**: `sub` (userId `Guid`),
  `email`, `role`. A claim `accountancy_id` é **opcional** — SystemAdmin
  não possui, AccountancyAdmin e AccountancyEmployee possuem.
- **Policies** (em `IdentityPolicies.Register`):
  - `CanManageAccountancies` → aplicada em `POST`, `GET /`, `GET /{id}`
    e `PUT`. Permitida **apenas para `SystemAdmin`**.
  - `GetMyAccountancyEndpoint` usa `[Authorize]` simples — permite
    qualquer usuário autenticado, e internamente valida presença da
    claim `accountancy_id`.
- **Regra prática para o frontend por perfil**:
  - `SystemAdmin`: vê back-office completo; **não** deve ver menu
    "Minha Contabilidade" (não resolve).
  - `AccountancyAdmin`: vê apenas "Minha Contabilidade" somente leitura;
    **não** deve ver back-office. Tentativas de acesso direto retornam
    `403`.
  - `AccountancyEmployee`: idêntico ao `AccountancyAdmin` no escopo
    desta feature.
- **Gap auth × produto**: nenhum identificado neste módulo. Produto,
  `logic.md` e código concordam: SystemAdmin escreve, os demais apenas
  leem via `/me`.

---

## 12. Impacto de UX que o frontend precisa saber

- **`POST` retorna id novo**. O frontend precisa capturar `id` da
  resposta e, no próximo passo do wizard, usá-lo como parâmetro para
  geração de convite em outra rota (fora do escopo deste doc).
- **`PUT` não retorna dados atualizados.** Após sucesso, o frontend
  deve re-consultar `GET /api/v1/accountancy/{id}` para repopular o
  snapshot e qualquer tela aberta.
- **Ordem da lista é autoritativa do backend.** Não reordenar por
  default; oferecer reordenação apenas como ação consciente do usuário.
- **Campos `cnpj`, `postalCode`, `phone` chegam sem máscara.** O
  frontend deve aplicar máscara para exibição (inclusive em
  `/me`). No formulário, pode aceitar máscara ou não — o backend
  normaliza.
- **Campos nulos não exibir.** Em "Minha Contabilidade", quando
  `tradeName` ou `email` vêm `null`, não renderizar rótulo.
- **Mensagens de erro vêm em inglês.** O frontend deve mapear códigos
  para mensagens localizadas pt-BR; exibir a mensagem crua do backend só
  como fallback.
- **Feedback inline para erros de validação.** Mapear `type` ou lista
  em `errors` para campo correspondente. Para `409` (criação ou edição),
  foco no campo `CNPJ`.
- **Pós-mutação**:
  - `POST`: redirecionar para o próximo passo do onboarding (geração de
    convite) ou para a tela de detalhe da contabilidade recém-criada.
  - `PUT`: permanecer na tela de detalhe e sinalizar "dados
    atualizados".

---

## 13. Impacto de dados no frontend

- Após `POST`, invalidar a query de listagem de contabilidades para
  que a próxima navegação para `GET /` mostre o registro novo.
- Após `PUT`, invalidar tanto o `GET /{id}` do item editado quanto
  qualquer `GET /` em cache — a ordem por `LegalName` pode ter mudado
  se a razão social foi alterada.
- "Minha Contabilidade" (`/me`) é segura de cachear longamente enquanto
  o SystemAdmin não editar. Como o usuário logado não pode editar a
  própria contabilidade, na prática invalidar só em login/refresh já é
  suficiente no fluxo do produto atual.
- **UI otimista**: viável em ambos os casos, com atenção — tanto `POST`
  quanto `PUT` podem falhar com `409 CNPJAlreadyExists` que a UI não
  consegue prever antes do request.

### 13.1. Identificadores estáveis e chaves de reconciliação

- `id` (`Guid`) é chave estável, única, gerada pelo backend. Usar como
  chave de lista (`key` em React) e como chave de reconciliação em
  cache (TanStack Query).
- `cnpj` é único por regra de negócio (validado em criação e edição),
  mas **não** deve ser usado como chave de reconciliação no frontend —
  porque pode mudar via `PUT` (e essa mudança não invalida o `id`).
- `legalName` e `tradeName` são texto livre e mutáveis; nunca usar
  como chave.
- `createdAt` é imutável depois de criado; pode servir como tie-breaker
  de ordenação local.

### 13.2. Compatibilidade e evolução de contrato

- Contrato novo. Não coexiste com nenhum contrato legado no frontend.
- Campos `tradeName` e `email` podem sair de `null` para valor a
  qualquer momento (via `PUT`); o frontend precisa estar preparado
  para essa transição após invalidação da query.

---

## 14. Observabilidade e suporte

- **Correlation ID**: o middleware `UseCorrelationId` adiciona um
  header de correlação por request. O frontend deve propagar o
  `X-Correlation-ID` recebido (ou gerar um próprio) para amarrar logs
  cliente-servidor. Não é formalmente um `traceId` no corpo do erro,
  mas serve como identificador para suporte.
- **Logs de servidor**: Serilog registra método, path, status, duração
  e metadados de host/agent. Falhas de projeção (`ProjectionAccountancyEventHandler`)
  logam `Failed to create accountancy: {Id}` / `Failed to update accountancy: {Id}`.
- **Recomendação para o frontend**: em qualquer erro 5xx ou 4xx não
  tratado, logar `{ method, path, status, correlationId, errorCode,
  errorMessage }` e expor o `correlationId` ao usuário como referência
  para suporte.
- **Swagger**: em ambiente de desenvolvimento, disponível em
  `http://localhost:5258/api/swagger`.

---

## 15. Exemplos reais de payload

```json
// Request — POST /api/v1/accountancy
{
  "cnpj": "11.222.333/0001-81",
  "legalName": "Contabilidade Alpha LTDA",
  "tradeName": "Alpha Contábil",
  "address": "Av. Paulista, 1000, Sala 501",
  "city": "São Paulo",
  "state": "SP",
  "postalCode": "01310-100",
  "phone": "(11) 99999-8888",
  "email": "contato@alpha.com.br"
}
```

```json
// Response — POST /api/v1/accountancy (200)
{
  "id": "7c9b8f1a-4d2e-4a6b-9c3f-1e2d5a7b8c9e"
}
```

```json
// Response — GET /api/v1/accountancy/{accountancyId} (200)
{
  "id": "7c9b8f1a-4d2e-4a6b-9c3f-1e2d5a7b8c9e",
  "cnpj": "11222333000181",
  "legalName": "Contabilidade Alpha LTDA",
  "tradeName": "Alpha Contábil",
  "address": "Av. Paulista, 1000, Sala 501",
  "city": "São Paulo",
  "state": "SP",
  "postalCode": "01310100",
  "phone": "11999998888",
  "email": "contato@alpha.com.br",
  "createdAt": "2026-04-21T14:05:32.128+00:00",
  "updatedAt": null
}
```

```json
// Response — GET /api/v1/accountancy/me (200) — com tradeName e email nulos
{
  "id": "7c9b8f1a-4d2e-4a6b-9c3f-1e2d5a7b8c9e",
  "cnpj": "11222333000181",
  "legalName": "Contabilidade Alpha LTDA",
  "tradeName": null,
  "address": "Av. Paulista, 1000, Sala 501",
  "city": "São Paulo",
  "state": "SP",
  "postalCode": "01310100",
  "phone": "11999998888",
  "email": null,
  "createdAt": "2026-04-21T14:05:32.128+00:00",
  "updatedAt": "2026-04-21T15:12:07.431+00:00"
}
```

```json
// Response — GET /api/v1/accountancy?pageNumber=1&pageSize=10 (200)
{
  "items": [
    {
      "id": "7c9b8f1a-4d2e-4a6b-9c3f-1e2d5a7b8c9e",
      "cnpj": "11222333000181",
      "legalName": "Contabilidade Alpha LTDA",
      "tradeName": "Alpha Contábil",
      "address": "Av. Paulista, 1000, Sala 501",
      "city": "São Paulo",
      "state": "SP",
      "postalCode": "01310100",
      "phone": "11999998888",
      "email": "contato@alpha.com.br",
      "createdAt": "2026-04-21T14:05:32.128+00:00",
      "updatedAt": null
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
```

```json
// Request — PUT /api/v1/accountancy/{accountancyId}
{
  "cnpj": "11.222.333/0001-81",
  "legalName": "Contabilidade Alpha Ltda ME",
  "tradeName": null,
  "address": "Av. Paulista, 1000, Sala 501",
  "city": "São Paulo",
  "state": "SP",
  "postalCode": "01310-100",
  "phone": "(11) 3333-4444",
  "email": null
}
```

```json
// Erro de validação — 400 Bad Request (ProblemDetails, via HandleFailure)
{
  "title": "Validation Error",
  "type": "ValidationError",
  "detail": "One or more validation failures have occurred.",
  "status": 400,
  "errors": [
    { "code": "CNPJ must be valid", "message": "CNPJ must be valid" },
    { "code": "Postal code must be valid", "message": "Postal code must be valid" }
  ]
}
```

```json
// Conflito — 409 (POST ou PUT com CNPJ duplicado) (ProblemDetails)
{
  "title": "Conflict",
  "type": "CNPJAlreadyExists",
  "detail": "An accountancy with the same CNPJ already exists",
  "status": 409,
  "errors": null
}
```

```json
// Não encontrado — 404 (GET /{id} inexistente) (ProblemDetails via HandleFailure)
{
  "title": "Not Found",
  "type": "AccountancyNotFound",
  "detail": "Accountancy 7c9b8f1a-4d2e-4a6b-9c3f-1e2d5a7b8c9e not found",
  "status": 404,
  "errors": null
}
```

```json
// Proibido — 403 (GET /me sem claim accountancy_id) (ProblemDetails via HandleFailure)
{
  "title": "Forbidden",
  "type": "AccountancyClaimMissing",
  "detail": "User does not have an associated accountancy.",
  "status": 403,
  "errors": null
}
```

```json
// Proibido — 403 (via ExceptionHandlingMiddleware, policy ASP.NET)
{
  "error": "Forbidden",
  "message": "You do not have permission to access this resource."
}
```

```json
// Não autorizado — 401 (via ExceptionHandlingMiddleware)
{
  "error": "Unauthorized",
  "message": "Authentication is required to access this resource."
}
```

```json
// Não encontrado — 404 (via ExceptionHandlingMiddleware, NotFoundException)
{
  "error": "NotFound",
  "message": "<mensagem específica da exceção>"
}
```

```json
// Erro interno — 500 (via ExceptionHandlingMiddleware)
{
  "error": "InternalServerError",
  "message": "An unexpected error occurred."
}
```

**Regra de interpretação no cliente**:

- Se o body tem `title` + `type` + `detail` + `status` → `ProblemDetails`
  (falha retornada pelo pipeline `Result`/`HandleFailure`). Use `type`
  como código legível e `errors[]` quando presente para feedback de
  formulário.
- Se o body tem `error` + `message` → envelope simplificado do
  `ExceptionHandlingMiddleware` (exceção de auth/auth ou exceção não
  tratada). Use `error` como código e `message` como descrição.

---

## 16. Gaps, limitações e decisões abertas

Gaps identificados entre `logic.md` aprovado e o código implementado:

1. **Validação de UF não verifica lista de siglas reais.** O validator
   aceita qualquer string de até 2 caracteres. Fica a cargo do frontend
   ofertar um dropdown com as 27 UFs válidas para reduzir entrada suja.

2. **Ordenação fixa sem opção de reversão.** `LegalName` ascendente é a
   única ordem possível. `logic.md` não exige reversão, mas é útil
   explicitar para o frontend não oferecer controles de ordenação.

Limitações por decisão consciente (não são gaps):

- Não existe exclusão ou ciclo de vida. Intencional.
- Não existe edição self-service pelo `AccountancyAdmin`. Intencional.
- Não existe busca textual ou filtro. Intencional nesta versão.
- Não existe identidade visual customizável. Intencional.
- `PUT` é replace total; não há `PATCH` parcial. Intencional; o
  frontend carrega o estado atual antes de editar.
- Soft-delete flag `IsDeleted` existe na projeção mas nenhum endpoint
  altera o valor. Intencional; mantém o read model preparado para
  evolução futura.
- Controle de concorrência otimista inexistente; "o último `save`
  vence" é decisão aceita em `logic.md` §5.

Nenhuma decisão aberta dependente do frontend para esta feature além
das listadas acima.
