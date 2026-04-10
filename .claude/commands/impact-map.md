# Command — Mapa de Impacto Cross-Repo

## Contexto

Antes de implementar uma feature em qualquer repo de execução, mapear o que
muda em cada repo e quais são os riscos de integração.

## Pré-condições

- [ ] Spec da feature aprovada em `specs/[domínio]-[conceito]/spec.md`

## Estrutura de specs

Specs vivem em `specs/[domínio]-[conceito]/` com arquivos fixos:
- `spec.md` — spec de produto (input deste command)
- `impact-map.md` — output deste command

Domínios válidos: `societario`, `accountancy`, `dashboard`, `identity`.

## Inputs obrigatórios

- `feature-name`: nome da feature no formato `[domínio]-[conceito]`
- `spec-path`: `specs/{feature-name}/spec.md`

## Passos

1. Ler a spec da feature
2. Ler `docs/planning/feature-map.md` para identificar dependências
3. Identificar módulos do backend afetados (aggregates, events, endpoints)
4. Identificar features do frontend afetadas (schemas, hooks, components)
5. Listar breaking changes e riscos de integração
6. Produzir documento de impacto e salvar em `specs/{feature-name}/impact-map.md`

## Output esperado

Documento de impacto contendo:
- Módulos backend afetados e o que precisa mudar
- Features frontend afetadas e o que precisa mudar
- Breaking changes identificados
- Checklist de validação por repo
- Ordem sugerida de implementação (backend primeiro, frontend depois, ou paralelo)

## Skills requeridas

- `.claude/skills/feature-design/SKILL.md` (para contexto de produto)