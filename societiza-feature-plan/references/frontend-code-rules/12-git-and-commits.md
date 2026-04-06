# Git, Branches, Commits e Pull Requests

## Objetivo

O padrão de git do projeto deve preservar:

- rastreabilidade;
- revisão clara;
- separação de responsabilidade;
- e PRs que expliquem a mudança de forma útil.

Este documento fixa `develop` como base canônica atual do fluxo.

---

## Branch base

A branch base oficial para sincronização e integração é:

```txt
develop
```

### Regra

- criar branch nova a partir de `develop`;
- sincronizar branch de trabalho com `origin/develop`;
- PRs devem mirar `develop`, salvo instrução contrária explícita do projeto.

---

## Nomenclatura de branch

Padrão:

```txt
feat/{nome-da-feature}
fix/{o-que-corrige}
refactor/{o-que-refatora}
docs/{o-que-documenta}
chore/{tarefa-administrativa}
test/{escopo-do-teste}
```

Exemplos:

```txt
feat/dashboard-overview
fix/workflow-template-publish-error
refactor/accountancy-hooks
docs/frontend-code-rules
```

### Regras

- sempre `kebab-case`;
- o nome descreve o que a branch faz, não quem fez;
- se a mudança mistura responsabilidades grandes, dividir em mais de uma branch.

---

## Fluxo recomendado de branch

```bash
git checkout develop
git pull origin develop
git checkout -b feat/nome-da-feature
```

Antes de abrir PR ou publicar trabalho:

```bash
git fetch origin
git rebase origin/develop
```

### Regra

O padrão preferido é rebase para manter histórico mais limpo e PR mais legível.

---

## Commits

O projeto usa Conventional Commits em português.

Formato:

```txt
<tipo>(<escopo>): <descrição imperativa em português>
```

Exemplos:

```txt
feat(accountancy): adicionar mutation de edição
fix(workflow-template): corrigir ordem de fields
refactor(workflow): extrair orchestration do drawer
docs(frontend-code-rules): revisar convenções de arquitetura
```

---

## Tipos de commit

| Tipo | Uso |
|---|---|
| `feat` | comportamento novo |
| `fix` | correção de bug |
| `refactor` | reorganização sem mudança funcional principal |
| `docs` | documentação |
| `test` | testes |
| `chore` | manutenção e tarefas não-funcionais |
| `style` | mudança puramente estética/formatting sem lógica |

---

## Escopo do commit

O escopo deve refletir o domínio mais impactado.

Exemplos:

```txt
feat(accountancy): adicionar criação de contabilidade
fix(workflow): corrigir abertura do drawer
refactor(workflow-template): separar hooks de mutation
docs(dashboard): documentar estratégia de filtros
```

### Regra

- usar o nome da feature quando houver um dono claro;
- omitir o escopo só quando a mudança realmente atravessar muitas áreas sem um
  núcleo evidente.

---

## Commits por responsabilidade

Cada commit deve ser coeso.

### Bom

- schema e type da mesma operação;
- mutation hook de uma mesma ação;
- refactor focado numa separação arquitetural;
- doc correspondente a uma mudança estrutural específica.

### Ruim

- “várias mudanças”;
- feature nova + refactor aleatório + doc não relacionada;
- ajustes visuais misturados com correção de backend contract no frontend.

### Regra

Se a revisão do código não conseguir explicar o commit em uma frase clara, ele
provavelmente está grande ou misturado demais.

---

## Pull Requests

PR deve ser documento de revisão, não só URL de merge.

### Título

Seguir o mesmo padrão do commit:

```txt
feat(accountancy): adicionar fluxo de criação
```

### Corpo mínimo esperado

```md
## Resumo

O que mudou e por quê.

## Mudanças principais

- item 1
- item 2
- item 3

## Riscos ou conflitos

- ponto 1
- ponto 2

## Validação executada

- npm run build
- npm run lint
- npm run test
- npx playwright test
```

### Regra

Não abrir PR com descrição pobre do tipo:

- “ajustes”
- “docs”
- “adiciona md”
- “várias mudanças”

O PR precisa explicar impacto, contexto e risco.

---

## Rebase, merge e histórico

### Padrão

```bash
git fetch origin
git rebase origin/develop
```

### Regra

- preferir rebase para sincronização;
- evitar merge commit só para trazer `develop` para dentro da branch;
- não reescrever histórico após revisão aprovada sem necessidade forte.

---

## Force push

Permitido apenas em branch de trabalho própria e com cuidado:

```bash
git push origin feat/nome-da-feature --force-with-lease
```

### Regra

- nunca force push em branch compartilhada ou protegida;
- nunca force push em `develop`;
- usar `--force-with-lease`, nunca `--force` puro.

---

## O que não commitar

- `.env` e variantes com segredo;
- `.next/`;
- `node_modules/`;
- arquivos temporários de debug;
- logs descartáveis;
- comentários ou TODOs sem valor real;
- código morto criado por tentativa local.

---

## Relação com documentação

Mudança relevante precisa considerar:

- `README.md` da feature;
- `docs/features/tech/...`;
- eventualmente `docs/integration/...`.

### Regra

Se a mudança altera arquitetura, fluxo de dados ou contrato de integração, a doc
deve acompanhar o PR.

---

## Resumo operacional

- base canônica: `develop`;
- branch em `kebab-case`;
- commit convencional em português;
- commit coeso por responsabilidade;
- PR descritivo com riscos e validação;
- rebase em vez de merge de sincronização.
