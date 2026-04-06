# TypeScript e Schemas — Societiza

## Objetivo

TypeScript e Zod sustentam o contrato de dados do frontend. Neste projeto, o
schema não é acessório; ele é a fonte de verdade do formato da API e do payload
que o frontend envia.

Regra central:

- sempre que existir schema, o tipo nasce dele;
- tipos manuais só entram quando forem composição legítima e não duplicação de
  contrato.

---

## Base técnica do projeto

O projeto roda TypeScript em modo estrito. O `tsconfig.json` atual torna estas
flags especialmente relevantes:

- `strict: true`
- `noImplicitAny: true`
- `noUnusedLocals: true`
- `exactOptionalPropertyTypes: true`
- `moduleResolution: "bundler"`

Isso tem consequências reais:

- `any` implícito quebra;
- variável local não usada quebra build;
- opcional e `undefined` não são a mesma coisa;
- imports precisam respeitar os aliases e a resolução moderna do projeto.

---

## Schema como fonte de verdade

### Schemas de response

Schemas de response modelam o que a API realmente devolve.

```ts
import { z } from 'zod';

export const workflowTemplateDetailSchema = z.object({
  id: z.string().uuid(),
  name: z.string(),
  description: z.string(),
  version: z.string(),
  status: z.enum(['Draft', 'Active', 'Archived']),
  createdAt: z.coerce.date(),
  updatedAt: z.coerce.date().nullable(),
  steps: z.array(workflowTemplateStepSchema),
});
```

### Schemas de request

Schemas de request modelam o payload que o frontend envia.

```ts
export const createTemplateSchemaDTO = z.object({
  name: z.string().min(1, 'Nome é obrigatório'),
  description: z.string().min(1, 'Descrição é obrigatória'),
});

export const updateTemplateSchemaDTO = createTemplateSchemaDTO;
```

### Tipos inferidos

```ts
import type { z } from 'zod';
import type {
  workflowTemplateDetailSchema,
  createTemplateSchemaDTO,
} from '@workflow-template/schemas';

export type WorkflowTemplateDetail = z.infer<
  typeof workflowTemplateDetailSchema
>;
export type CreateTemplateDTO = z.infer<typeof createTemplateSchemaDTO>;
```

### O que evitar

```ts
// Evitar duplicar contrato que já existe no schema
interface WorkflowTemplateDetail {
  id: string;
  name: string;
}
```

---

## DTO-first por entidade

O padrão ideal é separar claramente:

- schema de entidade de response;
- schema de create DTO;
- schema de update DTO;
- eventualmente schema de filtros/query params quando a feature exigir.

Exemplo:

```ts
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

---

## Nulidade, optional e campo omitido

Frontend costuma quebrar integração quando trata `null`, `undefined` e ausência
de campo como se fossem a mesma coisa. Não são.

| Semântica | Schema certo |
|---|---|
| Campo sempre presente, pode ser `null` | `z.string().nullable()` |
| Campo pode não vir | `z.string().optional()` |
| Campo pode não vir ou ser `null` | `z.string().nullish()` |
| Campo sempre presente e nunca `null` | `z.string()` |

### Regra prática

- se o backend devolve explicitamente `null`, usar `.nullable()`;
- se o backend omite a propriedade, usar `.optional()`;
- se o contrato aceita ambos, usar `.nullish()`;
- não use `.optional()` para “simular null”.

Exemplo:

```ts
tradeName: z.string().nullable(),
updatedAt: z.coerce.date().nullable(),
description: z.string().optional(),
```

---

## Datas

O padrão preferido do projeto é parsear datas no schema quando o frontend quer
trabalhar com `Date` de verdade.

```ts
createdAt: z.coerce.date(),
updatedAt: z.coerce.date().nullable(),
```

### Quando isso é correto

- quando a API devolve ISO string;
- quando o resto da feature precisa operar com `Date`;
- quando a transformação é estável e previsível.

### Quando tomar cuidado

- se o campo pode vir vazio, `null` ou formato inconsistente;
- se a feature só precisa exibir a string original;
- se o backend ainda não tem contrato estável para o campo.

---

## Enums e valores fechados

Sempre que o contrato tiver um conjunto fechado de valores, preferir `z.enum()`.

```ts
export const templateStatusEnum = z.enum(['Draft', 'Active', 'Archived']);
export const fieldTypeEnum = z.enum(['Text', 'Select']);
```

### Regra importante

Só documente como enum fechado aquilo que o contrato realmente estabilizou.
Quando a API ainda aceita legado ou valores abertos, não finja fechamento que o
backend não garante.

Por isso, em alguns domínios do projeto:

- `workflow-template` tem enums mais estáveis;
- `workflow` legado ainda exige mais cuidado porque o contrato é heterogêneo.

---

## Transformações e parsing defensivo

Schemas podem transformar dados, mas a transformação precisa ser segura e
determinística.

Exemplo:

```ts
options: z
  .string()
  .transform((value) => {
    try {
      return JSON.parse(value) as string[];
    } catch {
      return [];
    }
  }),
