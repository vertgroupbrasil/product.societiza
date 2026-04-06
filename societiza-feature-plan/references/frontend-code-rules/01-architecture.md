# Arquitetura do Frontend — Societiza

## Objetivo deste documento

Este documento define o **padrão ideal de arquitetura frontend** para o
`web.societiza`. Ele não é um espelho literal de cada arquivo legado do repositório.
Ele existe para orientar decisões novas e revisões de código a partir de um
contrato claro, estável e imediatamente aplicável.

A regra base é simples: o frontend é **feature-based**, com fluxo de abstração
previsível e separação rígida entre infraestrutura, dados, orchestration e
apresentação.

---

## Visão geral da arquitetura

O projeto usa Next.js App Router e organiza o código por domínio de produto.
Isso significa que quase todo código novo deve nascer dentro de
`src/features/{feature}`. Cada feature concentra:

- schemas;
- services;
- hooks;
- components;
- constants;
- documentação;
- e a sua própria superfície pública de import via `index.ts`.

Ao mesmo tempo, o projeto **também possui camadas compartilhadas reais** fora das
features, e elas fazem parte da arquitetura oficial:

- `src/app/` para rotas, layouts e páginas App Router;
- `src/components/` para componentes compartilhados verdadeiros;
- `src/lib/` para utilitários globais sem dono claro de feature;
- `src/hooks/` para hooks globais raros;
- `src/types/` para tipos globais raros;
- `src/routes/endpoints.ts` para o registro central de endpoints.

Não é correto fingir que essas camadas não existem. O padrão certo é:

- código de domínio vive na feature;
- código global só existe quando realmente é compartilhado e não pertence a uma
  única feature.

---

## Estrutura raiz do projeto

```txt
src/
  app/                    ← rotas e entrypoints do Next.js App Router
  features/               ← features de domínio
  components/             ← componentes compartilhados entre features
  hooks/                  ← hooks globais raros
  lib/                    ← utilitários globais e infraestrutura compartilhada
  routes/
    endpoints.ts          ← registro centralizado de endpoints
  types/                  ← tipos globais raros
```

Regra operacional:

- procurar primeiro a feature dona do problema;
- só criar algo global quando o código realmente não tiver um único dono de
  feature.

---

## Estrutura ideal de uma feature

Nem toda feature precisa de todas as pastas, mas a estrutura alvo é esta:

```txt
src/features/{feature}/
  schemas/
    {entity}.schema.ts
    index.ts
  server/
    services/
      {entity}.service.ts
      index.ts
    types/
      {entity}.types.ts
      index.ts
    index.ts
  hooks/
    queries/
      query-options.ts
      use{Entity}Queries.ts
      index.ts
    mutations/
      use{Entity}Mutations.ts
      index.ts
    forms/
      use{Entity}Form.ts
      index.ts
    utils/
      useSomething.ts
      index.ts
    components/
      use{Component}.ts
      index.ts
    index.ts
  contexts/
    index.ts
  constants/
    {feature}.constants.ts
    index.ts
  lib/
    something.ts
    index.ts
  components/
    forms/
      {Entity}Form.tsx
      index.ts
    ui/
      {Component}.tsx
      index.ts
    {Component}.tsx
    index.ts
  README.md
  index.ts
```

### Observações importantes

- `hooks/components/` é parte do padrão ideal e deve ser usado quando um
  componente exigir orchestration real. Nem todas as features atuais já adotam
  isso, mas o contrato novo passa a tratá-lo como padrão-alvo.
- `server/types/` é a API pública de tipos da feature. Em features legadas você
  ainda pode encontrar `lib/types/` ou tipos exportados direto de `schemas/`;
  isso não vira precedente para código novo.
- `lib/` é o bucket preferido para helpers internos puros da feature.
  `utils/` só deve existir se a feature já estiver organizada assim ou se houver
  um agrupamento misc realmente justificável.

---

## Fluxo de abstração obrigatório

O fluxo oficial do frontend é:

```txt
src/routes/endpoints.ts
  ↓
schemas/
  ↓
server/services/
  ↓
hooks/queries e hooks/mutations
  ↓
hooks/forms e hooks/components
  ↓
components/
  ↓
components/ui/
```

Leitura prática desse fluxo:

