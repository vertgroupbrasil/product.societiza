# Identity — Autenticação, Convites e Gestão de Usuário

## O módulo Identity como perímetro de identidade

O módulo Identity é o perímetro de identidade da Societiza. Ele cobre três
responsabilidades centrais: autenticar usuários, gerenciar o ciclo de vida de
convites para membros de uma contabilidade, e estabelecer a base para gestão
de usuário futura.

Convites e membros pertencem a uma contabilidade específica. No estado atual,
um usuário pertence a uma única contabilidade.

---

## O que está implementado

### Convites

Um administrador convida um membro informando o e-mail e o perfil de acesso.
O convidado recebe um e-mail e aceita o convite, criando sua conta e passando
a integrar a equipe da contabilidade. Enquanto pendente, o convite pode ser
revogado ou reenviado pelo administrador. Convites não aceitos dentro do prazo
de 72 horas expiram automaticamente.

### Autenticação

O usuário acessa a plataforma com e-mail e senha. A sessão é mantida de forma
segura e pode ser encerrada explicitamente. Caso esqueça a senha, o usuário
pode solicitar um link de redefinição por e-mail.

---

## O que NÃO está implementado

- Atualização de perfil do usuário
- Exclusão de usuário
- Listagem de membros da contabilidade
- Gestão granular de permissões dentro da contabilidade

---

## Direção futura

Autenticação via Google pode ser incorporada como método adicional — não como
substituto. As contabilidades utilizam provedores de e-mail variados (Google
Workspace, Microsoft Outlook, entre outros), o que torna inviável exigir Google
como único meio de acesso. Qualquer expansão de métodos de autenticação deve
preservar e-mail e senha como opção base.
