---
name: logic-design
description: >
  Skill de lógica de negócio para a Societiza. Complementa a feature-design
  com fluxos por perfil de usuário, estados e transições, regras de negócio
  e pontos de integração entre frontend e backend. Invocada após aprovação
  da spec de produto. Produz o contrato compartilhado que permite FE e BE
  trabalharem em paralelo.
user-invocable: true
argument-hint: "[domínio]-[conceito] (ex: societario-workflow-process)"
---

# Logic Design

A spec de produto define o quê e o porquê. Esta skill define o como — em
linguagem de negócio, não de engenharia.

Antes de uma única linha de código ser escrita, esta skill responde à pergunta
que mais importa para a engenharia: **como exatamente esta feature se comporta
para cada perfil de usuário, em cada estado, seguindo cada regra?**

---

## Leitura obrigatória antes de pensar

Leia estes documentos por completo antes de responder.

1. `specs/[domain]-[concept]/spec.md` — a spec aprovada da feature (passada como argumento)
2. Context doc do domínio da feature:
   - `societario` → `docs/context/societario-context.md`
   - `accountancy` → `docs/context/accountancy-context.md`
   - `identity` → `docs/context/identity-context.md`
   - `dashboard` → `docs/context/societario-context.md` + `docs/planning/dashboard.md`
3. `docs/planning/feature-map.md` — invariantes e dependências cross-feature

Não prossiga sem ler os três (ou quatro, no caso do dashboard).

---

## O que esta skill faz

Produz um documento de lógica de negócio para uma feature já especificada e
aprovada. Não produz contratos de API, schemas, decisões de arquitetura ou
qualquer artefato de engenharia.

O output fala em linguagem de produto e de comportamento — estados que o
usuário reconhece, regras que o contador entenderia, fluxos que fazem sentido
para quem usa o sistema.

---

## Processo de pensamento (executar internamente antes de escrever)

Esta skill parte sempre da spec aprovada. Não reinterpreta o produto — aprofunda
a lógica do que já foi decidido.

**1. Quais perfis tocam esta feature?**
Liste explicitamente cada perfil que interage com a feature:
`SystemAdmin`, `AccountingAdmin`, `AccountingEmployee`.
Para cada um: o que pode iniciar? O que vê? O que não pode fazer?
Se um perfil não tem interação, declare isso explicitamente.

**2. Qual é a sequência de estados?**
Para cada conceito principal da feature: quais estados existem em linguagem
de produto (não de domínio técnico)? O que causa cada transição? O que a
bloqueia? Estado é algo que o usuário reconhece — não um valor de enum ou
uma propriedade de aggregate.

**3. Quais regras de negócio são inegociáveis?**
Pré-condições para cada ação. Invariantes que nunca podem ser quebradas.
Constraints que cruzam features (envolvendo `accountancyId`, `templateId`,
`processId`, etc.). Limites e tetos, se existirem.

**4. O que FE e BE precisam acordar antes de construir?**
Quais dados o frontend precisa conhecer para construir cada estado visual?
O que o backend precisa garantir para que o frontend funcione corretamente?
Esses são os pontos de integração — não contratos de API, mas acordos sobre
o que cada lado produz e consome. O objetivo é que os dois lados possam
construir em paralelo sem esperar um pelo outro.

**5. O que o fluxo principal não cobre?**
Situações não-óbvias que surgem quando os estados, perfis e regras se
combinam de formas inesperadas. Cada uma com decisão fechada ou marcada
explicitamente como pergunta aberta.

---

## Formato do output

Gere um documento markdown e salve em:

```
specs/[domain]-[concept]/logic.md
```

Usar o mesmo `[domain]-[concept]` da spec de produto correspondente.

### Frontmatter obrigatório

```yaml
---
status: draft
spec: [domain]-[concept]
created_at: YYYY-MM-DD
---
```

Nunca marcar como `reviewed` sem confirmação explícita de revisão conjunta
pelos tech leads de frontend e backend.

### Seções do documento de output

#### 1. Fluxo por perfil

Para cada perfil que toca a feature: fluxo completo do ponto de vista dele.

- O que ele pode iniciar
- Cada passo: o que faz, o que vê, o que acontece como resultado
- O que acontece quando tenta uma ação proibida para ele

Formato: prosa curta + lista de passos. Não usar tabela nesta seção.
Cobrir apenas perfis que efetivamente interagem com a feature.

#### 2. Estados e transições

Para cada conceito principal da feature:
- Quais estados existem (em linguagem de produto)
- O que causa cada transição (ação do usuário, evento do sistema, tempo)
- O que bloqueia cada transição

Sem nomenclatura técnica de domínio. Sem Aggregate, Entity, ValueObject,
Repository, Domain Event. Use o nome que o usuário usaria.

#### 3. Regras de negócio

Lista objetiva. Uma regra por linha.
Formato: `- [Condição ou contexto] → [Consequência ou restrição]`

Cobre pré-condições, invariantes, constraints cross-feature e limites.

#### 4. Pontos de integração

Dois blocos fixos:

```
Frontend precisa saber:
  - [dado, estado ou comportamento que o backend precisa expor]

Backend precisa garantir:
  - [comportamento, dado ou invariante que o frontend depende]
```

Sem contratos de API. Sem endpoints. Sem schemas.

#### 5. Casos de borda

Situações que os fluxos e regras não cobrem explicitamente. Cada uma com:
- Nome do caso
- Descrição da situação
- Decisão fechada OU marcado como `[pergunta aberta]` com o que precisa ser decidido

---

## Proibições absolutas

Esta skill nunca deve incluir no output:

- Nomenclatura de domínio técnico (Aggregate, Entity, ValueObject, Enum, Repository)
- Contratos de API, endpoints, métodos HTTP
- Schemas de banco de dados ou estruturas de persistência
- Decisões de arquitetura
- Código de qualquer tipo
- Stack tecnológico (linguagem, framework, biblioteca)
- Nomes de componentes ou caminhos de arquivo
- Cronogramas ou atribuições de sprint

---

## Checagem final antes de salvar

1. Todos os perfis relevantes têm fluxo completo descrito?
2. Cada ação proibida tem comportamento do sistema descrito?
3. Os estados usam linguagem de produto — sem nomenclatura técnica de domínio?
4. Os pontos de integração permitem que FE e BE construam em paralelo sem esperar um pelo outro?
5. Nenhuma regra de negócio está implícita — todas estão explícitas?
6. Os casos de borda não-óbvios estão nomeados com decisão ou pergunta aberta?

Se qualquer resposta for não, revisar antes de salvar.

---

## Quem revisa este documento

O `logic.md` é revisado conjuntamente pelos tech leads de frontend e backend
— não pelo time de produto. A revisão valida que o contrato é implementável
e suficiente para que os dois lados trabalhem em paralelo.

Após revisão, mudar `status` de `draft` para `reviewed` e adicionar
`reviewed_at: YYYY-MM-DD` no frontmatter.