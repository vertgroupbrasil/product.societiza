# Formulários — react-hook-form + Zod

## Objetivo

Formulário no `web.societiza` não é só markup com inputs. Ele é a composição de:

- schema de request;
- `react-hook-form`;
- design system;
- mutation de submit;
- sincronização de defaults, edição e reset.

O padrão ideal é separar claramente:

- contrato do dado;
- lógica de formulário;
- UI visual do formulário;
- orchestration da tela, dialog ou sheet que usa esse formulário.

---

## Stack padrão

- `react-hook-form`
- `@hookform/resolvers/zod`
- schema Zod de request
- componentes de formulário do design system em `@shadcn/index`

### Regra principal

O schema do form é o schema de **request**, não o schema de response.

---

## Estrutura recomendada

```txt
schemas/
  accountancy.schema.ts

server/types/
  accountancy.types.ts

hooks/forms/
  useAccountancyForm.ts

components/forms/
  AccountancyForm.tsx

components/
  AccountancyFormDialog.tsx
```

Leitura dessa estrutura:

- schema define payload;
- hook de form prepara `useForm`, defaults e submit;
- form visual renderiza campos;
- dialog/sheet/container decide abertura, fechamento e contexto da tela.

---

## Formulário visual

O componente em `components/forms/` deve focar em:

- layout dos campos;
- ligação do `form.control` aos componentes visuais;
- exibição de erros;
- estados visuais de submit.

Exemplo:

```tsx
'use client';

import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
  Button,
  Input,
} from '@shadcn/index';
import type { UseFormReturn } from 'react-hook-form';
import type { CreateAccountancyDTO } from '@accountancy/server/types';

interface AccountancyFormProps {
  form: UseFormReturn<CreateAccountancyDTO>;
  onSubmit: () => void;
  isPending?: boolean;
}

export function AccountancyForm({
  form,
  onSubmit,
  isPending,
}: AccountancyFormProps) {
  return (
    <Form {...form}>
      <form onSubmit={onSubmit} className="space-y-4">
        <FormField
          control={form.control}
          name="legalName"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Razão social</FormLabel>
              <FormControl>
                <Input value={field.value} onChange={(value) => field.onChange(value)} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit" disabled={isPending}>
          {isPending ? 'Salvando...' : 'Salvar'}
        </Button>
      </form>
    </Form>
  );
}
```

### Regra

O form visual não deve ser o dono de:

- mutation;
- fetch;
- regra de negócio de submit;
- decisão de fechar dialog;
- sincronização complexa entre edição e criação.

---

## `hooks/forms/`

Quando o formulário passa do trivial, a lógica deve subir para `hooks/forms/`.

Isso inclui:

- `useForm(...)`;
- resolver Zod;
- default values;
- edição vs criação;
- integração com mutation;
- reset/sync quando dados chegam;
- transformação de payload.

Exemplo:

```ts
import { useEffect } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { createAccountancySchemaDTO } from '@accountancy/schemas';
import { useAccountancyMutations } from '@accountancy/hooks/mutations';
import type {
  Accountancy,
  CreateAccountancyDTO,
  UpdateAccountancyDTO,
} from '@accountancy/server/types';

interface UseAccountancyFormProps {
  accountancy?: Accountancy;
  onSuccess?: () => void;
}

export function useAccountancyForm({
  accountancy,
  onSuccess,
}: UseAccountancyFormProps) {
  const { create, update } = useAccountancyMutations();

  const form = useForm<CreateAccountancyDTO>({
    resolver: zodResolver(createAccountancySchemaDTO),
    defaultValues: {
      legalName: '',
      tradeName: '',
      cnpj: '',
      email: '',
      phone: '',
    },
  });

  useEffect(() => {
    if (!accountancy) return;

    form.reset({
      legalName: accountancy.legalName,
      tradeName: accountancy.tradeName ?? '',
      cnpj: accountancy.cnpj,
      email: accountancy.email ?? '',
      phone: accountancy.phone ?? '',
    });
  }, [accountancy, form]);

  const isEditing = Boolean(accountancy);

  const handleSubmit = form.handleSubmit(async (data) => {
    if (isEditing && accountancy) {
      await update.mutateAsync({
        id: accountancy.id,
        data: data as UpdateAccountancyDTO,
      });
    } else {
      await create.mutateAsync(data);
    }

    onSuccess?.();
  });

  return {
    form,
    handleSubmit,
    isEditing,
    isPending: create.isPending || update.isPending,
  };
}
```

### Regra

Se o formulário:

- cresce;
- reaparece em mais de um componente;
- tem lógica de edição;
- faz transformação relevante;
- depende de mutation;

então ele deve sair do componente e ir para `hooks/forms/`.

---

## `defaultValues`

`defaultValues` devem sempre ser explicitamente definidos para evitar campo
controlado virando `undefined`.

```ts
defaultValues: {
  legalName: '',
  tradeName: '',
  cnpj: '',
  email: '',
  phone: '',
}
```

### Regra

- para string, prefira string vazia;
- para arrays, prefira array vazio quando fizer sentido;
- para edição assíncrona, use `reset()` quando os dados chegarem;
- não faça `setValue` campo por campo sem necessidade.

---

## Edição e sincronização com dados do servidor

Quando o formulário abre em modo de edição com dados que chegam depois, use
`reset()`.

```ts
useEffect(() => {
  if (!entity) return;

  form.reset({
    name: entity.name,
    description: entity.description ?? '',
  });
}, [entity, form]);
```

### Regra

`reset()` é a forma preferida porque:

- sincroniza o estado inteiro do form;
- limpa dirty state de forma coerente;
- evita drift entre campos.

---

## Integração com o Input custom do projeto

O design system local pode expor `Input` com assinatura adaptada e suporte a
máscara. Não assuma `e.target.value` sem verificar o componente real.

Exemplo:

```tsx
<Input
  mask="cnpj"
  value={field.value}
  onChange={(_maskedValue, unmaskedValue) => {
    field.onChange(unmaskedValue);
  }}
/>
```

### Regra

Se o form depende de valor sem máscara, isso precisa ficar explícito.

---

## `components/forms/` vs `components/ui/`

`components/forms/` não é sinônimo de “componente super inteligente”. O objetivo
dele é ser a montagem visual do formulário.

Ele pode receber:

- `form`;
- `onSubmit`;
- `isPending`;
- props de variação visual.

Mas não deve assumir o papel de:

- mutation hook owner;
- dialog owner;
- regra de abertura/fechamento;
- fetch owner.

---

## Regra sobre “formulário” no domínio do produto

No contexto de Societiza, é importante não confundir duas coisas:

- **formulário técnico de UI** desta camada, usado para criar/editar dados;
- uma futura **feature de produto** de formulário de abertura de empresa.

Hoje, no MVP, o que existe operacionalmente no workflow template são campos
configuráveis de etapas e tarefas. Isso não deve ser documentado como se o
produto já tivesse um módulo formal de formulário de abertura entregue.

---

## Anti-padrões

- criar `useForm()` dentro de `components/ui/`;
- chamar mutation diretamente no form visual;
- usar schema de response para validar submit;
- deixar form sem `defaultValues`;
- usar `e.target.value` cegamente em componente de input customizado;
- misturar dialog, mutation, lógica de sucesso e UI do form no mesmo arquivo.

---

## Resumo operacional

- schema de request guia o form;
- `hooks/forms/` concentra lógica crescente;
- `components/forms/` renderiza o formulário;
- `components/` orquestra dialog, sheet, screen section e fluxo externo;
- `reset()` é a forma padrão de sincronizar edição;
- “formulário de UI” não deve ser confundido com futura feature de produto.
