# Identity — Autenticação, Convites e Gestão de Usuário

## O módulo Identity como perímetro de identidade

O módulo Identity é o perímetro de identidade da Societiza. Ele cobre três
responsabilidades centrais: autenticar usuários, gerenciar o ciclo de vida de
convites para membros de uma contabilidade, e estabelecer a base para gestão
de usuário futura.

O multi-tenancy do módulo é organizado por Accountancy: convites e membros
pertencem a uma contabilidade específica, identificada via `AccountancyId`. No
estado atual, um usuário pertence a uma única contabilidade.

---

## O que está implementado

### Convites

| Endpoint | Descrição |
|---|---|
| `InviteUser` | Cria e envia convite para um e-mail |
| `AcceptInvite` | Aceita convite e cria a conta do membro |
| `RevokeInvite` | Revoga um convite pendente |
| `ResendInvite` | Reenvia o e-mail de um convite pendente |

**Job:** `ExpireInvitesJob` — verifica periodicamente os convites pendentes e
marca como expirados os que ultrapassaram o prazo de 72h.

**E-mail:** `SendInviteEmail` — disparado ao criar ou reenviar um convite.

### Autenticação

| Endpoint | Descrição |
|---|---|
| `Login` | Autenticação com e-mail e senha; retorna JWT + RefreshToken |
| `Logout` | Invalida a sessão ativa |
| `Refresh` | Renova o JWT a partir de um RefreshToken válido |
| `ForgotPassword` | Envia e-mail com link de reset de senha |
| `ResetPassword` | Redefine a senha a partir do token recebido por e-mail |

**E-mail:** `SendPasswordResetEmail` — disparado ao solicitar reset de senha.

### Entidades de domínio

- `User`
- `UserInvite`
- `RefreshToken`
- `PasswordResetToken`

### Projeções

- `ProjectionUserEventHandler`
- `ProjectionUserInviteEventHandler`
- `ProjectionRefreshTokenEventHandler`
- `ProjectionPasswordResetTokenEventHandler`

---

## O que NÃO está implementado

- Update de perfil de usuário
- Delete de usuário
- Listagem de usuários da contabilidade
- Gestão granular de permissões

---

## Gaps atuais

**Endpoints desprotegidos:** os endpoints da API ainda não exigem autenticação.
O fluxo de autenticação está implementado, mas o middleware de proteção das
rotas ainda não foi aplicado. Qualquer requisição é aceita sem token válido.

Após a aplicação do middleware, toda requisição a recursos scoped à Accountancy
deve validar: (1) token JWT válido, (2) que o usuário é membro do `AccountancyId`
presente na rota ou no contexto da sessão.

---

## Direção futura

Autenticação via Google pode ser incorporada como método adicional — não como
substituto. As contabilidades utilizam provedores de e-mail variados (Google
Workspace, Microsoft Outlook, entre outros), o que torna inviável exigir Google
como único meio de acesso. Qualquer expansão de métodos de autenticação deve
preservar e-mail e senha como opção base.