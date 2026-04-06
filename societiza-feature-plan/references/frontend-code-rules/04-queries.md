# Queries — TanStack Query v5

## Objetivo

Queries são a camada oficial de leitura do servidor. No `web.societiza`, toda
leitura que venha de uma service da feature deve ser encapsulada em hooks de
query, com `query-options.ts` como centro de definição de chave, função e
política de cache.

Regra base:

- service faz request;
- `query-options.ts` define o contrato da query;
- hook expõe isso para o componente;
- componente consome o hook, nunca a service.

---

## `query-options.ts` é obrigatório

Toda feature que possua queries deve ter `hooks/queries/query-options.ts`.

Esse arquivo centraliza:

- query keys;
- query options;
- comportamento de `enabled`, `staleTime`, `select`, `placeholderData` quando
  fizer sentido.

Exemplo ideal:

```ts
import { queryOptions } from '@tanstack/react-query';
import {
  fetchAccountancies,
  fetchAccountancyById,
} from '@accountancy/server/services';

export const accountancyQueryKeys = {
  all: ['accountancy'] as const,
  lists: () => [...accountancyQueryKeys.all, 'list'] as const,
  details: () => [...accountancyQueryKeys.all, 'detail'] as const,
  detail: (id: string) => [...accountancyQueryKeys.details(), id] as const,
};

export const accountancyQueryOptions = {
  list: () =>
    queryOptions({
      queryKey: accountancyQueryKeys.lists(),
      queryFn: fetchAccountancies,
    }),
  detail: (id: string) =>
    queryOptions({
      queryKey: accountancyQueryKeys.detail(id),
      queryFn: () => fetchAccountancyById(id),
      enabled: Boolean(id),
    }),
};
```

### Por que isso é obrigatório

Sem esse arquivo, começam a surgir:

- query keys duplicadas ou divergentes;
- invalidações imprecisas;
- query functions recriadas em vários lugares;
- dificuldade para pré-carregamento, hidratação e refactor.

---

## Convenção de query keys

O padrão ideal é hierárquico e previsível:

```ts
export const entityQueryKeys = {
  all: ['entity'] as const,
  lists: () => [...entityQueryKeys.all, 'list'] as const,
  details: () => [...entityQueryKeys.all, 'detail'] as const,
  detail: (id: string) => [...entityQueryKeys.details(), id] as const,
};
```

### Regras

- `all` representa a família inteira;
- `lists` representa coleções;
- `details` representa o namespace de detalhe;
- `detail(id)` representa uma instância específica.

Se a feature tiver filtros importantes, o padrão pode ser expandido:

```ts
filtered: (filters: FiltersDTO) =>
  [...entityQueryKeys.lists(), 'filtered', filters] as const,
```

Mas o objeto de key precisa continuar estável e sem duplicações acidentais.

---

## Hooks de query

Os hooks de query devem ser finos. Eles não devem recriar `queryKey` nem
`queryFn`.

Exemplo:

```ts
import { useQuery } from '@tanstack/react-query';
import { accountancyQueryOptions } from './query-options';

export function useAccountancyQueries() {
  return {
    useList: () => useQuery(accountancyQueryOptions.list()),
    useDetail: (id: string) => useQuery(accountancyQueryOptions.detail(id)),
  };
}
```

### Regras

- hook de query consome `query-options.ts`;
- não chamar service inline no componente;
- não redefinir key fora do arquivo central;
- não misturar leitura com mutation dentro do mesmo hook.

---

## `enabled`

Queries dependentes de parâmetro opcional devem documentar e centralizar `enabled`
em `query-options.ts`.

```ts
detail: (id: string) =>
  queryOptions({
    queryKey: accountancyQueryKeys.detail(id),
    queryFn: () => fetchAccountancyById(id),
    enabled: Boolean(id),
  }),
```

### Regra prática

- se o componente precisa de um `id`, `slug`, filtro ou condição de autenticação
  para rodar a query, isso deve aparecer explicitamente no `queryOptions`;
- o componente não deve esconder esse comportamento em branches paralelas.

---

