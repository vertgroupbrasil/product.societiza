# Regra — Spec Antes da Implementação

Escopo: orchestration

## Nunca fazer

- Iniciar implementação em qualquer repo sem uma spec aprovada
- Pular a aprovação humana da spec
- Tratar um resumo de conversa como spec
- Gerar o logic doc sem spec aprovada
- Gerar o integration doc antes do backend ter implementado a feature
- Distribuir para os repos de execução sem logic doc revisado por FE + BE

## Sempre fazer

- Gerar uma spec usando a skill feature-design
- Salvar a spec em `specs/[domain]-[concept]/spec.md` — nomes de pasta e arquivo sempre em inglês (exceção: "societario" pode ser usado em português)
- Obter aprovação humana explícita antes de qualquer artefato subsequente
- Gerar o logic doc usando a skill logic-design após aprovação da spec
- Aguardar revisão conjunta de FE + BE no logic doc antes de distribuir

## Artefatos e seus momentos

Cada artefato tem um momento específico. Não antecipar nem pular etapas.

### 1. Spec (`spec.md`) — antes da implementação
- Gerada pela skill `feature-design`
- Salva em `specs/[domain]-[concept]/spec.md` — nomes em inglês
- Exige aprovação humana explícita antes de avançar

### 2. Logic doc (`logic.md`) — após aprovação da spec, antes da implementação
- Gerado pela skill `logic-design`
- Salvo em `specs/[domain]-[concept]/logic.md` — nomes em inglês
- Define fluxos por perfil, estados, regras de negócio e pontos de integração
- Exige revisão conjunta de tech leads de frontend e backend antes de avançar
- É o contrato compartilhado que permite FE e BE trabalharem em paralelo

### 3. Integration doc (`integration.md`) — após o backend implementar
- Gerado pelo agente `tech-lead.md` de `api.societiza`, cruzando o código implementado com a spec aprovada e o logic doc
- Salvo em `specs/[domain]-[concept]/integration.md` — nomes em inglês
- Exige aprovação do tech lead de backend antes de ser considerado canônico
- Consumido por `web.societiza` via MCP do GitHub ao integrar a feature

## Razão

A spec define o que construir. O logic doc define como o produto se comporta
— o contrato entre FE e BE que elimina ruído durante a implementação. O
integration doc fecha o ciclo documentando o que foi realmente construído.
Cada artefato depende do anterior — pular etapas desalinha produto, backend
e frontend.