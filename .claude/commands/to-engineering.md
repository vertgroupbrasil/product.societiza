# Command — To Engineering

## Contexto

Workflow completo de entrada de feature no ecossistema Societiza. Desde a ideia
até a distribuição para os repos de execução.

Este command é o **caminho de execução** — use quando a decisão de produto já
foi tomada ou está prestes a ser e o humano quer o ciclo completo.

## Pré-condições

- [ ] Problema identificado e ICP claro

## Inputs obrigatórios

- `feature-name`: nome da feature em kebab-case
- `problem-description`: descrição do problema que a feature resolve

## Passos

1. Verificar se existe `features/specs/{feature-name}.md`.
   - **Se existe:** ler o arquivo e apresentar ao humano: "Encontrei uma spec
     existente para essa feature. É essa que seguimos?"
     - Humano confirma → ir direto para o GATE de aprovação (passo 3)
     - Humano nega → invocar `feature-design` para gerar nova spec
   - **Se não existe:** invocar a skill `feature-design` passando
     `feature-name` e `problem-description`. A skill faz toda a leitura de
     contexto internamente.
2. Spec gerada e salva em `features/specs/{feature-name}.md`
3. **GATE — Aprovação humana.** Apresentar a spec ao humano e aguardar
   aprovação explícita. Não avançar sem aprovação. Se o humano pedir mudanças,
   iterar na spec até aprovação.
4. Executar `impact-map` para mapear impacto cross-repo
5. Distribuir: indicar o que cada repo de execução recebe como insumo

## Output esperado

- Spec aprovada em `features/specs/{feature-name}.md`
- Documento de impacto com checklist por repo

## Skills requeridas

- `.claude/skills/feature-design/SKILL.md`