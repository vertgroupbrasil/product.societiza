# TEMPLATE — Spec de Produto

Este arquivo é o modelo canônico para specs de produto da Societiza.

Ao criar uma spec, salvar como `specs/[domínio]-[conceito]/spec.md`.

> Para contexto completo antes de escrever, ler primeiro:
> - `docs/business-context.md` — índice de contexto do produto
> - `docs/product/product-vision.md` — visão, ICP e valores do produto
> - `docs/planning/feature-map.md` — dependências e integrações entre features

---

## Por que este modelo existe

Uma spec de produto existe para responder uma única pergunta antes que qualquer
linha de código seja escrita: **isso deve existir e, se sim, o que exatamente
deve ser?**

Sem uma spec clara, a engenharia implementa o que entende, não o que o produto
precisa. Revisões chegam tarde, custam mais, e criam atrito entre times.

Uma spec bem escrita elimina esse atrito na origem. Ela estabelece o contrato
de produto que orienta o mapa de impacto cross-repo, os contratos de API e a
implementação nos repos de execução.

---

## O que uma spec não é

- Não é um plano técnico
- Não é um guia de implementação
- Não é uma decisão de arquitetura
- Não é uma lista de endpoints
- Não é um schema de banco de dados
- Não é um cronograma de sprint

Uma pessoa não-técnica deve conseguir ler a spec e entender completamente a
feature. Se não conseguir, a spec está errada.

---

# Template: Spec de Produto

## Frontmatter obrigatório

Toda spec deve começar com um bloco YAML frontmatter. Os campos obrigatórios são:

```yaml
---
status: draft
domain: societario
created_at: 2026-04-11
---
```

Adicionar `approved_at` somente quando o humano aprovar explicitamente:

```yaml
---
status: approved
domain: societario
created_at: 2026-04-11
approved_at: 2026-04-11
---
```

**Valores possíveis de `status`:**

| Status | Significado |
|---|---|
| `draft` | Spec em elaboração, ainda não aprovada |
| `approved` | Spec aprovada pelo humano, pronta para implementação |
| `in-progress` | Spec aprovada e com implementação em andamento |
| `done` | Spec implementada com sucesso |
| `archived` | Spec descartada, não deve ser implementada |

**Valores possíveis de `domain`:** `societario`, `accountancy`, `dashboard`, `identity`

O status `approved` só é atribuído após aprovação explícita do humano responsável.
Nunca marcar como aprovada sem confirmação direta.

## Cabeçalho

```
# [Nome legível da feature]
```

---

## 1. Visão da feature

Descreva em um a três parágrafos, na linguagem do usuário:

- O que é a feature
- Como o usuário a percebe e a experimenta
- O que ela permite que ele faça que não conseguia antes

Sem linguagem técnica. Sem mencionar stack, endpoints ou implementação.

O objetivo desta seção é que qualquer pessoa — técnica ou não — consiga ler
e entender imediatamente o valor da feature.

---

## 2. Problema que resolve

Descreva o problema específico e real que esta feature elimina:

- Qual é o problema concreto no dia a dia do usuário
- Com que frequência acontece
- O que o usuário faz hoje para contornar (workaround atual)
- Por que esse workaround é insuficiente

Sem problema específico, não há feature justificada. Evitar linguagem vaga
como "melhora a experiência" ou "facilita o uso". Ir direto ao ponto de dor.

---

## 3. Para quem é

Identifique quem se beneficia:

- **Beneficiário direto** — quem usa ou vê a feature
- **Beneficiário indireto** — quem ganha sem interagir com ela diretamente
- Perfil do usuário que interage com a feature
- Em qual contexto ou momento da jornada a feature entra

Se a feature tem um beneficiário que nunca usa a Societiza (ex: cliente do
contador), deixar isso explícito. A interface que esse beneficiário vê precisa
ser descrita com o nível correto de simplicidade.

---

## 4. Como melhora a experiência atual

Descreva o contraste entre antes e depois:

**Antes:**
O que o usuário faz hoje. Quantas etapas. Qual o atrito. Qual o custo de tempo
ou atenção. Qual é o pior momento do fluxo atual.

**Depois:**
O que muda com a feature. Como o fluxo diário fica diferente. O que o usuário
para de precisar fazer. O que passa a acontecer de forma automática ou mais
simples.

Essa seção é o argumento de valor mais direto da spec. Se o "depois" não for
significativamente melhor que o "antes", a feature precisa ser repensada.

---

## 5. Como se conecta com o produto existente

Documente o encaixe da feature no ecossistema:

