# Skills — Guia de Uso

Skills são comportamentos especializados que o Claude carrega sob demanda.
Em vez de ter um assistente genérico, cada skill transforma o Claude em um
especialista para um contexto específico.

Este projeto tem duas skills principais: uma para planejar produto e outra
para planejar implementação de código.

---

## Como ativar uma skill

Você não usa comando nenhum. Basta descrever o que quer fazer em linguagem
natural. O Claude identifica qual skill é adequada e a ativa automaticamente.

Exemplos:

```
"quero planejar a feature de notificações"
"preciso de uma especificação de produto para o portal do cliente"
"vamos implementar o kanban, me ajuda a planejar"
"quero definir o que seria a feature de relatórios antes de codar"
```

---

## `societiza-product-plan`

**Para usar:** Descreva uma ideia ou feature que você quer pensar antes de
construir.

### O que faz

Planeja produto com mentalidade Steve Jobs. Antes de escrever uma linha de
código, responde a pergunta mais importante: **essa feature deve existir, e se
sim, o que exatamente ela deve ser?**

Não produz código, arquitetura, ou decisões técnicas. Só produto: o usuário,
o problema, a experiência, o fit com o produto, e os custos.

### Quando usar

- Quando você tem uma ideia mas não sabe se vale construir
- Quando quer definir o escopo antes de entrar no código
- Quando precisa de um documento de especificação para alinhar expectativas

### O que a skill faz internamente

1. Lê `docs/business-context.md` e `docs/feature-map.md` para entender o
   produto atual
2. Analisa o problema real, para quem é, o fit com o produto, e o custo
3. Gera um documento de especificação salvo em:

```
docs/features/product/[nome-da-feature].md
```

### O que o documento gerado contém

- Visão da feature (na linguagem do usuário, sem jargão técnico)
- Problema que resolve (específico, real, com contexto)
- Para quem é (qual perfil, em qual situação)
- Como melhora a experiência atual (antes e depois)
- Como se conecta com o produto existente (dependências e o que habilita)
- O que não é — escopo negativo explícito
- Custos adicionais (se houver serviços externos)
- Perguntas abertas (decisões não resolvidas)

### Exemplo de conversa

```
você: "quero planejar a feature de portal do cliente"

Claude: [ativa societiza-product-plan, lê o contexto do produto, faz
perguntas se necessário, gera docs/features/product/portal-do-cliente.md]
```

---

## `societiza-feature-plan`

**Para usar:** Diga que quer implementar uma feature específica.

### O que faz

Planeja a implementação de código de uma feature. Lê as regras de código do
projeto, entende o contexto de negócio da feature, e gera um plano técnico
completo com arquivos a criar, ordem de implementação e verificação.

### Quando usar

- Quando a feature já está definida (tem especificação de produto ou regra de
  negócio clara) e você quer planejar o código
- Antes de começar a implementar qualquer feature nova
- Quando quer garantir que a implementação vai seguir as regras do projeto

### Pré-requisito

A feature precisa estar definida. Se não houver especificação em
`docs/features/product/` nem regra de negócio documentada, a skill para e
redireciona para `societiza-product-plan` primeiro.

Se você tiver a regra de negócio na cabeça mas não documentada, pode fornecer
manualmente na conversa e a skill aceita isso como base.

### O que a skill faz internamente

1. Busca a regra de negócio da feature em `docs/`
2. Pergunta: frontend ou backend?
3. Carrega **todos** os arquivos de regras de código correspondentes:
   - Frontend: `.agents/skills/societiza-feature-plan/references/frontend-code-rules/`
   - Backend: `.agents/skills/societiza-feature-plan/references/backend-code-rules/`
4. Verifica se existe contrato de integração em `docs/integration/`
5. Gera o plano de implementação

### O que o plano gerado contém

- **Mapa de integração técnica** — o que já existe no codebase que é relevante
- **Arquivos a criar ou modificar** — caminho, responsabilidade e justificativa
  de cada arquivo
- **Ordem de implementação** — dependências explícitas entre cada passo
- **Verificação end-to-end** — como confirmar que a feature funciona depois

### Boundary rules (não negociáveis)

**Se backend foi selecionado:** frontend não existe na conversa. Zero menção
a componentes, hooks, schemas de UI ou caminhos de arquivo frontend.

**Se frontend foi selecionado:** backend é bloqueado por padrão. A skill pode
sugerir uma melhoria de backend uma única vez se isso impactar diretamente a
experiência do usuário — mas só se você permitir.

### Exemplo de conversa

```
você: "vamos implementar a feature de auth, preciso de um plano"

Claude: [ativa societiza-feature-plan, busca regra de negócio de auth,
pergunta frontend ou backend, carrega todas as regras de código, gera plano]
```

---

## Fluxo ideal entre as duas skills

```
1. societiza-product-plan
   → define o que a feature é, para quem, e por quê
   → salva em docs/features/product/[feature].md

2. societiza-feature-plan
   → lê a especificação gerada
   → planeja como implementar no código
   → você aprova o plano

3. implementação
   → código segue o plano aprovado

4. commit
   → só depois que você testar e liberar
```

Nunca pule a etapa 1 achando que a feature "é óbvia". Features sem
especificação acumulam retrabalho e scope creep.

---

## Onde ficam as regras de código

As regras que a `societiza-feature-plan` carrega ficam em:

```
.agents/skills/societiza-feature-plan/references/frontend-code-rules/
├── 01-architecture.md
├── 02-creating-a-feature.md
├── 03-typescript-and-schemas.md
├── 04-queries.md
├── 05-mutations.md
├── 06-forms.md
├── 07-components.md
├── 08-naming-conventions.md
├── 09-testing.md
├── 10-imports-and-barrels.md
├── 11-build-and-quality.md
├── 12-git-and-commits.md
├── 13-documentation.md
├── 14-folder-contracts.md
└── 15-workflow-domain.md
```

Esses arquivos são a memória técnica do projeto. Quando o Claude comete um
erro de padrão e você corrige, a regra correspondente deve ser atualizada
para que o erro não se repita.
