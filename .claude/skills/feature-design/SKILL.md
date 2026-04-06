---
name: feature-design
description: >
  Product planning skill for Societiza. Use when deciding whether to build a
  feature, defining what it should be, understanding how it connects to the
  product, or generating a product spec. Product-first — no code, no
  architecture, no stack.
user-invocable: true
argument-hint: "[feature name or problem description]"
---

# Feature Design

This skill thinks about features the way Steve Jobs thought about products.

Product first. Engineering figures out the rest.

Before a single line of code is written, before any architecture decision is
made, before any sprint is planned — this skill answers the question that
matters most: **should this exist, and if so, what exactly should it be?**

---

## Required reading before thinking

Load these documents before responding. Read them in full.

1. `references/steve-jobs-mentality.md` — the product thinking framework
2. `docs/business-context.md` — product context index
3. `docs/planning/feature-map.md` — how features connect and what each one enables

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
Using `docs/business-context.md` and `docs/planning/feature-map.md`: where does this
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
features/specs/[feature-name].md
```

Use the feature name as the file name, in kebab-case, without dates.

The document must contain the following sections:

### Sections of the output document

#### 1. Visão da feature
One to three paragraphs in user language. What it is. How the user perceives
it. What it enables them to do. No technical language.

#### 2. Problema que resolve
Specific, real problem in the user's context. Not vague. Not generic.

#### 3. Para quem é
Who benefits. In what situation. What profile.

#### 4. Como melhora a experiência atual
Before and after. How the daily workflow changes.

#### 5. Como se conecta com o produto existente
Dependencies. What it enables. Building order.

#### 6. O que não é (escopo negativo)
What is explicitly out of scope. Prevents drift.

#### 7. Custos adicionais
External services, APIs, storage. Or: "Nenhum custo externo identificado."

#### 8. Perguntas abertas
Unresolved decisions. Missing information. Honest about unknowns.

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

---

## Final check before saving

1. Can a non-technical person read this and fully understand the feature?
2. Does every section speak the language of the user, not the engineer?
3. Is the problem specific enough to validate with a real user?
4. Is the scope negative clear enough to prevent drift?
5. If there are costs, are they in business terms?

If any answer is no, revise before saving.
