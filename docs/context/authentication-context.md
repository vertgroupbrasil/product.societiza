# Autenticação — Estado Atual e Gaps

## O que está implementado

O módulo Identity cobre hoje dois fluxos distintos:

**Gestão de convites:** envio de convite, revogação e aceite. Um job agendado
verifica periodicamente os convites pendentes e marca como expirados os que
ultrapassaram o prazo de 72h.

**Autenticação:** login com e-mail e senha, logout, refresh de token e reset
de senha.

## Gaps atuais

**Endpoints desprotegidos:** os endpoints da API ainda não exigem autenticação.
O fluxo de autenticação está implementado, mas o middleware de proteção das
rotas ainda não foi aplicado. Qualquer requisição é aceita sem token válido.

**Fluxo de acesso em reestruturação:** o fluxo de entrada e navegação dos
usuários na plataforma está sendo reestruturado em função do plano freemium
planejado para o MVP. Enquanto essa reestruturação não estiver consolidada,
evite tratar o fluxo atual como definitivo.

## Direção futura

Autenticação via Google pode ser incorporada como método adicional — não como
substituto. As contabilidades utilizam provedores de e-mail variados (Google
Workspace, Microsoft Outlook, entre outros), o que torna inviável exigir Google
como único meio de acesso. Qualquer expansão de métodos de autenticação deve
preservar e-mail e senha como opção base.