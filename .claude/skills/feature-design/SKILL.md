---
name: feature-design
description: >
  Skill de planejamento de produto para a Societiza. Use para decidir se uma
  feature deve ser construída, definir o que ela deve ser, entender como se
  conecta ao produto, ou gerar uma spec de produto. Produto primeiro — sem
  código, sem arquitetura, sem stack.
user-invocable: true
argument-hint: "[nome da feature ou descrição do problema]"
---

# Feature Design

Essa skill pensa em features da mesma forma que Steve Jobs pensava em produtos.

Produto primeiro. Engenharia resolve o resto.

Antes de uma única linha de código ser escrita, antes de qualquer decisão de
arquitetura, antes de qualquer sprint ser planejado — essa skill responde à
pergunta que mais importa: **isso deve existir e, se sim, o que exatamente
deve ser?**

---

## Leitura obrigatória antes de pensar

Carregue estes documentos antes de responder. Leia-os por completo.

1. `docs/business-context.md` — índice de contexto do produto
2. `docs/planning/feature-map.md` — como features se conectam e o que cada uma habilita
3. `references/steve-jobs-mentality.md` — o framework de pensamento de produto

Não prossiga sem ler os três.

---

## O que essa skill faz

Essa skill produz um documento de especificação de produto para uma feature.
Ela não produz planos técnicos, guias de implementação ou decisões de
arquitetura.

Tudo no output é escrito na linguagem do usuário e do produto, não na
linguagem de engenheiros.

---

## Processo de pensamento (executar internamente antes de escrever)

Esta skill trata cada invocação como ponto de partida completo. Não pressuponha
que o agente ou humano que a invocou já fez análise de fit, dependências,
coerência de produto ou perguntas Steve Jobs. Toda análise começa aqui.

Antes de escrever o documento de output, pense:

**1. Qual é o problema real?**
Não o que a feature faz. Que problema ela resolve. Quem tem esse problema. Com
que frequência enfrentam. O que fazem hoje para contornar. Se esse é um
problema que vale resolver agora ou depois.

**2. Isso se encaixa no produto?**
Usando `docs/business-context.md` e `docs/planning/feature-map.md`: onde essa
feature se posiciona no produto? A qual horizonte pertence? Ela fortalece o
core ou adiciona ruído? De quais features ela depende? Quais features ela
habilita?

**3. Qual é a pergunta Steve Jobs?**
De `references/steve-jobs-mentality.md`: execute as 10 perguntas obrigatórias.
Se não conseguir responder pelo menos 8 com clareza, diga isso no documento
de output e marque a feature como precisando de mais definição antes de ser
aprovada.

**4. Qual é a versão mínima que importa?**
Qual é a menor versão dessa feature que genuinamente resolve o problema?
Qual é a versão que faria um usuário dizer "finalmente"? Não a visão completa
— qual é a menor versão com impacto real.

**5. Existem custos além de código?**
Essa feature exige serviços externos, APIs de terceiros, armazenamento,
provedores de email, integrações com WhatsApp, APIs governamentais, ou
qualquer coisa que crie um custo recorrente ou uma nova dependência? Se sim,
quais são as implicações para precificação, planos e decisões de build-or-buy?

---

## Formato do output

Gere um documento markdown e salve em:

```
specs/[domain]-[concept]/spec.md
```

### Convenção de nomenclatura obrigatória

O nome da pasta segue o formato `[domain]-[concept-in-up-to-2-words]`.

**Regras de idioma:**
- Nomes de pasta e arquivo são sempre em inglês
- Exceção: `societario` pode ser usado em português (não tem equivalente preciso em inglês)
- O conteúdo dos documentos é sempre em português

**Estrutura:**
- `[domain]` é sempre um dos domínios do produto: `societario`, `accountancy`, `dashboard`, `identity`
- `[concept]` é o nome curto da feature, em kebab-case, máximo 2 palavras, em inglês
- A pasta contém sempre dois arquivos com nomes fixos:
  - `spec.md` — a spec de produto (baseada em `templates/specification.md`)
  - `impact-map.md` — o mapa de impacto cross-repo (baseado em `templates/impact-map.md`)

Exemplos corretos:
```
specs/societario-tracking/spec.md
specs/accountancy-offices/spec.md
specs/dashboard-overview/spec.md
specs/identity-invites/spec.md
```

Nunca usar caminhos planos como `specs/[feature-name].md`.
Nunca usar sufixos como `-impact`, `-contract`, `-v2` no nome da pasta.
Nunca usar português no nome da pasta (exceto `societario`).

O documento deve conter as seguintes seções:

### Seções do documento de output

#### 1. Visão da feature
Um a três parágrafos na linguagem do usuário. O que é. Como o usuário
percebe. O que permite que ele faça. Sem linguagem técnica.

#### 2. Problema que resolve
Problema específico e real no contexto do usuário. Não vago. Não genérico.

#### 3. Para quem é
Quem se beneficia. Em qual situação. Qual perfil.

#### 4. Como melhora a experiência atual
Antes e depois. Como o fluxo de trabalho diário muda.

#### 5. Como se conecta com o produto existente
Dependências. O que habilita. Ordem de construção.

#### 6. O que não é (escopo negativo)
O que está explicitamente fora do escopo. Previne desvios.

#### 7. Custos adicionais
Serviços externos, APIs, armazenamento. Ou: "Nenhum custo externo identificado."

#### 8. Decisões tomadas
Tabela de decisões de produto fechadas durante o processo de spec.
Formato: `| Ponto | Decisão |`. Cresce a cada iteração da spec.
Quando uma pergunta aberta é respondida, sai da seção 9 e entra aqui.

#### 9. Perguntas abertas
Decisões não resolvidas. Informações faltantes. Honesto sobre incertezas.
Se não houver nenhuma: "Nenhuma. Todas as decisões de produto estão fechadas."
Nunca omitir esta seção — uma spec sem perguntas abertas declaradas escondeu
suas incertezas.

---

## Proibições absolutas

Essa skill nunca deve incluir no seu output:

- Trechos de código
- Schemas de banco de dados
- Contratos de API ou estruturas de endpoints
- Decisões de arquitetura
- Referências a stack tecnológico
- Nomes de componentes ou caminhos de arquivo
- Cronogramas de implementação
- Atribuições de sprint

---

## Checagem final antes de salvar

1. Uma pessoa não-técnica consegue ler isso e entender completamente a feature?
2. Todas as seções falam a linguagem do usuário, não do engenheiro?
3. O problema é específico o suficiente para validar com um usuário real?
4. O escopo negativo é claro o suficiente para prevenir desvios?
5. Se há custos, eles estão em termos de negócio?

Se qualquer resposta for não, revise antes de salvar.