- endpoints definem o endereço;
- schemas definem o contrato;
- services fazem a chamada primitiva;
- hooks transformam isso em leitura/escrita reutilizável;
- components conectam dados e UI;
- `components/ui` só renderiza e recebe props/callbacks.

### Violações proibidas

- `components/ui/` chamando `useQuery`, `useMutation` ou service;
- componente fazendo `fetcher.get(...)` direto;
- mutation vivendo dentro de um componente visual;
- service disparando toast;
- hook de query recriando endpoint inline em vez de usar service;
- lógica de negócio forte dentro de `page.tsx` quando ela deveria viver na
  feature.

---

## Como decidir onde um código novo deve viver

### 1. Identifique a feature dona

Antes de criar arquivo, responda:

- esse comportamento pertence a `workflow`, `workflow-template`, `accountancy`,
  `auth`, `dashboard` ou outra feature existente?
- esse código é um detalhe da feature ou é realmente compartilhado?

Se ele tem um dono claro, ele vai para a feature.

### 2. Descubra a camada certa

Perguntas rápidas:

- é contrato de dados? `schemas/`
- é request HTTP? `server/services/`
- é tipo público da feature? `server/types/`
- é leitura do servidor? `hooks/queries/`
- é escrita do servidor? `hooks/mutations/`
- é lógica de formulário? `hooks/forms/`
- é orchestration de componente? `hooks/components/` ou `components/`
- é visual puro? `components/ui/`

### 3. Verifique se o código já existe

Antes de criar:

- procure schema existente;
- procure service existente;
- procure hook existente;
- procure `index.ts` existente;
- estenda o que já existe antes de abrir uma nova trilha paralela.

---

## Aliases do projeto

Os aliases precisam refletir o `tsconfig.json` real do projeto. Os principais são:

```ts
@societiza/*         → src/*
@shadcn/*            → src/components/ui/shadcnui/*
@components/*        → src/components/*
@features/*          → src/features/*
@workflow/*          → src/features/workflow/*
@auth/*              → src/features/auth/*
@accountancy/*       → src/features/accountancy/*
@dashboard/*         → src/features/dashboard/*
@form/*              → src/features/form/*
@workflow-template/* → src/features/workflow-template/*
```

### Regra de uso

- cruzou feature ou camada compartilhada: usar alias;
- dentro da mesma feature: usar a API pública por `index.ts` quando houver;
- import relativo longo é sintoma de arquitetura ruim.

Exemplo correto:

```ts
import { API_ENDPOINTS } from '@societiza/routes/endpoints';
import { fetcher } from '@societiza/lib/axios';
import { useAccountancyQueries } from '@accountancy/hooks/queries';
import { Button, Dialog, DialogContent } from '@shadcn/index';
```

Exemplo errado:

```ts
import { useAccountancyQueries } from '../../../features/accountancy/hooks/queries/useAccountancyQueries';
import { Button } from '../../components/ui/shadcnui/button';
```

---

## Relação com Next.js App Router

O App Router dita a estrutura de entrada da aplicação, mas **não substitui** a
arquitetura das features.

### Regras importantes

- `src/app/` é o entrypoint de rota, não o lugar para concentrar regra de
  negócio de feature;
- páginas e layouts podem ser Server Components por padrão;
- componentes com hooks, handlers e estado devem receber `'use client'`;
- a lógica relevante de domínio deve continuar na feature, não espalhada nas
  páginas.

### `params` e async APIs

No projeto atual, o padrão de App Router já considera APIs assíncronas do
Next.js moderno. Sempre que a rota usar `params` assíncronos, o código deve
respeitar isso:

```tsx
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  return <Component id={id} />;
}
```

---

## O que esta arquitetura tenta evitar

O ruleset existe para evitar os problemas mais caros do projeto:

- feature virando pasta de arquivos soltos sem dono claro;
- UI misturada com mutation e regras de negócio;
- services fazendo papel de hook;
- deep imports espalhados;
- tipos duplicados e divergentes do schema;
- páginas Next.js absorvendo responsabilidade demais;
- código novo copiando padrões ruins de áreas legadas.

---

## Resumo operacional

- arquitetura é feature-based;
- shared layers existem e são oficiais, mas são exceção e não regra;
- toda feature nova segue o contrato ideal desta pasta;
- código legado não redefine o padrão;
- fluxo padrão é:
  - endpoints
  - schemas
  - services
  - hooks
  - components
  - `components/ui`