## Lista, detalhe e filtros

O projeto precisa separar semanticamente estes cenários:

### Query de lista

Usada para coleções principais.

```ts
list: () =>
  queryOptions({
    queryKey: entityQueryKeys.lists(),
    queryFn: fetchEntities,
  }),
```

### Query de detalhe

Usada para uma instância única.

```ts
detail: (id: string) =>
  queryOptions({
    queryKey: entityQueryKeys.detail(id),
    queryFn: () => fetchEntityById(id),
    enabled: Boolean(id),
  }),
```

### Query filtrada

Usada quando o conjunto depende diretamente de filtros do usuário.

```ts
filtered: (filters: EntityFiltersDTO) =>
  queryOptions({
    queryKey: [...entityQueryKeys.lists(), 'filtered', filters] as const,
    queryFn: () => fetchEntitiesByFilters(filters),
  }),
```

### Regra

Não empurrar toda leitura para uma única “query genérica” que varia demais de
semântica. Query diferente pede key diferente.

---

## `placeholderData`, `initialData` e `select`

Esses recursos são úteis, mas não devem ser usados como atalho para mascarar
contrato ou esconder loading de maneira arbitrária.

### `select`

Usar quando a transformação é de leitura e pertence ao cache da query.

```ts
list: () =>
  queryOptions({
    queryKey: entityQueryKeys.lists(),
    queryFn: fetchEntities,
    select: (data) => data.filter((item) => item.isActive),
  }),
```

### `placeholderData`

Usar quando a experiência se beneficia de transição suave entre listas ou filtros
sem “piscada” desnecessária.

### `initialData`

Usar quando a tela já nasce com dados fornecidos por pré-carregamento legítimo.

### Regra

- `select` transforma dado de leitura;
- `placeholderData` melhora continuidade visual;
- `initialData` é para bootstrap legítimo, não para esconder ausência de fetch.

---

## Prefetch e hidratação

Quando uma rota Server Component precisa entregar UI já hidratada com dados, use
prefetch com as mesmas `queryOptions`.

```tsx
import {
  dehydrate,
  HydrationBoundary,
  QueryClient,
} from '@tanstack/react-query';
import { accountancyQueryOptions } from '@accountancy/hooks/queries';

export default async function Page() {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery(accountancyQueryOptions.list());

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <AccountancyScreen />
    </HydrationBoundary>
  );
}
```

### Regra

Se houver prefetch, ele deve usar o mesmo contrato central de query. Nunca criar
uma versão paralela da query só para SSR.

---

## `staleTime` e ownership do cache

`staleTime` deve refletir o comportamento do domínio, não ser copiado sem pensar.

### Dados que mudam pouco

- listas administrativas;
- catálogos estáveis;
- templates pouco mutáveis.

Podem aceitar `staleTime` maior.

### Dados operacionais

- board;
- processos em andamento;
- estados muito sujeitos a mutação.

Devem usar políticas mais conservadoras.

### Regra

Não existe um único `staleTime` universal. Cada query precisa ter dono claro e
semântica de atualização clara.

---

## O que pode ser inline e o que não pode

### Pode ficar inline no hook

- `useQuery(accountancyQueryOptions.list())`;
- pequenos nomes locais da própria composição do hook.

### Não pode ficar inline

- `queryKey` reescrita manualmente;
- `queryFn` recriada do zero;
- endpoint manual;
- parsing da response fora da service;
- regra de invalidar/refetch dentro do componente.

---

## Anti-padrões

- chamar service direto em componente;
- usar string literal de query key fora de `query-options.ts`;
- criar uma key por conveniência sem hierarquia;
- esconder filtro em closure sem refletir isso na key;
- usar a query como mutation disfarçada;
- acoplar hook de query à UI com toasts de regra de negócio.

---

## Resumo operacional

- `query-options.ts` é obrigatório;
- query keys são hierárquicas e centralizadas;
- hooks de query são finos;
- componente consome hook, nunca service;
- lista, detalhe e filtro são contratos diferentes;
- prefetch, hidratação e cache sempre reaproveitam o mesmo contrato central.
