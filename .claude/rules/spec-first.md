# Regra — Spec Antes da Implementação

Escopo: orchestration

## Nunca fazer

- Iniciar implementação em qualquer repo sem uma spec aprovada
- Pular a aprovação humana da spec
- Tratar um resumo de conversa como spec
- Gerar o integration doc antes do backend ter implementado a feature

## Sempre fazer

- Gerar uma spec usando a skill feature-design
- Salvar a spec em `specs/[domain]-[concept]/spec.md` — nomes de pasta e arquivo sempre em inglês (exceção: "societario" pode ser usado em português)
- Obter aprovação humana explícita antes de qualquer artefato subsequente

## Artefatos e seus momentos

Cada artefato tem um momento específico. Não antecipar nem pular etapas.

### 1. Spec (`spec.md`) — antes da implementação
- Gerada pela skill `feature-design`
- Salva em `specs/[domain]-[concept]/spec.md` — nomes em inglês
- Exige aprovação humana explícita antes de avançar

### 2. Integration doc (`integration.md`) — após o backend implementar
- Gerado pelo agente de produto lendo o código de `api.societiza` via MCP + cruzando com a spec aprovada
- Salvo em `specs/[domain]-[concept]/integration.md` — nomes em inglês
- Exige aprovação do backend (tech lead ou dev responsável) antes de ser considerado canônico
- Consumido por `web.societiza` via MCP do GitHub ao integrar a feature

## Razão

A spec é o ponto de controle de qualidade das decisões de produto. O
integration doc fecha o ciclo documentando o que foi realmente construído.
Cada artefato depende do anterior — pular etapas desalinha produto, backend
e frontend.