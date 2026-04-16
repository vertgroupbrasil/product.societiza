# TEMPLATE — Logic Doc

Este arquivo é o modelo canônico para docs de lógica de negócio da Societiza.

Ao criar um logic doc, salvar como `specs/[domínio]-[conceito]/logic.md`.

> Para escrever corretamente, ler primeiro:
> - `specs/[domínio]-[conceito]/spec.md` — a spec aprovada da feature
> - Context doc do domínio correspondente (`docs/context/`)
> - `docs/planning/feature-map.md` — invariantes e dependências cross-feature

---

## Por que este documento existe

A spec de produto define o quê e o porquê. O logic doc define o como — em
linguagem de negócio, não de engenharia.

Sem um logic doc claro, frontend e backend interpretam a spec de formas
diferentes. Ambos esperam que o outro resolva ambiguidades durante a
implementação. O resultado é ruído, retrabalho e dependências desnecessárias
entre os dois lados.

Um logic doc bem escrito elimina esse atrito na origem: frontend e backend
podem construir em paralelo porque o contrato entre eles está explícito antes
de qualquer linha de código.

---

## O que um logic doc não é

- Não é um contrato de API
- Não é um schema de banco de dados
- Não é uma decisão de arquitetura
- Não é uma lista de endpoints
- Não usa nomenclatura de domínio técnico (Aggregate, Entity, ValueObject, Enum)
- Não é um plano de implementação

O logic doc fala em linguagem de produto e de comportamento — estados que um
usuário reconhece, regras que um contador entenderia, fluxos que fazem sentido
para quem usa o sistema.

---

# Template: Logic Doc

## Frontmatter obrigatório

```yaml
---
status: draft
spec: [domain]-[concept]
created_at: YYYY-MM-DD
---
```

Adicionar `reviewed_at` e mudar `status` para `reviewed` somente após revisão
conjunta explícita de tech leads de frontend e backend.

**Valores possíveis de `status`:**

| Status | Significado |
|---|---|
| `draft` | Em elaboração, ainda não revisado |
| `reviewed` | Revisado e aprovado por FE + BE em conjunto |

## Cabeçalho

```
# [Nome legível da feature] — Lógica de Negócio
```

---

## 1. Fluxo por perfil

Para cada perfil de usuário que toca esta feature, descrever o fluxo completo
do ponto de vista dele:

- O que ele pode iniciar nesta feature
- Cada passo do fluxo: o que faz, o que vê, o que acontece como resultado
- O que acontece quando tenta uma ação que não é permitida para ele

**Perfis da plataforma:**
- `SystemAdmin` — operadores da própria Societiza
- `AccountingAdmin` — administrador do escritório de contabilidade
- `AccountingEmployee` — funcionário operacional do escritório

Cobrir apenas os perfis que efetivamente interagem com a feature. Se um perfil
não tem interação com a feature, declarar explicitamente: "[Perfil] não interage
com esta feature."

Formato: prosa curta + lista de passos. Não usar tabela nesta seção.

---

## 2. Estados e transições

Para cada conceito principal da feature:

**Estados:** quais estados existem, em linguagem de produto (não de domínio
técnico). Estado é algo que o usuário reconhece — não um valor de enum.

**Transições:**
- O que causa cada mudança de estado (ação do usuário, evento do sistema, tempo)
- O que bloqueia cada transição (pré-condição não atendida)

Sem nomenclatura técnica: sem Aggregate, Entity, ValueObject, Repository,
Domain Event. Se precisar nomear um conceito, use o nome que o usuário usaria.

---

## 3. Regras de negócio

Lista objetiva de regras. Uma regra por linha.

Formato: `- [Condição ou contexto] → [Consequência ou restrição]`

Cobre:
- Pré-condições obrigatórias para cada ação principal
- Invariantes que nunca podem ser quebradas, independente da ação
- Restrições que cruzam features (envolvendo `accountancyId`, `templateId`, etc.)
- Limites e tetos (se aplicável)

Se não houver regras de negócio além do descrito nos fluxos: escrever
explicitamente "Nenhuma regra adicional além das descritas nos fluxos acima."

Nunca deixar esta seção em branco.

---

## 4. Pontos de integração

O que frontend e backend precisam acordar para construir em paralelo.

Não é contrato de API. Não são endpoints. É o que cada lado precisa saber
do outro para que as partes se conectem corretamente.

```
Frontend precisa saber:
  - [dado, estado ou comportamento que o backend precisa expor]
  - [...]

Backend precisa garantir:
  - [comportamento, dado ou invariante que o frontend depende]
  - [...]
```

Se não houver pontos de integração não-óbvios: escrever "Nenhum ponto de
integração adicional identificado."

Nunca deixar esta seção em branco.

---

## 5. Casos de borda

Situações que o fluxo principal não cobre explicitamente. Cada uma nomeada
e com resolução clara.

Formato por caso:
- **[Nome do caso]:** descrição da situação + decisão fechada

Se o caso não tiver decisão fechada: marcar como `[pergunta aberta]` e
descrever o que precisa ser decidido antes da implementação.

Se não houver casos de borda identificados: escrever "Nenhum caso de borda
identificado além dos cobertos pelos fluxos e regras acima."

Nunca omitir esta seção.

---

## Checklist antes de marcar como `reviewed`

A revisão deve ser feita conjuntamente pelos tech leads de frontend e backend.

1. Todos os perfis relevantes estão cobertos com fluxo completo?
2. Cada ação proibida tem comportamento descrito (o que o sistema faz)?
3. Os estados usam linguagem de produto — não nomenclatura de domínio técnico?
4. Os pontos de integração são claros o suficiente para FE e BE construírem em paralelo sem esperar um pelo outro?
5. Nenhuma regra de negócio está implícita — todas estão explícitas?
6. Os casos de borda não-óbvios estão nomeados e com decisão ou marcados como pergunta aberta?

Se qualquer resposta for não, revisar antes de marcar como `reviewed`.