# Command — Nova Feature

## Contexto

Workflow completo de entrada de feature no ecossistema Societiza. Desde a ideia
até a distribuição para os repos de execução.

## Pré-condições

- [ ] Problema identificado e ICP claro
- [ ] Acesso ao `docs/business-context.md` e `docs/planning/feature-map.md`

## Inputs obrigatórios

- `feature-name`: nome da feature em kebab-case
- `problem-description`: descrição do problema que a feature resolve

## Passos

1. Ler `docs/business-context.md` para entender o contexto do produto
2. Ler `docs/planning/feature-map.md` para entender dependências e integrações
3. Usar a skill `feature-design` para gerar a spec de produto
4. Salvar a spec em `features/specs/{feature-name}.md`
5. Apresentar a spec ao humano para aprovação
6. Após aprovação: gerar contrato OpenAPI em `contracts/api/{feature-name}.yaml`
7. Executar `impact-map` para mapear impacto cross-repo
8. Distribuir: indicar o que cada repo de execução precisa implementar

## Output esperado

- Spec aprovada em `features/specs/{feature-name}.md`
- Contrato em `contracts/api/{feature-name}.yaml`
- Documento de impacto com checklist por repo

## Skills requeridas

- `.claude/skills/feature-design/SKILL.md`
