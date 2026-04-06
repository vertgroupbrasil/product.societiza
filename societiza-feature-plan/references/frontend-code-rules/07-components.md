# Componentes — Separação entre UI e Orchestration

## Objetivo

Este documento define uma das regras mais importantes do frontend da Societiza:

**componente visual não é lugar de regra de negócio.**

O projeto separa componentes em camadas diferentes para evitar que UI se torne o
centro de:

- mutation;
- query;
- navegação contextual;
- validação de domínio;
- reorder;
- publish;
- delete;
- fluxos assíncronos complexos.

---

## As três camadas de componente

### 1. `components/ui/`

Camada visual pura.

### 2. `components/forms/`

Camada visual de formulário da feature.

### 3. `components/`

Camada de composição e orchestration entre hooks, contexts e UI.

Quando necessário, `hooks/components/` entra como apoio para retirar orchestration
de um componente que cresceu demais.

---

## `components/ui/`

`components/ui/` existe para:

- layout;
- composição visual;
- texto;
- estado visual local;
- microcomportamento de interface.

### Permitido

- `useState`, `useEffect`, `useMemo` para estado visual local;
- hover, expand/collapse, tab visual, accordion visual;
- callbacks recebidos por props;
- renderização condicional baseada em props;
- markup, classes, variants e composição do design system.

Exemplo:

```tsx
interface ProcessCardProps {
  title: string;
  statusLabel: string;
  isDeleting?: boolean;
  onEdit: () => void;
  onDelete: () => void;
}

export function ProcessCard({
  title,
  statusLabel,
  isDeleting,
  onEdit,
  onDelete,
}: ProcessCardProps) {
  const [isHovered, setIsHovered] = useState(false);

  return (
    <Card
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <CardHeader>
        <span>{title}</span>
        <Badge>{statusLabel}</Badge>
      </CardHeader>

      {isHovered ? (
        <CardFooter>
          <Button onClick={onEdit}>Editar</Button>
          <Button disabled={isDeleting} onClick={onDelete}>
            Remover
          </Button>
        </CardFooter>
      ) : null}
    </Card>
  );
}
```

### Proibido

- `useQuery`;
- `useMutation`;
- service call;
- `fetcher`;
- toast de regra de negócio;
- handler que valida regra de domínio e faz `mutateAsync`;
- receber só um `id` e buscar dado internamente;
- decidir publish/archive/delete por conta própria.

Exemplo proibido:

```tsx
export function TemplateCard({ templateId }: { templateId: string }) {
  const { data } = useTemplateQueries().useDetail(templateId);
  const { publish } = useTemplateMutations();

  async function handlePublish() {
    await publish.mutateAsync(templateId);
    toast.success('Template publicado.');
  }

  return <div>{data?.name}</div>;
}
```

Esse tipo de código não pertence a `components/ui/`.

---

## `components/`

`components/` é a camada que liga:

- query hooks;
- mutation hooks;
- hooks/forms;
- hooks/components;
- contexts;
- router;
- e UI visual.

Exemplo:

```tsx
import { useTemplateQueries } from '@workflow-template/hooks/queries';
import { useTemplateMutations } from '@workflow-template/hooks/mutations';
import { TemplateCard } from './ui';

export function TemplateCardContainer({
  templateId,
}: {
  templateId: string;
}) {
  const { useDetail } = useTemplateQueries();
  const { archive } = useTemplateMutations();
  const { data: template } = useDetail(templateId);

  if (!template) return null;

  return (
    <TemplateCard
      title={template.name}
      statusLabel={template.status}
      onEdit={() => {
        /* navegação ou abertura */
      }}
      onDelete={() => {
        void archive.mutateAsync(template.id);
      }}
      isDeleting={archive.isPending}
    />
  );
}
```

### Responsabilidade

`components/` pode:

- consumir hooks;
- preparar props derivadas;
- converter contrato bruto em contrato visual;
- coordenar abertura/fechamento;
- acionar navegação;
- lidar com loading, empty state e error state.

Mas deve evitar virar um monólito de 500 linhas. Quando isso acontecer, mover
parte da orchestration para `hooks/components/`.

---

## `hooks/components/`

Esta camada é o lugar correto para orchestration quando:

- o componente precisa coordenar muitas mutations;
- existe reorder;
- há lógica de publish/archive/save complexa;
- há muitos handlers;
- o componente tem muito estado local misturado com estado de servidor.

Exemplo:

```ts
export function useTemplateColumn(stepId: string) {
  const { useDetail } = useTemplateQueries();
  const { updateStep, removeStep } = useTemplateMutations();
  const [isSheetOpen, setIsSheetOpen] = useState(false);

  function handleOpenSheet() {
    setIsSheetOpen(true);
  }

  async function handleDelete(templateId: string) {
    await removeStep.mutateAsync({ templateId, stepId });
  }

  return {
    isSheetOpen,
    setIsSheetOpen,
    handleOpenSheet,
    handleDelete,
    updateStep,
  };
}
```

### Regra importante

Se um handler tiver:

- validação de negócio;
- comparação de estado anterior e próximo;
- chamada de mutation;
- rollback mental;
- side effects de navegação ou fechamento;

ele é orchestration e deve sair do `components/ui/`.

---

## `components/forms/`

`components/forms/` é a camada visual de formulário da feature.

Ela pode:

- receber `form`, `onSubmit`, `isPending`;
- montar `FormField`, `FormItem`, `FormControl` e afins;
- renderizar inputs, selects, textareas e mensagens de erro.

Ela não deve:

- criar mutation;
- buscar dados;
- decidir fechamento de dialog;
- conter regra de submit maior que o necessário para montar o HTML do formulário.

---

## Regras de props

Componente de UI recebe:

- dado pronto;
- labels prontas quando necessário;
- callbacks prontos;
- flags de loading/pending.

Evitar passar:

- id cru quando a UI não precisa dele;
- objeto de mutation;
- query result inteiro sem necessidade;
- `queryClient` ou service.

### Exemplo bom

```ts
interface TemplateHeaderProps {
  title: string;
  description: string;
  statusLabel: string;
  canPublish: boolean;
  isPending?: boolean;
  onBack: () => void;
  onPublish: () => void;
}
```

### Exemplo ruim

```ts
interface TemplateHeaderProps {
  template: WorkflowTemplateDetail;
  publishMutation: ReturnType<typeof useMutation>;
  queryClient: QueryClient;
}
```

---

## Loading, error e empty state

Esses estados podem viver em `components/`, porque dependem da leitura do
servidor e da composição da tela.

`components/ui/` pode receber skeletons, cards vazios e placeholders visuais,
mas não deve ser a dona do fetch necessário para decidir qual estado mostrar.

---

## Regra especial sobre toasts e handlers

Toasts de negócio não pertencem a `components/ui/`.

Também não pertencem a handlers visuais puros.

Se um botão “Salvar” dispara um handler que:

- valida nome duplicado;
- monta payload;
- chama `mutateAsync`;
- fecha sheet no sucesso;
- trata erro com toast;

esse handler é parte da orchestration da feature, não parte da UI.

---

## Anti-padrões

- `components/ui` importando hooks de mutation;
- `components/ui` importando service;
- componente visual buscando dado por id;
- `components/forms` virando lugar de mutation;
- `components/` acumulando tudo sem extrair `hooks/components`;
- props recebendo objeto de Query completo quando só 3 campos seriam suficientes.

---

## Resumo operacional

- `components/ui` é apresentação;
- `components/forms` é apresentação de formulário;
- `components/` integra dados e UI;
- `hooks/components/` segura orchestration complexa;
- handler com regra de negócio + mutation não pertence à UI.
