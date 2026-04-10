# TEMPLATE — Mapa de Impacto Cross-Repo

Este arquivo é o modelo canônico para geração do mapa de impacto de uma feature
após aprovação da spec.

## Quem gera

O agente de produto (`product.societiza`) gera este documento usando o command
`impact-map` imediatamente após a aprovação humana da spec. Exige aprovação
humana antes de ser distribuído para engenharia via `/to-engineering`.

## Onde salvar

Salvar como `specs/[domínio]-[conceito]/impact-map.md`.

---

# Impact Map — [Nome da Feature]

> Spec de origem: `specs/[domínio]-[conceito]/spec.md`
> Data: AAAA-MM-DD
> Status: `rascunho` | `aprovado`

---

## Resumo da feature

[Descrição em 2-3 frases do que a feature faz e por que existe, extraída da spec.]

---

## Repos impactados

| Repo | Tipo de impacto | Prioridade |
|------|----------------|------------|
| `api.societiza` | backend | alta / média / baixa |
| `web.societiza` | frontend | alta / média / baixa |

---

## `api.societiza` — O que precisa ser feito

### Domínio / módulo afetado

[Nome do módulo ou bounded context afetado.]

### Novos endpoints

| Método | Path | Finalidade |
|--------|------|-----------|
| | | |

### Alterações em endpoints existentes

[Descrever mudanças de contrato, novos campos, campos removidos ou comportamentos alterados.]

### Novos modelos / entidades

[Listar entidades ou value objects novos, se houver.]

### Migrations

[Indicar se há mudança de schema. Se sim, descrever o impacto.]

### Dependências internas

[Módulos ou serviços de `api.societiza` que esta feature consome ou afeta.]

---

## `web.societiza` — O que precisa ser feito

### Páginas / rotas afetadas

[Listar páginas ou rotas existentes que mudam, e novas páginas que serão criadas.]

### Novos componentes

[Listar componentes de orquestração e/ou UI necessários.]

### Endpoints a registrar

[Listar os endpoints que precisam ser adicionados em `src/routes/endpoints.ts`.]

### Queries e mutations

[Descrever as queries e mutations necessárias.]

### Estados de UX relevantes

[Loading, empty state, error state, estados de negócio que afetam a UI.]

---

## Dependências entre repos

[Descrever se o frontend pode começar antes do backend terminar, ou se há
bloqueio. Indicar o que pode ser feito em paralelo.]

---

## Gaps e decisões abertas

[O que ainda não está definido, o que precisa de validação, o que pode mudar
antes da implementação começar.]

---

## Ordem de execução sugerida

1. [Primeiro passo]
2. [Segundo passo]
3. [...]