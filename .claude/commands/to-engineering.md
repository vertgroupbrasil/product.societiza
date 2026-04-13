# Command — To Engineering

## Contexto

Workflow completo de entrada de feature no ecossistema Societiza. Desde a ideia
até a distribuição para os repos de execução.

Este command é o **caminho de execução** — use quando a decisão de produto já
foi tomada ou está prestes a ser e o humano quer o ciclo completo.

## Pré-condições

- [ ] Problema identificado e ICP claro

## Estrutura de specs

Specs vivem em `specs/[domínio]-[conceito]/` com um arquivo principal:
- `spec.md` — spec de produto

Domínios válidos: `societario`, `accountancy`, `dashboard`, `identity`.
Conceito: máximo 2 palavras em kebab-case.

Exemplo: `specs/societario-tracking/spec.md`

## Inputs obrigatórios

- `feature-name`: nome da feature no formato `[domínio]-[conceito]`
- `problem-description`: descrição do problema que a feature resolve

## Passos

1. Verificar se existe `specs/{feature-name}/spec.md`.
   - **Se existe:** ler o arquivo e apresentar ao humano: "Encontrei uma spec
     existente para essa feature. É essa que seguimos?"
     - Humano confirma → ir direto para o GATE de aprovação (passo 3)
     - Humano nega → invocar `feature-design` para gerar nova spec
   - **Se não existe:** invocar a skill `feature-design` passando
     `feature-name` e `problem-description`. A skill faz toda a leitura de
     contexto internamente.
2. Spec gerada e salva em `specs/{feature-name}/spec.md`
3. **GATE — Aprovação humana.** Apresentar a spec ao humano e aguardar
   aprovação explícita. Não avançar sem aprovação. Se o humano pedir mudanças,
   iterar na spec até aprovação.
4. Distribuir: indicar o que cada repo de execução recebe como insumo

## Output esperado

- Spec aprovada em `specs/{feature-name}/spec.md`

## Skills requeridas

- `.claude/skills/feature-design/SKILL.md`