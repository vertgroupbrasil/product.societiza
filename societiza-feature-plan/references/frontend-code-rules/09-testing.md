# Testes — Jest + Playwright

## Objetivo

No `web.societiza`, teste não é apêndice. Ele é parte do contrato de entrega.

O padrão esperado é:

- sempre tentar rodar Jest quando houver mudança de lógica, contrato ou camada
  reutilizável;
- sempre tentar rodar Playwright quando houver fluxo navegável, UI, formulário,
  feedback visual ou comportamento do usuário;
- nunca encerrar uma entrega só com “não testado” sem explicação objetiva.

---

## O que cada camada valida

### Jest

Usar para validar:

- schemas e transforms;
- services e parsing;
- hooks de query;
- hooks de mutation;
- hooks/forms;
- helpers puros;
- constants com semântica relevante;
- comportamento lógico que não precisa de navegador real.

### Playwright

Usar para validar:

- páginas;
- dialogs, sheets e drawers;
- navegação;
- submits de formulário;
- toasts;
- estados loading/empty/error;
- interações reais da UI;
- fluxos operacionais do produto.

---

## Comandos reais do projeto

Os comandos hoje disponíveis no repositório são:

```bash
npm run test
npm run test:watch
```

Para Playwright, o padrão deve ser o comando real da ferramenta:

```bash
npx playwright test
```

### Regra importante

Não documentar comandos inexistentes no repo. Se o projeto ainda não tiver um
script dedicado como `npm run test:e2e`, a regra deve citar `npx playwright test`
explicitamente.

---

## Quando Jest é obrigatório

Rodar Jest por padrão quando houver mudança em:

- `schemas/`
- `server/services/`
- `hooks/queries/`
- `hooks/mutations/`
- `hooks/forms/`
- `lib/`
- `constants/` com impacto de comportamento

Isso inclui:

- parsing de response;
- validação de campos;
- transforms;
- enabled condicional;
- invalidate/refetch;
- rollback;
- submit logic;
- composição de payload.

---

## Quando Playwright é obrigatório

Rodar Playwright por padrão quando houver mudança em:

- página App Router;
- card interativo;
- dialog, sheet, drawer ou modal;
- form com submit;
- navegação;
- filtro;
- comportamento visual dependente de estado assíncrono;
- workflow do board;
- workflow template builder;
- experiência operacional do usuário.

Exemplos típicos:

- criar/editar contabilidade;
- abrir processo no board;
- publicar template;
- criar step/task/field;
- trocar modo de edição;
- aplicar filtro via dashboard.

---

## O que testar com Jest

### Schemas

- parse válido;
- parse inválido;
- nulidade;
- opcionalidade;
- datas;
- transforms;
- regras condicionais com `refine`.

### Services

- URL certa;
- parse certo da response;
- shape final retornado;
- comportamento quando response é `void`;
- propagação de erro.

### Query hooks

- uso correto de query options;
- `enabled` quando houver dependência;
- transformações de `select` quando existirem.

### Mutation hooks

- `mutationFn` correta;
- `invalidateQueries`/`refetchQueries`;
- toast de sucesso e erro;
- rollback em UI otimista quando existir.

### Form hooks

- default values;
- reset em edição;
- submit com create/update;
- composição de payload.

---

## O que testar com Playwright

### Checklist mínimo para fluxos de UI

- a tela carrega;
- o estado visual esperado aparece;
- a ação principal pode ser executada;
- sucesso mostra feedback visível;
- erro mostra feedback visível;
- navegação ou fechamento acontece como esperado;
- a lista/estado final reflete a ação executada.

### Casos comuns

- abrir dialog e fechar no sucesso;
- validar erro de formulário;
- clicar em card e abrir detalhe;
- mover entre estados/steps;
- aplicar filtro e ver lista resultante;
- publicar ou arquivar e ver atualização da interface.

---

## Como reportar a validação

Toda entrega deve dizer claramente:

- o que foi rodado;
- o que passou;
- o que falhou;
- se a falha é pré-existente ou introduzida pela mudança;
- se algum teste não se aplica.

Exemplo bom:

```txt
Jest:
- npm run test -- src/features/accountancy
- passou

Playwright:
- npx playwright test __tests__/e2e/accountancy
- passou

Build:
- npm run build
- passou
```

Exemplo ruim:

```txt
Não testei.
```

---

## Estrutura recomendada para Jest

O teste pode ficar:

- ao lado do arquivo;
- ou em `__tests__/` da camada.

O importante é manter proximidade semântica.

Exemplos:

```txt
src/features/accountancy/schemas/accountancy.schema.test.ts
src/features/accountancy/server/services/__tests__/accountancy.service.test.ts
src/features/workflow-template/hooks/mutations/useTemplateMutations.test.ts
```

---

## Estrutura recomendada para Playwright

```txt
__tests__/
  e2e/
    accountancy/
      accountancy-crud.spec.ts
    workflow/
      board.spec.ts
    workflow-template/
      template-builder.spec.ts
```

O nome do spec deve explicar o fluxo, não só a feature.

---

## O que fazer quando a suíte já está quebrada

Se Jest ou Playwright falhar por problema legado:

- deixar isso explícito;
- isolar, quando possível, o teste da área alterada;
- não fingir green suite;
- distinguir claramente regressão nova de falha anterior.

### Regra

Falha preexistente não autoriza encerrar sem explicar a situação.

---

## Anti-padrões

- inventar comando que não existe no repo;
- testar só manualmente e reportar como se fosse automação;
- não testar UI alterada;
- deixar mutation crítica sem teste nem validação de fluxo;
- esconder falha preexistente em frase genérica.

---

## Resumo operacional

- Jest valida lógica e contrato;
- Playwright valida experiência real;
- usar `npm run test` e `npx playwright test`;
- descrever claramente o que foi validado;
- teste é parte da entrega, não detalhe opcional.
