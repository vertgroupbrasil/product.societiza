# Como Criar uma Feature do Zero — Societiza

## Objetivo

Este documento define a ordem ideal para criar uma feature frontend nova no
`web.societiza`. A intenção é evitar duas falhas comuns:

- criar interface sem contrato de dados claro;
- criar camadas fora de ordem e depois tentar “encaixar” schemas, services,
  hooks e documentação.

O processo correto é **de baixo para cima**, começando pelo contrato e só depois
chegando na interface.

---

## Quando realmente criar uma feature nova

Criar uma pasta nova em `src/features/` somente quando:

- o domínio tem identidade própria no produto;
- a feature tem fluxo, contrato e responsabilidades próprios;
- o código novo não é uma extensão natural de uma feature existente.

Exemplos legítimos:

- `accountancy`
- `workflow`
- `workflow-template`
- `dashboard`

Exemplos que normalmente **não** justificam nova feature:

- um dialog adicional de `workflow`;
- um bloco visual novo de `accountancy`;
- uma mutation extra de `auth`;
- um utilitário que pertence claramente a uma feature já existente.

Se o domínio já existe, estenda a feature atual.

---

## Sequência obrigatória de criação

### 1. Registrar endpoints em `src/routes/endpoints.ts`

Toda feature integrada à API começa pelo registro central de endpoints.

O padrão real do projeto hoje usa:

- `NEXT_PUBLIC_API_URL`
- helper `withBase(...)`
- agrupamento em `API_ENDPOINTS`

Exemplo alinhado ao projeto:

```ts
const api = process.env.NEXT_PUBLIC_API_URL;

const withBase = (endpoint = '') => `${api}${endpoint}`;

export const API_ENDPOINTS = {
  accountancy: {
    create: withBase('/accountancy'),
    getAll: withBase('/accountancy'),
    getById: (id: string) => withBase(`/accountancy/${id}`),
    update: (id: string) => withBase(`/accountancy/${id}`),
    delete: (id: string) => withBase(`/accountancy/${id}`),
  },
};
```

#### Regras

- rotas parametrizadas são funções;
- não hardcodar endpoint dentro de service;
- endpoint representa endereço, não regra de negócio;
- manter agrupamento por domínio/feature.

### 2. Criar schemas em `schemas/`

Schemas vêm antes dos tipos porque são a fonte de verdade do contrato.

Exemplo:

```ts
import { z } from 'zod';

export const accountancySchema = z.object({
  id: z.string().uuid(),
  legalName: z.string(),
  tradeName: z.string().nullable(),
  cnpj: z.string(),
  email: z.string().email().nullable(),
  phone: z.string().nullable(),
  createdAt: z.coerce.date(),
  updatedAt: z.coerce.date().nullable(),
});

export const createAccountancySchemaDTO = z.object({
  legalName: z.string().min(1, 'Razão social é obrigatória'),
  tradeName: z.string().optional(),
  cnpj: z.string().length(14, 'CNPJ deve ter 14 dígitos'),
  email: z.string().email('E-mail inválido').optional(),
  phone: z.string().optional(),
});

export const updateAccountancySchemaDTO =
  createAccountancySchemaDTO.partial();
```

#### Regras

- um arquivo por entidade ou DTO relevante;
- nomear schemas de request com sufixo `DTO` quando o schema representar o
  payload enviado/recebido;
- distinguir response schema de create/update schema;
- modelar `null`, `optional` e datas com precisão;
- não duplicar contrato em interface manual.

### 3. Expor tipos em `server/types/`

`server/types/` é a API pública de tipos da feature. Os tipos devem ser
inferidos dos schemas.

```ts
import type { z } from 'zod';
import type {
  accountancySchema,
  createAccountancySchemaDTO,
  updateAccountancySchemaDTO,
} from '@accountancy/schemas';

export type Accountancy = z.infer<typeof accountancySchema>;
export type CreateAccountancyDTO = z.infer<typeof createAccountancySchemaDTO>;
export type UpdateAccountancyDTO = z.infer<typeof updateAccountancySchemaDTO>;
```

#### Regras

- preferir `type` inferido, não `interface` duplicada;
- usar `server/types/` como superfície pública para o resto da feature;
- evitar importar schema bruto em toda a árvore quando o tipo público já existe.

### 4. Criar `server/services/`

Service é a camada HTTP mais primitiva da feature.

O padrão real do projeto hoje usa `fetcher` de `@societiza/lib/axios`.

```ts
import { fetcher } from '@societiza/lib/axios';
import { API_ENDPOINTS } from '@societiza/routes/endpoints';
import { accountancyListSchema, accountancySchema } from '@accountancy/schemas';
import type {
  CreateAccountancyDTO,
  UpdateAccountancyDTO,
} from '@accountancy/server/types';

export async function fetchAccountancies() {
  const response = await fetcher.get(API_ENDPOINTS.accountancy.getAll);
  return accountancyListSchema.parse(response.data);
}

export async function fetchAccountancyById(id: string) {
  const response = await fetcher.get(API_ENDPOINTS.accountancy.getById(id));
  return accountancySchema.parse(response.data);
}

export async function createAccountancy(data: CreateAccountancyDTO) {
  const response = await fetcher.post(API_ENDPOINTS.accountancy.create, data);
  return accountancySchema.parse(response.data);
}

export async function updateAccountancy(
  id: string,
  data: UpdateAccountancyDTO,
) {
  await fetcher.put(API_ENDPOINTS.accountancy.update(id), data);
}

export async function deleteAccountancy(id: string) {
  await fetcher.delete(API_ENDPOINTS.accountancy.delete(id));
}
```

