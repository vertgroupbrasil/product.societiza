# Build e Qualidade de Código

## Objetivo

Qualidade no `web.societiza` significa entregar código que:

- compila;
- respeita o contrato de lint;
- está formatado;
- passa pelos testes cabíveis;
- e não deixa inconsistência estrutural escondida.

Build limpo não é detalhe. É parte da definição de pronto.

---

## Comandos reais do projeto

Os comandos de qualidade disponíveis hoje são:

```bash
npm run build
npm run lint
npm run format
npm run test
```

Para E2E, usar:

```bash
npx playwright test
```

### Regra

Documentação de qualidade deve citar comandos reais do repo. Se uma convenção
ideal depender de comando ainda não encapsulado em `package.json`, o comando deve
ser explicitado do jeito que realmente é executado.

---

## Ordem recomendada de validação

1. `npm run build`
2. `npm run lint`
3. `npm run test`
4. `npx playwright test` quando a mudança afetar UI ou fluxo
5. `npm run format` antes do fechamento final, se necessário

### Por que essa ordem

- build pega TypeScript, App Router e estrutura geral rapidamente;
- lint pega problemas de consistência e hygiene;
- Jest valida lógica;
- Playwright valida fluxo real;
- format fecha o estado final do código.

---

## O que `npm run build` realmente valida

No projeto atual, `npm run build` cobre principalmente:

- build do Next.js;
- type checking do TypeScript;
- imports inexistentes;
- erros de App Router;
- uso incorreto de Server/Client Components;
- inconsistências estruturais que só aparecem em produção.

### Implicações práticas

- `noUnusedLocals: true` pode quebrar build;
- alias errado quebra build;
- componente com hook sem `'use client'` quebra build;
- assinatura errada de rota App Router pode quebrar build.

---

## O que `npm run lint` realmente valida

O projeto usa `eslint.config.mjs` com:

- regras base de JS;
- `typescript-eslint`;
- `eslint-plugin-react`;
- `eslint-plugin-unicorn`;
- regras do Next;
- `plugin:react-hooks/recommended`.

### Regras relevantes do estado atual

- `@typescript-eslint/no-unused-vars`: `warn`
- `@typescript-eslint/no-explicit-any`: `off`
- `no-console`: `warn`, exceto `warn` e `error`
- `react-hooks/exhaustive-deps`: `warn`
- `unicorn/no-lonely-if`: `warn`
- `unicorn/prefer-optional-catch-binding`: `warn`
- `unicorn/filename-case`: `off`

### Regra de interpretação

Mesmo quando a configuração atual não bloqueia tudo, o ruleset ideal continua
mandando resolver o que for razoável antes de fechar a entrega. Warning tolerado
não é convite para degradar o código.

---

## `npm run format`

O projeto usa:

```bash
prettier --write .
```

Isso significa que `npm run format` formata o repositório inteiro.

### Regra prática

- usar com consciência;
- evitar rodar format global sem necessidade em branch carregada de mudanças não
  relacionadas;
- mas não fechar alteração estrutural deixando arquivo fora do padrão.

---

## Relação entre TypeScript e ESLint

Nem todo problema aparece nas mesmas camadas:

- TypeScript quebra tipos e imports;
- ESLint sinaliza hygiene e consistência;
- Jest e Playwright validam comportamento.

Portanto:

- build verde não substitui lint;
- lint verde não substitui teste;
- teste verde não substitui build.

---

## Flags importantes do `tsconfig`

### `exactOptionalPropertyTypes`

Exige modelagem correta de opcionais.

### `noImplicitAny`

Qualquer brecha de inferência vira erro.

### `noUnusedLocals`

Variável local não usada quebra.

### `moduleResolution: bundler`

Imports e aliases precisam respeitar o modelo moderno do projeto.

---

## Qualidade mínima antes de fechar trabalho

Checklist mínimo:

```txt
□ build passou
□ lint passou ou warnings ficaram explicitamente justificados
□ testes Jest relevantes foram rodados
□ Playwright foi rodado quando houve UI/fluxo
□ imports respeitam barrels
□ nenhum componente visual ficou com orchestration indevida
□ documentação foi atualizada se a mudança for relevante
```

---

## O que não pode entrar no código final

- `console.log` de debug esquecido;
- import não usado;
- arquivo morto desconectado da feature;
- contrato quebrado escondido por cast desnecessário;
- código novo copiando padrão ruim do legado sem justificativa.

---

## Como reportar qualidade

A entrega deve informar:

- quais comandos foram rodados;
- se houve warning relevante;
- se algum teste falhou por problema pré-existente;
- se algum passo não se aplicava ao escopo.

Exemplo:

```txt
Build:
- npm run build
- passou

Lint:
- npm run lint
- passou com warnings antigos fora do escopo

Jest:
- npm run test -- src/features/accountancy
- passou

Playwright:
- npx playwright test __tests__/e2e/accountancy
- passou
```

---

## Resumo operacional

- usar os comandos reais do repo;
- build, lint e teste têm papéis diferentes;
- `npx playwright test` é o padrão de E2E enquanto não houver script dedicado;
- warning tolerado pelo tooling não vira licença para relaxar qualidade;
- fechamento de trabalho exige evidência de validação.
