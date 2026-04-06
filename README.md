# Skills — Guia de Uso

Skills são comportamentos especializados que o Claude carrega sob demanda.
Em vez de ter um assistente genérico, cada skill transforma o Claude em um
especialista para um contexto específico: planejar features, fazer deploy,
gerar conteúdo social, construir interfaces, etc.

---

## Como funciona

Quando você invoca uma skill, o Claude:

1. Carrega o arquivo `SKILL.md` daquela skill
2. Lê todos os arquivos de referência indicados no SKILL.md
3. Segue o protocolo definido na skill — fazendo perguntas, carregando
   contexto do projeto, ou executando ações em sequência
4. Responde já operando dentro do especialismo daquela skill

Skills são **stateful dentro da conversa**: uma vez ativada, a skill permanece
no contexto enquanto a conversa durar.

---

## Como invocar uma skill

### Via `/` (slash command)

Digite `/` seguido do nome da skill. Você pode passar um argumento opcional:

```
/societiza-feature-plan auth
/societiza-product-plan notificações
/deploy-to-vercel
/social-content
/frontend-design
```

### Via descrição natural

Se você descrever uma tarefa que corresponde ao gatilho de uma skill, o Claude
ativa a skill automaticamente. Por exemplo:

- "quero planejar a feature de kanban" → ativa `societiza-feature-plan`
- "faz deploy da aplicação" → ativa `deploy-to-vercel`
- "cria um post para o LinkedIn" → ativa `social-content`

---

## Skills disponíveis neste projeto

### `societiza-feature-plan`

**O que faz:** Planejamento de implementação de código para features do
Societiza. Lê as regras de código (frontend ou backend), entende o contexto
de negócio da feature, e gera um plano de implementação completo com arquivos,
ordem de criação e verificação.

**Quando usar:** Quando você sabe o que precisa construir e quer um plano
técnico antes de escrever código.

**Fluxo:**

1. Pergunta qual feature
2. Busca a regra de negócio em `docs/`
3. Pergunta: frontend ou backend?
4. Carrega todas as regras de código correspondentes
5. Gera o plano de implementação

**Exemplo:**

```
/societiza-feature-plan kanban
/societiza-feature-plan auth
```

**Referências que esta skill usa:**

```
.agents/skills/societiza-feature-plan/references/frontend-code-rules/
.agents/skills/societiza-feature-plan/references/backend-code-rules/
docs/business-context.md
docs/feature-map.md
docs/features/product/
docs/integration/
```

---

### `societiza-product-plan`

**O que faz:** Planejamento de produto com mentalidade Steve Jobs. Decide se
uma feature deve existir, define o que ela deve ser, e gera uma especificação
de produto salva em `docs/features/product/`.

**Quando usar:** Antes de escrever qualquer código. Quando você tem uma ideia
mas precisa validar se faz sentido para o produto, para quem serve, e o que
exatamente deve ser construído.

**Fluxo:**

1. Lê `docs/business-context.md` e `docs/feature-map.md`
2. Analisa o problema real, o fit com o produto, e o escopo mínimo
3. Gera um documento de especificação de produto em `docs/features/product/`

**Exemplo:**

```
/societiza-product-plan portal do cliente
/societiza-product-plan notificações em tempo real
```

**Saída:** Arquivo `.md` em `docs/features/product/[nome-da-feature].md`

---

### `deploy-to-vercel`

**O que faz:** Faz deploy da aplicação no Vercel. Detecta o estado do projeto
(vinculado ao Vercel ou não, autenticado ou não) e escolhe o método correto.
Sempre faz preview por padrão, não produção.

**Quando usar:** Quando você quer publicar a aplicação ou gerar um link de
preview.

**Métodos em ordem de preferência:**

1. Git push (se o projeto já está vinculado ao Vercel com remote)
2. `vercel deploy` via CLI (se vinculado mas sem remote)
3. Link + deploy (se não vinculado mas CLI autenticada)
4. Script sem autenticação (fallback para ambientes sem CLI)

**Exemplo:**

```
/deploy-to-vercel
"faz deploy e me manda o link"
```

---

### `frontend-design`

**O que faz:** Cria interfaces frontend com alta qualidade de design — evitando
a estética genérica de IA. Gera código de produção com tipografia, cores,
espaçamento e motion intencionais.

**Quando usar:** Quando você precisa construir um componente, página ou tela
com qualidade visual além do padrão.

**Referências que esta skill usa:**