#### Regras

- service faz request e parse, só isso;
- sem toast;
- sem query cache;
- sem hook de React;
- sem try/catch genérico apenas para logar;
- service pode retornar entidade, lista, `void` ou envelope parseado, desde que
  isso seja fiel ao backend real.

### 5. Criar `hooks/queries/query-options.ts`

Toda feature com leitura deve centralizar query keys e query options.

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

### 6. Criar hooks de query em `hooks/queries/`

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

O nome exato do hook pode variar por ergonomia da feature, mas o padrão ideal é:

- query options separadas;
- hooks finos;
- nenhuma duplicação de key/fn fora de `query-options.ts`.

### 7. Criar hooks de mutation em `hooks/mutations/`

```ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { toast } from 'sonner';
import {
  createAccountancy,
  updateAccountancy,
} from '@accountancy/server/services';
import { accountancyQueryKeys } from '@accountancy/hooks/queries';
import type {
  CreateAccountancyDTO,
  UpdateAccountancyDTO,
} from '@accountancy/server/types';

export function useAccountancyMutations() {
  const queryClient = useQueryClient();

  const create = useMutation({
    mutationFn: (data: CreateAccountancyDTO) => createAccountancy(data),
    onSuccess: () => {
      void queryClient.invalidateQueries({
        queryKey: accountancyQueryKeys.lists(),
      });
      toast.success('Contabilidade criada com sucesso.');
    },
    onError: (error) => {
      console.error(error);
      toast.error('Erro ao criar contabilidade.');
    },
  });

  const update = useMutation({
    mutationFn: ({
      id,
      data,
    }: {
      id: string;
      data: UpdateAccountancyDTO;
    }) => updateAccountancy(id, data),
    onSuccess: (_, variables) => {
      void queryClient.invalidateQueries({
        queryKey: accountancyQueryKeys.lists(),
      });
      void queryClient.invalidateQueries({
        queryKey: accountancyQueryKeys.detail(variables.id),
      });
      toast.success('Contabilidade atualizada.');
    },
    onError: (error) => {
      console.error(error);
      toast.error('Erro ao atualizar contabilidade.');
    },
  });

  return { create, update };
}
```

### 8. Criar hooks de formulário ou orchestration quando necessário

Se a feature tiver form ou fluxo complexo, criar:

- `hooks/forms/` para submit, default values, reset e integração com mutation;
- `hooks/components/` para orchestration de componente que ficou complexa demais.

### 9. Só então criar componentes

Ordem recomendada:

1. `components/ui/` para blocos visuais puros;
2. `components/forms/` para form visual;
3. `components/` para dialog, sheet, card container, screen section e afins.

### 10. Criar barrel `index.ts` em todas as subpastas

**Esta etapa é obrigatória e não opcional.** Toda subpasta que foi criada na
feature deve ter um `index.ts` que expõe a superfície pública dessa camada.

Barrels obrigatórios em toda feature completa:

```txt
src/features/{feature}/schemas/index.ts
src/features/{feature}/server/services/index.ts
src/features/{feature}/hooks/queries/index.ts
src/features/{feature}/hooks/mutations/index.ts
src/features/{feature}/hooks/forms/index.ts       ← se a pasta existir
src/features/{feature}/components/forms/index.ts  ← se a pasta existir
src/features/{feature}/components/ui/index.ts     ← se a pasta existir
src/features/{feature}/components/index.ts
src/features/{feature}/index.ts                   ← API pública da feature
```

O `index.ts` da feature raiz deve re-exportar dos barrels das subpastas, não
dos arquivos individuais:

```ts
// ✅ certo — re-exporta do barrel da subpasta
export { useCurrentUser } from './hooks/queries';
export { LoginForm } from './components/forms';

// ❌ errado — importa arquivo individual, bypassando o barrel
export { useCurrentUser } from './hooks/queries/use-auth-queries';
export { LoginForm } from './components/forms/login-form';
```

### 11. Criar documentação da feature

Toda feature relevante criada do zero deve sair com:

- `src/features/{feature}/README.md`
- `docs/features/tech/{date-or-feature}.md`

Sem isso, a feature nasce sem contexto institucional.

---

## Ordem resumida

```txt
1. endpoints
2. schemas
3. server/types
4. server/services
5. hooks/queries/query-options
6. hooks/queries
7. hooks/mutations
8. hooks/forms / hooks/components
9. components
10. barrels index.ts (todas as subpastas + feature raiz)
11. README + docs/features/tech
```

---

## Erros mais comuns ao criar feature

- começar pelo componente e só depois descobrir contrato de dados;
- criar request inline no componente;
- pular `query-options.ts`;
- usar schema como tipo público em tudo sem organizar `server/types/`;
- criar feature nova para algo que já pertence a uma feature existente;
- esquecer documentação;
- inventar endpoint local em vez de registrar em `src/routes/endpoints.ts`;
- criar arquivos das camadas sem criar os barrels correspondentes — barrels não
  são etapa opcional que vira dívida técnica, são parte da entrega da feature;
- usar caminhos relativos cruzando diretórios (`../mutations/file`) quando o
  barrel da subpasta já existe ou está sendo criado agora.

---

## Regra final

Uma feature nova só está realmente criada quando:

- tem contrato de dados;
- tem fluxo de leitura/escrita organizado;
- tem UI ligada às camadas corretas;
- **tem barrel `index.ts` em toda subpasta criada**;
- **todos os imports internos usam `@feature/*` ou o barrel da subpasta**;
- tem documentação suficiente para outra pessoa continuar o trabalho sem adivinhar.
