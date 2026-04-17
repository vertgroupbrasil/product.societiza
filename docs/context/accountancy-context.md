# Accountancy — Unidade Organizacional

> Documento de contexto. Descreve a Accountancy como entidade multi-tenant atual da Societiza.
> Atualizado em 08/04/2026.

---

## O que é uma Accountancy

A Accountancy é a unidade organizacional que representa um escritório de
contabilidade dentro da Societiza. É o contorno de isolamento do sistema —
todos os processos, templates e configurações de equipe pertencem a um
escritório específico e são visíveis apenas por ele.

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

A Accountancy é o contorno de isolamento de todos os dados da plataforma.
Nenhum dado de uma contabilidade é visível por outra — processos, templates,
convites e membros pertencem exclusivamente ao seu escritório.

O societário opera sempre dentro do contexto de uma Accountancy ativa: os
processos visíveis no board, os templates disponíveis e os membros da equipe
pertencem à contabilidade da sessão.

---

## Estado atual

A Accountancy é a estrutura vigente no sistema. Reestruturações futuras (como
modelo de auto-criação pelo contador, planos de assinatura e suporte a múltiplas
contabilidades por usuário) estão sendo planejadas na spec
`specs/accountancy-offices/spec.md`, ainda aguardando aprovação.