# Imports e Barrels

## Objetivo

Este projeto trata import como parte da arquitetura. O import define:

- qual camada pode conhecer qual outra camada;
- qual é a API pública de uma feature;
- e quais detalhes são internos.

Regra central:

**todo import interno deve sair de um `index.ts` público quando esse barrel já
existir.**

---

## Superfície pública vs detalhe interno

Cada pasta relevante da feature deve ter uma superfície pública clara exposta por
`index.ts`.

Exemplos:

```txt
src/features/accountancy/index.ts
src/features/accountancy/hooks/index.ts
src/features/accountancy/hooks/queries/index.ts
src/features/accountancy/server/index.ts
src/features/accountancy/server/services/index.ts
src/features/accountancy/components/ui/index.ts
```

### Regra

- se um módulo é consumido por fora da pasta, ele deve ser exposto por barrel;
- se já existe barrel, deep import fica proibido;
- se ainda não existe barrel e a pasta passou a ser consumida, criar o barrel.

---

## Ordem de preferência dos imports

### 1. Alias público da feature ou shared layer

```ts
import { API_ENDPOINTS } from '@societiza/routes/endpoints';
import { fetcher } from '@societiza/lib/axios';
import { useAccountancyQueries } from '@accountancy/hooks/queries';
```

### 2. Alias de subpasta da própria feature

Dentro da feature, imports que cruzam diretórios devem usar o alias da feature:

```ts
// ✅ certo
import { loginSchema } from '@auth/schemas';
import { useLoginMutation } from '@auth/hooks/mutations';
import { fetchCurrentUser } from '@auth/server/services';

// ❌ errado — caminho relativo cruzando diretório
import { loginSchema } from '../../schemas/auth.schema';
import { useLoginMutation } from '../mutations/use-login-mutation';
import { fetchCurrentUser } from '../../server/services/current-user.service';
```

### 3. Import relativo do mesmo diretório

A única exceção válida para caminho relativo dentro de uma feature é quando o
import está no **mesmo diretório**:

```ts
// ✅ aceitável — mesmo diretório
import { currentUserQueryOptions } from './query-options';
```

### 4. Arquivo direto

Só quando o barrel causaria dependência circular real e comprovada. Não como
atalho por conveniência.

---

## Aliases reais do projeto

Os aliases relevantes hoje são:

```txt
@societiza/*
@shadcn/*
@components/*
@features/*
@workflow/*
@auth/*
@accountancy/*
@dashboard/*
@form/*
@workflow-template/*
```

### Regra

- entre features: usar alias;
- para shared layers: usar alias;
- dentro da própria feature: usar barrel local quando existir;
- caminho relativo longo cruzando camadas ou features é anti-padrão.

---

## Regras de barrel

### Quando criar barrel

**Barrel não é opcional.** Criar `index.ts` para toda subpasta da feature no
momento da criação, não retroativamente.

A entrega de uma feature só está completa quando cada subpasta tem seu barrel.

Subpastas que sempre exigem barrel:

```txt
schemas/index.ts
server/services/index.ts
hooks/queries/index.ts
hooks/mutations/index.ts
hooks/forms/index.ts
components/forms/index.ts
components/ui/index.ts
components/index.ts
```

O `index.ts` raiz da feature deve re-exportar dos barrels das subpastas, não
dos arquivos individuais. Importar arquivo individual no barrel raiz é anti-padrão.

### O que exportar

Exportar:

- hooks públicos;
- types públicos;
- schemas públicos quando fizer sentido;
- components públicos;
- constants públicas;
- services quando a própria arquitetura da feature precisar disso.

Não exportar:

- helpers privados sem motivo;
- detalhes de implementação só porque “é mais fácil”;
- arquivos experimentais;
- qualquer coisa que aumente acoplamento sem necessidade.

---

## Import por “um mesmo index”

O padrão ideal é manter consistência de origem no arquivo. Isso significa:

- se a subpasta já expõe um barrel, importar dela;
- preferir o mesmo ponto de entrada público para o conjunto de módulos da mesma
  camada;
- evitar misturar no mesmo arquivo uma metade vindo de barrel e outra metade
  vindo de deep import da mesma área.

Exemplo ideal:

