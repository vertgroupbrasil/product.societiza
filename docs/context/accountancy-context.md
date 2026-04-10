# Accountancy — Unidade Organizacional

> Documento de contexto. Descreve a Accountancy como entidade multi-tenant atual da Societiza.
> Atualizado em 08/04/2026.

---

## O que é uma Accountancy

A Accountancy é a unidade organizacional que representa um escritório de
contabilidade dentro da Societiza. É o contorno multi-tenant do sistema —
todos os processos, templates e configurações de equipe pertencem a uma
Accountancy específica, identificada por `accountancyId`.

Cada Accountancy tem uma identidade que espelha o escritório de contabilidade
no mundo físico:

- CNPJ
- Nome legal
- Nome fantasia
- Endereço
- Cidade
- Estado
- CEP
- Telefone
- E-mail

---

## Quem cria e gerencia

A Accountancy é criada e gerenciada pela própria Societiza via SystemAdmin.
O contador acessa a plataforma dentro do contexto de uma Accountancy já
configurada — ele não cria nem configura a Accountancy diretamente.

---

## Papel no sistema

`accountancyId` é o discriminante central de toda separação de dados na
plataforma. Está presente em todo processo, template, convite e filtro.
Nenhum dado de uma Accountancy vaza para outra.

O societário opera sempre dentro do contexto de uma Accountancy ativa: os
processos visíveis no board, os templates disponíveis e os membros da equipe
são todos scoped ao `accountancyId` da sessão.

---

## Estado atual

A Accountancy é a estrutura vigente no sistema. Reestruturações futuras (como
modelo de auto-criação pelo contador, planos de assinatura e suporte a múltiplas
contabilidades por usuário) estão sendo planejadas na spec
`specs/accountancy-offices/spec.md`, ainda aguardando aprovação.