```

### Regras

- só transforme no schema quando isso for parte estável do contrato;
- se a transformação puder mascarar erro grave de backend, documente isso;
- `.catch(...)` não deve virar desculpa para engolir contrato quebrado sem
  critério.

---

## `server/types/` como API pública de tipos

Mesmo com schema sendo a fonte da verdade, a feature não deve obrigar todo
consumidor a importar schemas crus. O padrão ideal é:

- schema fica em `schemas/`;
- type público fica em `server/types/`.

Isso melhora:

- ergonomia de import;
- separação entre contrato e uso;
- previsibilidade da API pública da feature.

Exemplo:

```ts
import type {
  WorkflowTemplateDetail,
  CreateTemplateDTO,
} from '@workflow-template/server/types';
```

---

## `exactOptionalPropertyTypes`

Esta flag muda bastante a modelagem de payloads.

```ts
interface Input {
  description?: string;
}

const invalid: Input = {
  description: undefined,
};
```

O padrão preferido é omitir a propriedade quando ela não existir:

```ts
const data = {
  name,
  ...(description ? { description } : {}),
};
```

### Regra prática

- não preencher objeto com `undefined` “só para completar”;
- omitir a chave quando o contrato permitir ausência;
- usar `null` apenas quando o contrato realmente espera `null`.

---

## `noUnusedLocals`

`noUnusedLocals: true` quebra build para variáveis locais não usadas. Isso vale
para helpers, destructuring e constantes intermediárias.

O ESLint também trata argumentos não usados com tolerância a prefixo `_`, então
o padrão recomendado é:

```ts
function handleError(_error: unknown) {
  toast.error('Erro inesperado.');
}
```

---

## Schemas aninhados

O padrão ideal é construir do menor para o maior.

```ts
export const workflowTemplateFieldSchema = z.object({
  id: z.string().uuid(),
  label: z.string(),
  fieldType: fieldTypeEnum,
});

export const workflowTemplateTaskSchema = z.object({
  id: z.string().uuid(),
  title: z.string(),
  fields: z.array(workflowTemplateFieldSchema),
});

export const workflowTemplateStepSchema = z.object({
  id: z.string().uuid(),
  tasks: z.array(workflowTemplateTaskSchema),
});
```

Isso facilita:

- reutilização;
- legibilidade;
- composição de tipos;
- testes isolados por entidade.

---

## `parse` vs `safeParse`

### `parse`

Usar quando um contrato deveria ser válido e falha indica problema real.

```ts
return workflowTemplateDetailSchema.parse(response.data);
```

### `safeParse`

Usar quando a invalidade é esperada, controlável ou faz parte de uma validação
exploratória.

```ts
const result = createTemplateSchemaDTO.safeParse(data);

if (!result.success) {
  return { success: false, issues: result.error.issues };
}
```

Regra:

- service costuma preferir `parse`;
- validação local de formulário pode usar `safeParse` quando necessário.

---

## Input custom do design system

O `Input` custom do projeto não deve ser tratado como input HTML puro em todos
os casos. Em partes da base, ele expõe `onChange` adaptado com valor formatado e
`unmaskedValue`.

Portanto, a documentação da feature deve sempre conferir o contrato do componente
de UI usado antes de assumir `e.target.value`.

Exemplo compatível com o padrão local:

```tsx
<Input
  value={field.value}
  onChange={(value, unmaskedValue) => {
    field.onChange(unmaskedValue);
  }}
/>
```

### Regra

Isso é uma particularidade do design system da Societiza, não uma regra geral de
React. Sempre documentar isso como comportamento do componente local, não como
API universal do DOM.

---

## Resumo operacional

- schema é fonte de verdade;
- `server/types/` é a superfície pública dos tipos;
- datas, nullabilidade e enums precisam ser modelados com rigor;
- `exactOptionalPropertyTypes` exige omitir chave em vez de “preencher com undefined”;
- transformações em schema devem ser seguras e conscientes;
- o contrato do design system local precisa ser respeitado ao integrar forms.
