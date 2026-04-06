---
name: societiza-product-plan
description: Product planning skill for Societiza with Steve Jobs mentality. Use when deciding whether to build a feature, defining what a feature should be, understanding how a new feature connects to the product, or generating a product spec document. This skill thinks product-first. No code, no architecture, no stack. Only product: the user, the problem, the experience, the fit, and the cost.
user-invocable: true
argument-hint: "[feature name or problem description]"
---

# Societiza Product Plan

This skill thinks about features the way Steve Jobs thought about products.

Product first. Engineering figures out the rest.

Before a single line of code is written, before any architecture decision is
made, before any sprint is planned — this skill answers the question that
matters most: **should this exist, and if so, what exactly should it be?**

---

## Required reading before thinking

Load these documents before responding. Read them in full.

1. `references/steve-jobs-mentality.md` — the product thinking framework
2. `docs/business-context.md` — what Societiza is, who it serves, what exists
3. `docs/feature-map.md` — how features connect and what each one enables

Do not proceed without reading all three.

---

## What this skill does

This skill produces a product specification document for a feature. It does not
produce technical plans, implementation guides, or architecture decisions.

Everything in the output is written in the language of the user and the product,
not in the language of engineers.

---

## Thinking process (run internally before writing)

Before writing the output document, think through:

**1. What is the real problem?**
Not what the feature does. What problem it solves. Who has that problem. How
often they face it. What they do today to work around it. Whether this is a
problem worth solving now or later.

**2. Does this fit the product?**
Using `docs/business-context.md` and `docs/feature-map.md`: where does this
feature sit in the product? Which horizon does it belong to? Does it strengthen
the core, or does it add noise? What features does it depend on? What features
does it enable?

**3. What is the Steve Jobs question?**
From `references/steve-jobs-mentality.md`: run the 10 mandatory questions.
If you cannot answer at least 8 clearly, say so in the output document and mark
the feature as needing more definition before it can be approved.

**4. What is the minimal version that matters?**
What is the smallest version of this feature that genuinely solves the problem?
What is the version that would make a user say "finally"? Not what is the
complete vision — what is the smallest version with real impact.

**5. Are there costs beyond code?**
Does this feature require external services, third-party APIs, storage, email
providers, WhatsApp integrations, government APIs, or anything else that creates
a recurring cost or a new dependency? If so, what are the implications for
pricing, plans, and build-or-buy decisions?

---

## Output format

Generate a markdown document and save it at:

```
web.societiza/docs/features/product/[feature-name].md
```

Use the feature name as the file name, in kebab-case, without dates. For example:
- `dashboard.md`
- `notificacoes.md`
- `portal-do-cliente.md`
- `formulario-nativo.md`

The document must contain the following sections:

---

### Sections of the output document

#### 1. Visão da feature

One to three paragraphs describing the feature in the language of the user, not
the engineer. What it is. How the user perceives it. What it enables them to do
that they could not do before.

No technical language. No architecture. No stack. Just the experience.

#### 2. Problema que resolve

What specific, real problem does this feature solve? Describe the problem in the
user's context. What happens today without this feature? What frustration,
inefficiency, or risk does the user live with? How often does this problem occur?

Be specific. "The accountant doesn't have visibility" is not a problem. "The
accountant discovers that a process has been stopped for 5 days only when the
client calls complaining" is a problem.

#### 3. Para quem é

Who exactly benefits from this feature? Is it the accountant? The office manager?
The business owner (client)? All of them? In what specific situation do they
use it?

#### 4. Como melhora a experiência atual

Before and after. What the user does today (without this feature) and what
they will do after (with this feature). How the daily workflow changes. What
becomes possible that was impossible before.

#### 5. Como se conecta com o produto existente

Based on `docs/feature-map.md`: what existing features does this depend on?
What features become more powerful because of this one? What features need to
be built before this one can exist? What does this feature enable for the future?

If the feature depends on something that does not yet exist (e.g., notifications
when that feature hasn't been built), state this clearly. Building order matters.

#### 6. O que não é (escopo negativo)

What is explicitly NOT part of this feature. What the user might expect but is
out of scope. What is a future evolution. What is a different feature entirely.

This section prevents scope creep and sets correct expectations.

#### 7. Custos adicionais

Only include this section if the feature requires external services, third-party
APIs, storage infrastructure, or any other recurring cost beyond development.

If the feature is pure code with no external dependencies, write:
"Nenhum custo externo identificado. Esta feature requer apenas código."

If costs exist, describe:
- What service or resource is needed
- What it enables (why it's necessary)
- General cost range or pricing model (if known)
- Which product plan or tier this cost belongs to
- Whether this cost is passed to the user or absorbed

#### 8. Perguntas abertas

What decisions are not yet made? What information is still needed to fully
define this feature? What tradeoffs have not been resolved? What depends on
user research, market validation, or a business decision?

This section is honest about what is unknown.

---

## Absolute prohibitions

This skill must never include in its output:

- Code snippets
- Database schemas
- API contracts or endpoint structures
- Architecture decisions
- Technology stack references
- Component names or file paths
- Implementation timelines
- Sprint assignments

Any mention of these means the skill has left product territory and entered
engineering territory. Stay in product territory.

---

## Final check before saving the document

Before saving, verify:

1. Can a non-technical person read this document and fully understand the feature?
2. Does every section speak the language of the user, not the engineer?
3. Is the problem stated specifically enough that someone could validate it with
   a real user?
4. Is the "what this is not" section clear enough to prevent scope drift?
5. If there are costs, are they described in business terms, not technical terms?

If any answer is no, revise before saving.
