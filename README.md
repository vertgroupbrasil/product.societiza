# Societiza — Hub de Produto e Orquestração

Este repositório é o centro do ecossistema Societiza. Ele não contém
código de aplicação. Seu papel é receber intenções de produto, transformá-las
em decisões claras e distribuir o trabalho certo para os repos certos.

Tudo que é implementado nos repos de execução começa aqui — como uma spec
aprovada por um humano, depois de análise de coerência de produto, fit no
ecossistema, dependências e escopo.

---

## Por que este repo existe

Times de produto e engenharia costumam divergir porque o contexto de produto
não está estruturado onde a engenharia trabalha. A engenharia implementa o que
entende, não o que o produto precisa. O produto pede o que parece certo, sem
saber o que o ecossistema já tem ou o que quebra.

Este repo resolve isso criando uma camada explícita de orquestração:
- O produto pensa aqui
- As decisões são documentadas aqui
- A engenharia busca os insumos aqui

Nenhuma feature é implementada sem uma spec aprovada. Nenhuma decisão de
produto fica implícita em uma conversa.

---

## Ecossistema

- [`product.societiza`](https://github.com/vertgroupbrasil/product.societiza) — este repo (orquestrador de produto)
- [`api.societiza`](https://github.com/vertgroupbrasil/api.societiza) — backend (.NET 9)
- [`web.societiza`](https://github.com/vertgroupbrasil/web.societiza) — frontend (Next.js)

Os repos de execução não esperam que o produto vá até eles. A comunicação é
estruturada via **agente + MCP do GitHub**: `api.societiza` e `web.societiza`
buscam specs diretamente neste repositório. Quando uma feature é aprovada,
os repos de execução têm acesso imediato aos artefatos — sem repassar
contexto manualmente, sem sumarizar em PR descriptions o que já está
documentado aqui.

Este repo é a fonte de verdade de produto para todo o ecossistema. Qualquer
agente rodando em `api.societiza` ou `web.societiza` pode consultar o contexto
de produto aqui, sem depender de memória humana.

---

## Estrutura do repositório

```
.claude/
  agents/
    product-thinking.md    <- agente principal (cérebro de produto)
  skills/
    feature-design/
      SKILL.md             <- skill de planejamento de produto
      references/          <- materiais de referência da skill
  rules/
    cross-repo.md          <- o que este repo faz e não faz
    spec-first.md          <- spec aprovada antes de qualquer implementação
  CLAUDE.md                <- instruções gerais do repositório

docs/
  product/
    product-vision.md      <- identidade, ICP, valores, diretrizes de features
    product-priorities.md  <- semântica de prioridades P1–P5
  context/
    societario-context.md  <- workflow, template, board, campos
    accountancy-context.md <- accountancy como unidade multi-tenant
    identity-context.md    <- estado e lacunas de autenticação
  planning/
    dashboard.md           <- próxima feature obrigatória
    future-features.md     <- todas as features futuras planejadas
    feature-map.md         <- dependências e acoplamento entre features
  business-context.md      <- índice de contexto do produto (leitura obrigatória)

specs/
  [domínio]-[conceito]/    <- ex: societario-tracking/
    spec.md                <- spec de produto aprovada
    integration.md         <- contrato backend → frontend (gerado após implementação)

templates/
  specification.md         <- modelo canônico de spec de produto
  integration.md           <- modelo de contrato backend → frontend
```

---

## Skill: Feature Design

A skill `feature-design` é o motor de pensamento de produto do ecossistema.

Ela não lista requisitos. Não produz backlog. Não define sprint. Ela responde
à pergunta mais importante antes de qualquer linha de código ser escrita:
**isso deve existir e, se sim, o que exatamente deve ser?**

### Mentalidade Steve Jobs

O diferencial central desta skill é o framework de pensamento baseado na
mentalidade de Jobs. Antes de qualquer análise de fit, dependências ou escopo,
a skill força clareza sobre o que realmente importa:

**Produto é experiência, não software.** A pergunta não é "o que esse módulo
faz?" É "o que o usuário sente quando usa isso?" Mais funcionalidade quase
sempre piora a experiência — porque adiciona fricção, confusão e sobrecarga
cognitiva.

**Qual é o problema real?** Não o que a feature faz — que dor específica ela
elimina, para quem, com que frequência, e o que o usuário faz hoje sem ela.
Features sem problema definido são devolvidas para refinamento.

**O que você não constrói importa tanto quanto o que você constrói.** Uma
feature que não lança não cria dívida de produto, não distrai o usuário, não
cria expectativa que precisa ser sustentada. Dizer não é uma forma de proteção,
não de limitação.

**Versão mínima que importa.** Qual é a menor versão desta feature que faria
um usuário dizer "finalmente"? Não a visão completa — a menor versão com
impacto real.

**Custo além do código.** Se a feature exige serviços externos, APIs de
terceiros ou integrações, esse custo é parte da decisão de produto — não só
do plano financeiro.

A skill não aprova uma feature se não consegue responder pelo menos 8 das 10
perguntas obrigatórias do framework. O output é sempre um documento na
linguagem do usuário — sem código, sem arquitetura, sem stack. Algo que o
usuário final pode ler e reconhecer como verdadeiro sobre o próprio problema.

---

## Skill: Excalidraw Diagram

A skill `excalidraw-diagram` gera diagramas Excalidraw a partir de descrições
em linguagem natural. Não são diagramas decorativos — são **argumentos
visuais** que tornam workflows, arquiteturas e conceitos compreensíveis de
forma que texto sozinho não consegue.

### Diagramas que argumentam, não exibem

O diferencial central desta skill é tratar cada diagrama como um argumento
visual, não como texto formatado em caixas. Cada forma e agrupamento espelha
o conceito que representa — fan-outs para relações um-para-muitos, timelines
para sequências, convergência para agregação. Se remover todo o texto, a
estrutura sozinha ainda comunica o conceito.

**Artefatos de evidência.** Diagramas técnicos incluem exemplos concretos —
snippets de código reais, payloads JSON, nomes de eventos da spec, mockups
de UI. Não caixas genéricas com "Input" e "Output", mas o que os dados
realmente parecem.

**Arquitetura multi-zoom.** Diagramas abrangentes operam em três níveis
simultâneos: fluxo resumido (visão geral do pipeline), seções agrupadas
(fronteiras de responsabilidade) e detalhes internos (artefatos de evidência
dentro de cada seção).

**Validação visual integrada.** Um pipeline baseado em Playwright renderiza
o diagrama gerado, permitindo que o agente veja o próprio output, detecte
problemas de layout (textos sobrepostos, setas desalinhadas, espaçamento
desbalanceado) e corrija antes de entregar.

**Customizável por marca.** Todas as cores e estilos vivem em um único
arquivo (`references/color-palette.md`). Para adaptar os diagramas à
identidade visual do projeto, basta editar esse arquivo — todo o resto da
skill é metodologia universal de design.

### Setup

Para configurar o renderer de validação visual, peça ao agente:
*"Configure o renderer da skill excalidraw-diagram seguindo as instruções
do SKILL.md."* Ou manualmente:

```bash
cd .claude/skills/excalidraw-diagram/references
uv sync
uv run playwright install chromium
```

### Uso

Descreva o que precisa ser visualizado em linguagem natural. A skill cuida
do mapeamento conceitual, layout, geração do JSON Excalidraw, renderização
e validação visual.

---

## Como trabalhar com este repositório

### Agente principal

O agente `product-thinking` (`.claude/agents/product-thinking.md`) é o ponto
de entrada para qualquer sessão de produto. Ele lê o contexto, faz triagem de
prioridade e decide qual caminho seguir.

### Dois caminhos de trabalho

**Caminho de produto** — quando a ideia ainda está sendo formada.

Descreva a ideia em linguagem natural. O agente invoca `feature-design`, que
lê o contexto, aplica o framework de pensamento e gera uma spec em
`specs/`. O humano aprova ou pede revisões. Nenhum repo de execução
é acionado.

**Caminho de execução** — quando a decisão de produto já foi tomada.

O agente invoca `feature-design`, aguarda aprovação humana da spec e, após aprovação,
invoca `logic-design` para gerar o contrato compartilhado entre FE e BE.

---

## Regras ativas

Toda sessão carrega as rules em `.claude/rules/` automaticamente:

- **spec-first** — nenhuma feature é implementada sem spec aprovada
- **cross-repo** — este repo produz specs, não código