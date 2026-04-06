# Contratos das Pastas

## Objetivo

Cada pasta da feature tem um papel específico. Misturar esses papéis é a forma
mais rápida de criar código difícil de testar, de integrar e de evoluir.

Este documento define o contrato ideal de cada camada.

---

## `src/routes/endpoints.ts`

### É

O registro centralizado dos endpoints consumidos pelo frontend.

### Pode

- declarar paths e helpers parametrizados;
- agrupar endpoints por domínio;
- usar `withBase(...)`;
- refletir o contrato HTTP público disponível.

### Não pode

- conter lógica de negócio;
- montar filtro de domínio arbitrário como se fosse infraestrutura;
- fazer request;
- importar código de feature.

### Regra

Toda integração com backend começa aqui.

---

## `schemas/`

### É

A fonte de verdade do formato dos dados.

### Pode

- modelar response DTOs;
- modelar create/update DTOs;
- usar `zod` para parse, transform e refine;
- referenciar outros schemas da própria feature;
- expor enums estáveis do domínio.

### Não pode

- fazer request;
- importar hook;
- importar componente;
- carregar estado;
- esconder regra de negócio extensa que deveria ser documentada em outro nível.

---

## `server/types/`

### É

A superfície pública dos tipos TypeScript da feature.

### Pode

- expor `z.infer<>`;
- compor tipos derivados de schemas;
- declarar tipos utilitários públicos da feature.

### Não pode

- duplicar contrato manualmente sem necessidade;
- conter lógica;
- importar UI ou hooks.

---

## `server/services/`

### É

A camada HTTP mais primitiva da feature.

### Pode

- usar `fetcher`;
- chamar endpoints de `API_ENDPOINTS`;
- parsear response com schema;
- devolver entidade, lista, `void` ou envelope parseado.

### Não pode

- usar hook de React;
- disparar toast;
- invalidar cache;
- importar componente;
- conter regra de UX.

### Regra

Service fala com a API. Ele não fala com o usuário.

---

## `hooks/queries/query-options.ts`

### É

O centro de definição de query keys e query options.

### Pode

- definir `queryKey`;
- definir `queryFn`;
- definir `enabled`, `select`, `staleTime` e afins;
- importar services.

### Não pode

- chamar `useQuery`;
- carregar estado de componente;
- importar UI.

---

## `hooks/queries/`

### É

A camada de leitura para componentes.

### Pode

- usar `useQuery`, `useSuspenseQuery`, `useInfiniteQuery`;
- expor hooks de leitura por caso de uso;
- reutilizar `query-options.ts`.

### Não pode

- fazer mutation;
- chamar endpoint inline;
- conter request direto fora de service;
- assumir responsabilidade de formulário.

---

## `hooks/mutations/`

### É

A camada de escrita e side effects de persistência.

### Pode

- usar `useMutation`;
- chamar services;
- invalidar/refetchar cache;
- usar UI otimista e rollback;
- disparar toast de sucesso/erro.

### Não pode

- importar componente;
- conter JSX;
- viver dentro de `components/ui`;
- substituir `hooks/forms` quando a lógica for claramente de formulário.

---

## `hooks/forms/`

### É

O lugar da lógica de formulário reutilizável ou crescente.

### Pode

- usar `useForm`;
- usar resolver Zod;
- integrar com mutations;
- preparar defaults;
- sincronizar edição via `reset`.

### Não pode

- renderizar JSX;
- fazer fetch direto;
- acumular lógica que claramente pertence a orchestration de tela inteira.

---

## `hooks/utils/`

### É

Hooks utilitários que não são query, mutation ou form.

### Pode

- debounce;
- atalhos;
- estado derivado local;
- auxiliares de interação.

### Não pode

- virar depósito genérico para qualquer coisa sem dono;
- esconder fetch ou mutation;
- concentrar regra principal da feature quando `hooks/components` seria mais claro.

---

## `hooks/components/`

### É

Camada formal de orchestration de componente.

### Pode

- combinar query, mutation, form, state e context;
- expor handlers prontos;
- organizar fluxo de publish, reorder, save, archive e afins;
- concentrar orchestration que não deve ficar em `components/ui`.

### Não pode

- renderizar JSX;
- assumir responsabilidade de múltiplos componentes sem relação;
- virar “service layer paralela”.

---

## `contexts/`

### É

Estado compartilhado transversal dentro da feature.

### Pode

- compartilhar estado visual ou coordenativo entre componentes irmãos;
- centralizar filtros, seleção, estado de drawer ou UI específica da feature.

### Não pode

- substituir TanStack Query para dados de servidor;
- encapsular request direto;
- virar store global improvisada sem necessidade real.

### Regra

Context entra quando prop drilling real fica caro. Não entra por reflexo.

---

## `constants/`

### É

Valores imutáveis compartilhados.

### Pode

- options;
- labels;
- maps;
- limits;
- tokens fixos;
- thresholds.

### Não pode

- guardar estado;
- carregar hook;
- conter regra procedural longa.

---

## `components/`

### É

A camada de composição e integração entre hooks e UI.

### Pode

- usar hooks da feature;
- usar router;
- usar context;
- montar loading, error e empty state;
- traduzir contrato técnico em props visuais.

### Não pode

- chamar service direto como atalho;
- virar componente visual inchado quando a UI deveria estar em `components/ui`;
- concentrar orchestration enorme sem extrair hook dedicado.

---

## `components/forms/`

### É

A camada visual dos formulários da feature.

### Pode

- renderizar campos;
- receber `form`, `onSubmit` e `isPending`;
- compor primitives de formulário.

### Não pode

- criar mutation inline;
- buscar dados por conta própria;
- assumir a lógica principal do submit.

---

## `components/ui/`

### É

Apresentação pura e estado visual local.

### Pode

- layout;
- texto;
- composição visual;
- hover, open local, expand/collapse local;
- callbacks recebidos por props.

### Não pode

- `useQuery`;
- `useMutation`;
- service call;
- toast de regra de negócio;
- handler com validação de domínio + mutation;
- fetch para “carregar” o componente.

---

## `lib/`

### É

Helpers internos puros da feature.

### Pode

- funções de transformação;
- normalizadores;
- comparadores;
- builders de payload;
- helpers sem estado.

### Não pode

- request;
- hooks;
- componente;
- regra que só existe para maquiar má separação de camadas.

---

## `README.md`

### É

O mapa vivo da feature.

### Pode

- explicar domínio;
- explicar fluxo de dados;
- explicar arquitetura da feature;
- registrar decisões importantes.

### Não pode

- ser genérico;
- ser cópia de outra feature;
- contradizer o comportamento real documentado em integração ou produto.

---

## Resumo operacional

- cada pasta tem um dono de responsabilidade;
- service não fala com UI;
- UI não fala com service;
- orchestration não entra em `components/ui`;
- hooks especializados existem para reduzir mistura de responsabilidade;
- `lib/` e `contexts/` só entram quando realmente fazem sentido.