```
.agents/skills/frontend-design/reference/typography.md
.agents/skills/frontend-design/reference/color-and-contrast.md
.agents/skills/frontend-design/reference/spatial-design.md
.agents/skills/frontend-design/reference/motion-design.md
.agents/skills/frontend-design/reference/interaction-design.md
.agents/skills/frontend-design/reference/responsive-design.md
.agents/skills/frontend-design/reference/ux-writing.md
```

**Exemplo:**

```
/frontend-design
"cria uma landing page para o produto"
"cria um componente de card de cliente com design profissional"
```

---

### `shadcn`

**O que faz:** Gerencia componentes shadcn/ui — instalação, configuração,
composição correta, regras de estilo e atualização. Ativa automaticamente
quando o projeto tem `components.json`.

**Ativação:** Automática. Não precisa invocar manualmente.

**Quando é útil:** Quando você quer adicionar um componente novo, corrigir
composição errada, ou entender como usar uma variante específica.

---

### `social-content`

**O que faz:** Criação de conteúdo para redes sociais — LinkedIn, Twitter/X,
Instagram, TikTok. Define pilares de conteúdo, gera posts, threads, carrosséis
e calendário editorial.

**Quando usar:** Quando você precisa criar conteúdo para divulgar o Societiza
ou conteúdo de marca pessoal.

**Exemplo:**

```
/social-content
"cria um post para o LinkedIn sobre o lançamento da feature de kanban"
```

---

## Estrutura de uma skill

Toda skill vive em `.agents/skills/[nome-da-skill]/`:

```
.agents/skills/
└── nome-da-skill/
    ├── SKILL.md           ← protocolo da skill (obrigatório)
    ├── references/        ← arquivos de contexto que a skill lê
    │   └── *.md
    ├── resources/         ← scripts e assets que a skill usa
    │   └── *.sh
    └── evals/             ← testes de avaliação da skill (opcional)
        └── evals.json
```

### `SKILL.md`

O arquivo principal. Contém:

- **Frontmatter** — metadados: `name`, `description`, `user-invocable`,
  `argument-hint`
- **Protocolo** — o passo a passo que o Claude segue ao executar a skill
- **Referências** — quais arquivos carregar e quando
- **Regras de boundary** — o que a skill pode e não pode fazer

---

## Tipos de skill

### `user-invocable: true`

A skill pode ser invocada explicitamente via `/nome-da-skill`. O Claude também
pode ativá-la automaticamente quando detecta que o contexto corresponde.

Exemplos: `societiza-feature-plan`, `societiza-product-plan`, `social-content`

### `user-invocable: false` (ou ausente)

A skill ativa automaticamente quando o contexto corresponde ao gatilho. Não
precisa ser invocada pelo usuário.

Exemplos: `shadcn` (ativa quando há `components.json` no projeto)

---

## Como adicionar uma nova skill

1. Crie a pasta: `.agents/skills/[nome-da-skill]/`
2. Crie o `SKILL.md` com:
   - Frontmatter com `name` e `description`
   - O protocolo passo a passo
   - Indicação dos arquivos de referência
3. Adicione arquivos de referência em `references/` se necessário
4. Teste invocando com `/nome-da-skill`

### Template mínimo de `SKILL.md`

```markdown
---
name: nome-da-skill
description: O que esta skill faz e quando deve ser usada.
user-invocable: true
argument-hint: "[argumento opcional]"
---

# Nome da Skill

Descrição do que a skill faz.

## Passo 1 — ...

## Passo 2 — ...
```

---

## Relação entre as skills do Societiza

```
societiza-product-plan
    ↓ gera docs/features/product/[feature].md
societiza-feature-plan
    ↓ lê docs/features/product/ + frontend-code-rules/
    ↓ gera plano de implementação
[você implementa]
deploy-to-vercel
    ↓ publica
```

A `societiza-feature-plan` bloqueia a execução se a feature não tiver
especificação de produto. Se não existir, ela redireciona para
`societiza-product-plan` antes de continuar.

---

## Regras de negócio das skills do Societiza

As skills do Societiza leem as regras de código de:

```
.agents/skills/societiza-feature-plan/references/frontend-code-rules/
```

São 15 arquivos cobrindo arquitetura, criação de feature, TypeScript, queries,
mutations, forms, components, nomenclatura, testes, imports/barrels,
build/qualidade, git, documentação, contratos de pasta e domínio workflow.

Manter esses arquivos atualizados é crítico — eles são a memória técnica do
projeto e definem como todo código novo deve ser escrito.