```ts
import {
  Button,
  Dialog,
  DialogContent,
  Input,
  Sheet,
  SheetContent,
} from '@shadcn/index';

import { useTemplateQueries } from '@workflow-template/hooks/queries';
import { useTemplateMutations } from '@workflow-template/hooks/mutations';
import type { WorkflowTemplateDetail } from '@workflow-template/server/types';
```

Exemplo ruim:

```ts
import { Button } from '@shadcn/button';
import { Input } from '@shadcn/index';
import { Dialog } from '../../components/ui/shadcnui/dialog';
```

### Regra

Dentro do possível, um mesmo domínio de imports deve vir do mesmo barrel público.

---

## Ordem dos blocos de import

Padrão recomendado:

1. bibliotecas externas;
2. Next.js;
3. aliases compartilhados do projeto;
4. aliases de feature;
5. barrels relativos internos;
6. types com `import type` seguindo o mesmo agrupamento lógico.

Exemplo:

```ts
import { useMemo, useState } from 'react';
import { useRouter } from 'next/navigation';
import { useQueryClient } from '@tanstack/react-query';
import { toast } from 'sonner';

import { API_ENDPOINTS } from '@societiza/routes/endpoints';
import {
  Button,
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from '@shadcn/index';

import { useAccountancyQueries } from '@accountancy/hooks/queries';
import { useAccountancyMutations } from '@accountancy/hooks/mutations';
import type { Accountancy } from '@accountancy/server/types';

import { ACCOUNTANCY_STATUS_LABELS } from '../constants';
```

---

## `import type`

Usar `import type` sempre que o import existir apenas para tipagem.

```ts
import type { Accountancy } from '@accountancy/server/types';
import type { TemplateStatus } from '@workflow-template/server/types';
```

### Regra

- melhora clareza;
- evita carga de runtime desnecessária;
- deixa explícito o que é valor e o que é apenas tipo.

---

## Dependência entre camadas

O import também precisa respeitar a direção da arquitetura:

```txt
schemas
  ↓
server/services
  ↓
hooks
  ↓
components
  ↓
components/ui
```

### Consequências

- `components/ui` não importa `hooks`;
- `server/services` não importa `components`;
- `schemas` não importa `hooks` nem `components`;
- mutation hook pode importar query keys, mas não componente;
- componente pode importar hook e UI, mas não deveria importar service direto.

---

## Exceções válidas

Deep import só é aceitável quando:

- ainda não existe barrel na pasta;
- o barrel causaria dependência circular real;
- o arquivo está estritamente dentro do mesmo contexto privado e a API pública
  ainda não foi estabelecida.

Mesmo nessas situações, a preferência é resolver a arquitetura em vez de
normalizar o deep import como padrão.

---

## Anti-padrões

- alias errado para caminho que já tem barrel público;
- misturar `@shadcn/index`, `@shadcn/button` e path relativo no mesmo arquivo;
- importar direto de arquivo privado quando a pasta já expõe index;
- criar barrel gigante que exporta tudo sem critério;
- usar import relativo longo para cruzar feature;
- criar arquivos de uma camada sem criar o barrel da pasta na mesma entrega;
- usar `../subpasta/arquivo` quando existe barrel `@feature/subpasta`;
- re-exportar no `index.ts` raiz importando arquivo individual em vez de barrel da subpasta:

```ts
// ❌ anti-padrão — bypassa barrel da subpasta
export { useCurrentUser } from './hooks/queries/use-auth-queries';

// ✅ correto — usa barrel da subpasta
export { useCurrentUser } from './hooks/queries';
```

---

## Resumo operacional

- **barrel é entrega, não dívida** — toda subpasta criada sai com seu `index.ts`;
- imports internos nunca cruzam diretórios via caminho relativo;
- dentro da feature, imports entre diretórios usam `@feature/subpasta`;
- a única exceção para caminho relativo é import no mesmo diretório (`./arquivo`);
- a API pública da feature deve ser explícita no `index.ts` raiz;
- o `index.ts` raiz re-exporta dos barrels das subpastas, não dos arquivos;
- aliases são obrigatórios entre features e shared layers;
- deep import é exceção comprovada por circular dependency, não padrão.
