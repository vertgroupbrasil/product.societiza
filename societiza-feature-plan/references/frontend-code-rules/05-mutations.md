# Mutations — TanStack Query v5

## Objetivo

Toda escrita no frontend da Societiza deve passar por mutation hook da feature.
Isso inclui:

- create;
- update;
- delete;
- archive;
- publish;
- reorder;
- activate/deactivate;
- qualquer side effect que altere dado persistido.

O papel da mutation não é só chamar a API. Ela também centraliza:

- estratégia de atualização do cache;
- UI otimista quando fizer sentido;
- rollback quando necessário;
- tratamento de erro;
- mensagens de sucesso e falha;
- shape de parâmetros da operação.

---

## Regra fundamental

Mutation vive em `hooks/mutations/`.

O componente:

- pode chamar o hook;
- pode decidir quando executar;
- pode reagir ao estado (`isPending`, `isSuccess`, `isError`);

mas não deve virar o lugar onde a regra da mutation é implementada.

---

## Estrutura ideal de uma mutation

```ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { toast } from 'sonner';
import { createAccountancy } from '@accountancy/server/services';
import { accountancyQueryKeys } from '@accountancy/hooks/queries';
import type { CreateAccountancyDTO } from '@accountancy/server/types';

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

  return { create };
}
```

---

## Shape dos parâmetros

A mutation hook deve expor um shape previsível.

### Quando a operação recebe só payload

```ts
mutationFn: (data: CreateAccountancyDTO) => createAccountancy(data)
```

### Quando recebe `id` + payload

```ts
mutationFn: ({
  id,
  data,
}: {
  id: string;
  data: UpdateAccountancyDTO;
}) => updateAccountancy(id, data)
```

### Regra

- prefira um único objeto para operações compostas;
- isso facilita evolução da assinatura;
- evita ordem posicional frágil em `mutate`/`mutateAsync`.

---

## `invalidateQueries` vs `refetchQueries`

O padrão default do projeto deve ser:

- `invalidateQueries` para permitir refetch no fluxo natural do Query;
- `refetchQueries` quando a UX depender explicitamente da reconciliação imediata
  depois da mutation.

### Usar `invalidateQueries`

Quando basta marcar o dado como stale:

```ts
void queryClient.invalidateQueries({
  queryKey: accountancyQueryKeys.lists(),
});
```

### Usar `refetchQueries`

Quando a operação precisa reconciliar imediatamente com o backend:

- reorder;
- transições de estado mais sensíveis;
- fluxos em que a UI otimista não captura tudo;
- backend que recalcula coleções ou ordenação final.

```ts
await queryClient.refetchQueries({
  queryKey: workflowTemplateQueryKeys.detail(templateId),
});
```

### Regra

Não use `refetchQueries` por reflexo. Ele é mais custoso. Mas também não esconda
necessidade real de reconciliação quando a UI depende dela.

---

## UI otimista

UI otimista é recomendada quando:

- a resposta esperada é previsível;
- a latência degrada claramente a experiência;
- a ação tem rollback razoável.

Exemplo:

```ts
const reorder = useMutation({
  mutationFn: updateStepOrder,
  onMutate: async (variables) => {
    await queryClient.cancelQueries({
      queryKey: workflowTemplateQueryKeys.detail(variables.templateId),
    });

    const previous = queryClient.getQueryData(
      workflowTemplateQueryKeys.detail(variables.templateId),
    );

    queryClient.setQueryData(
      workflowTemplateQueryKeys.detail(variables.templateId),
      (current) => applyOptimisticReorder(current, variables),
    );

    return { previous };
  },
  onError: (_error, variables, context) => {
    if (context?.previous) {
      queryClient.setQueryData(
        workflowTemplateQueryKeys.detail(variables.templateId),
        context.previous,
      );
    }

    toast.error('Erro ao reordenar itens.');
  },
  onSettled: async (_data, _error, variables) => {
    await queryClient.refetchQueries({
      queryKey: workflowTemplateQueryKeys.detail(variables.templateId),
    });
  },
});
```

### Regra

Se usar UI otimista, o contrato mínimo é:

- cancelar query relevante;
- tirar snapshot;
- aplicar update otimista;
- rollback em erro;
- reconciliar no final.

---

## `mutate` vs `mutateAsync`

### `mutate`

Usar quando o componente só dispara a ação e não precisa de fluxo assíncrono
sequencial logo depois.

### `mutateAsync`

Usar quando o componente precisa coordenar:

- fechar dialog;
- navegar;
- resetar formulário;
- encadear mutation;
- executar lógica após sucesso.

Exemplo:

```ts
await create.mutateAsync(data);
onOpenChange(false);
```

### Regra

`mutateAsync` é para controle de fluxo. Não use `try/catch` redundante se a
própria mutation já centraliza feedback do erro, a menos que o componente
realmente precise tomar uma decisão adicional.

---

## Tratamento de erro

Toda mutation deve ter tratamento de erro consistente.

### Padrão mínimo

- log técnico quando útil;
- mensagem de toast clara;
- uso da mensagem do backend quando ela for confiável e amigável.

Exemplo:

```ts
onError: (error) => {
  console.error(error);

  const message =
    error instanceof Error ? error.message : 'Erro inesperado. Tente novamente.';

  toast.error(message);
},
```

### Regra

- nunca silenciar erro;
- nunca deixar `onError` vazio;
- nunca delegar todo o tratamento ao componente;
- se a feature tiver normalizador de erro da API, a mutation deve usá-lo.

---

## Toasts

Toasts de negócio pertencem à mutation ou à orchestration acima dela, nunca ao
service e nunca ao `components/ui`.

### Mensagens

- curtas;
- em português;
- específicas quando possível;
- sem jargão técnico desnecessário.

Exemplos:

```ts
toast.success('Template publicado com sucesso.');
toast.success('Alterações salvas.');
toast.error('Erro ao salvar template.');
toast.error('Não foi possível remover a contabilidade.');
```

---

## Mutations encadeadas

Quando uma ação de produto envolve mais de uma mutation, a orchestration deve
subir para `hooks/components/` ou para um hook especializado da feature.

Exemplo:

```ts
export function useTemplatePublishFlow() {
  const draft = useTemplateMutations().createDraft;
  const publish = useTemplateMutations().publish;

  async function run(templateId: string) {
    const createdDraft = await draft.mutateAsync(templateId);
    await publish.mutateAsync(createdDraft.id);
  }

  return {
    run,
    isPending: draft.isPending || publish.isPending,
  };
}
```

### Regra

Se a ação ficou grande demais para uma mutation única, não empurre esse fluxo
para o componente visual. Suba a orchestration.

---

## O que é proibido

- chamar service diretamente de `components/ui`;
- criar mutation inline no componente;
- usar mutation sem política clara de cache;
- deixar erro sem feedback;
- usar UI otimista sem rollback;
- disparar toast dentro de service;
- misturar regra de formulário dentro da mutation sem necessidade.

---

## Resumo operacional

- toda escrita vira mutation hook;
- mutation centraliza cache, feedback e semântica da operação;
- `invalidateQueries` é default, `refetchQueries` entra quando reconciliação imediata importa;
- UI otimista é encorajada quando houver ganho real de UX;
- orchestration multi-step sobe para hook próprio;
- `components/ui` não participa da implementação da mutation.