**Dependências obrigatórias:**
Liste as features ou módulos existentes dos quais esta feature depende para
funcionar. Se depende de dados de outro módulo, diga qual e por quê.

**O que esta feature habilita no futuro:**
Quais features futuras ficam possíveis ou mais fáceis depois que esta existir.
Qual é a posição desta feature no roadmap.

**Posição no produto:**
É uma feature core, complementar ou derivada? Ela fortalece o núcleo ou
adiciona uma camada periférica?

Sem essa seção, a engenharia implementa a feature de forma isolada, sem saber
o que ela está construindo para o futuro.

---

## 6. O que não é (escopo negativo)

Liste explicitamente o que está fora do escopo desta versão.

Cada item deve começar com **"Não é"** ou **"Não inclui"** e ter uma justificativa
breve. Evitar listas vazias de conteúdo. O escopo negativo serve para:

- Prevenir desvios durante a implementação
- Deixar claro o que é uma próxima versão versus o que está sendo construído agora
- Forçar decisões explícitas sobre o que ficou de fora

Se não há nada relevante fora do escopo, essa é uma feature com escopo
indefinido — revisar antes de aprovar.

---

## 7. Custos adicionais

> **Item bloqueante** — se houver qualquer custo adicional de infraestrutura,
> a feature não pode avançar para implementação sem aprovação explícita do
> **head de infraestrutura** e deliberação conjunta dos **membros do conselho da Societiza**. 
> Nenhum custo adicional de infraestrutura é aprovado sem passar por esses dois níveis de aprovação.

Identifique dependências que criam custo fora do código:

- Serviços externos (APIs de terceiros, provedores de e-mail, WhatsApp, etc.)
- Infraestrutura adicional (armazenamento, filas, etc.)
- Integrações com APIs governamentais
- Implicações para precificação ou planos de assinatura

Se não houver nenhum custo externo identificado, escrever explicitamente:
**"Nenhum custo externo identificado."**

Nunca deixar esta seção em branco.

---

## 8. Decisões tomadas

Tabela de decisões de produto que foram fechadas durante o processo de spec.
Cada linha representa uma decisão que não precisa mais ser discutida.

| Ponto | Decisão |
|---|---|
| [aspecto em questão] | [decisão tomada e justificativa breve] |

Esta seção cresce ao longo das iterações da spec. Quando uma pergunta aberta
(seção 9) é respondida, ela sai de lá e entra aqui como decisão fechada.

---

## 9. Perguntas abertas

Liste decisões não resolvidas, informações faltantes e incertezas conhecidas:

- O que ainda precisa ser decidido antes ou durante a implementação
- O que depende de informação externa (usuário, time, terceiro)
- O que está em discussão e ainda não tem consenso

Se não houver nenhuma pergunta aberta, escrever:
**"Nenhuma. Todas as decisões de produto estão fechadas."**

Nunca deixar esta seção em branco. Uma spec sem perguntas abertas declaradas
é uma spec que escondeu suas incertezas.

---

## Checagem antes de marcar como aprovada

1. Uma pessoa não-técnica consegue ler e entender completamente a feature?
2. O problema (seção 2) é específico o suficiente para validar com um usuário real?
3. O "depois" (seção 4) é significativamente melhor que o "antes"?
4. O escopo negativo (seção 6) tem pelo menos um item relevante?
5. Os custos externos (seção 7) estão identificados ou explicitamente ausentes?
6. Todas as decisões tomadas estão registradas na seção 8?
7. As perguntas abertas (seção 9) estão declaradas honestamente?

Se qualquer resposta for não, revisar antes de submeter para aprovação.

---

## Prompt para IA gerar esta spec

```
Você vai criar uma spec de produto para a Societiza.

Importante:
- não escreva código, schemas, endpoints ou decisões de arquitetura;
- escreva na linguagem do usuário, não na linguagem do engenheiro;
- o problema deve ser específico e real — sem linguagem vaga;
- o escopo negativo deve ter ao menos um item relevante;
- as decisões tomadas devem ser registradas como tabela;
- as perguntas abertas devem ser honestas — não esconder incertezas;
- o status inicial no frontmatter é sempre "draft" até aprovação explícita do humano.

Leia antes de escrever:
- docs/business-context.md
- docs/planning/feature-map.md
- docs/product/product-vision.md

Gere a spec seguindo exatamente a estrutura de templates/specification.md.

Salve em specs/[domínio]-[conceito]/spec.md.
Domínios válidos: societario, accountancy, dashboard, identity.
Conceito: máximo 2 palavras em kebab-case.
```