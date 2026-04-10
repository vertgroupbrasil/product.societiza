# Regra — Fronteira Cross-Repo

Escopo: orchestration

## Nunca fazer

- Escrever código de aplicação (.cs, .tsx, .ts, .csproj, package.json)
- Criar componentes, hooks, services ou endpoints
- Alterar migrations de banco de dados ou configurações de infraestrutura
- Tomar decisões de implementação que pertencem aos repos de execução

## Sempre fazer

- Produzir specs de feature em `specs/[domínio]-[conceito]/spec.md`
- Referenciar repos de execução pelo nome (api.societiza, web.societiza)
- Mapear impacto cross-repo antes de distribuir o trabalho

## Razão

Este repo orquestra decisões de produto e as distribui. Ele não executa.
A implementação acontece em api.societiza e web.societiza.