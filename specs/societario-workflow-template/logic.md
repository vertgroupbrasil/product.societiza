---
status: approved
spec: societario-workflow-template
created_at: 2026-05-06
approved_at: 2026-05-06
---

# WorkflowTemplate — Lógica de Negócio

## 1. Fluxo por perfil

### SystemAdmin

O SystemAdmin é o único ator que gerencia templates. Ele cria, edita, ativa e
arquiva templates.

**Criação de template:**

1. Acessa o módulo de templates.
2. Cria um novo template informando nome (máx 150 caracteres) e descrição
   (máx 500 caracteres).
3. O template nasce em Draft imediatamente.
4. Adiciona steps ao template (title, description, order).
5. Em cada step, adiciona tasks (title, description, type, configuration,
   order, isOptional).
6. Em cada step ou task, adiciona fields (label, fieldType, options se Select,
   order).
7. Pode editar e reordenar steps, tasks e fields livremente enquanto o template
   está em Draft.
8. Pode remover steps, tasks e fields (com cascata conforme descrito na seção
   de regras).

**Ativação:**

1. Solicita a ativação do template em Draft.
2. O sistema verifica:
   - O template tem ao menos uma step.
   - Cada step tem ao menos uma task.
3. Se alguma condição falhar, a ativação é recusada com erro específico
   (`ActivationRequiresAtLeastOneStep` ou `ActivationRequiresAllStepsToHaveTask`).
4. Se tudo for satisfeito, o template passa para Active.
5. A partir desse momento, o template é imutável — nenhuma escrita em steps,
   tasks ou fields é aceita.

**Arquivamento:**

1. Solicita o arquivamento de um template Active.
2. O template passa para Archived.
3. Archived é estado terminal — não pode ser revertido.
4. Templates arquivados continuam visíveis para o SystemAdmin via query param
   `includeArchive=true` na listagem.

**Atualização de nome e descrição:**

O SystemAdmin pode atualizar nome e descrição de um template em qualquer status,
desde que o template não esteja arquivado. A edição de steps, tasks e fields
exige Draft.

**Listagem:**

- Padrão (sem query params): retorna apenas Active.
- `includeDraft=true`: inclui Draft.
- `includeArchive=true`: inclui Archived.

**Ações não permitidas e o que acontece:**

- Tentar criar/editar/remover steps, tasks ou fields em template Active ou
  Archived: recusado com `TemplateNotEditable`.
- Tentar ativar template sem steps: recusado com `ActivationRequiresAtLeastOneStep`.
- Tentar ativar template com step sem tasks: recusado com
  `ActivationRequiresAllStepsToHaveTask`.
- Tentar reverter Archived para Active: ação não existe no sistema.

---

### AccountancyAdmin

AccountancyAdmin não gerencia templates. Sua interação se limita à leitura.

1. Acessa a listagem de templates disponíveis.
2. Vê apenas templates com status Active.
3. Pode ver o detalhe completo de um template Active (steps, tasks, fields
   ordenados por order).
4. Não pode criar, editar, ativar ou arquivar templates.

Templates em Draft ou Archived são invisíveis para esse perfil na listagem e no
detalhe — qualquer tentativa de acessar um template não-Active resulta em
`NotFoundError`.

---

### AccountancyEmployee

Mesmo comportamento que o AccountancyAdmin: somente leitura de templates Active.
Não gerencia templates.

---

## 2. Estados e transições

### Template

**Estados:** Rascunho (Draft) · Publicado (Active) · Arquivado (Archived)

**Transições:**

- **Criação → Draft:** imediato. Todo template nasce em Draft sem ação adicional.
- **Draft → Active:** ação explícita do SystemAdmin. Pré-condições: ao menos
  1 step, cada step com ao menos 1 task.
- **Active → Archived:** ação explícita do SystemAdmin.

**Restrições:**

- Draft: editável. Steps, tasks e fields podem ser criados, atualizados e
  removidos.
- Active: imutável para steps/tasks/fields. Nome e descrição do template ainda
  podem ser atualizados.
- Archived: imutável e terminal. Nenhuma escrita de qualquer tipo.
- Não existe transição Active → Draft, Archived → Active ou Archived → Draft.

---

### Step

**Estados:** editável (template em Draft) · congelada (template Active ou Archived)

**Transições:**

- Criada e modificável enquanto o template está em Draft.
- Congelada permanentemente quando o template é ativado.

**Restrições:**

- Título único dentro do template (case-insensitive, trim).
- Order 1-based, clamped e com insert-shift (ver seção 3).
- Remoção remove em cascata: step fields, tasks e task fields da step.

---

### Task

**Estados:** editável (template em Draft) · congelada (template Active ou Archived)

**Transições:**

- Criada e modificável enquanto o template está em Draft.
- Congelada permanentemente quando o template é ativado.

**Restrições:**

- Título único dentro da step (case-insensitive, trim).
- Order 1-based, clamped e com insert-shift.
- Tipos: Manual, Automatic, Approval.
- `isOptional` indica se a task é obrigatória no fluxo de execução
  (relevante para o WorkflowProcess).
- `configuration` é JSON string, padrão "{}".
- Remoção remove em cascata todos os task fields.

---

### Field (step ou task)

**Estados:** editável (template em Draft) · congelado (template Active ou Archived)

**Transições:**

- Criado e modificável enquanto o template está em Draft.
- Congelado permanentemente quando o template é ativado.

**Restrições:**

- Label único dentro do escopo (step ou task), case-insensitive com trim.
- Order 1-based, clamped e com insert-shift.
- Tipos: Text, Boolean, Integer, Select, Date.
- Se `fieldType = Select`: `options` obrigatório, JSON string não vazio.
- Se `fieldType ≠ Select`: `options` deve ser nulo.

---

## 3. Regras de negócio

### Template

- Todo template nasce em Draft. Nunca é criado já Active ou Archived.
- Apenas SystemAdmin pode criar, editar, ativar ou arquivar templates.
- Qualquer operação de escrita em steps, tasks ou fields de um template
  não-Draft é recusada com `TemplateNotEditable`.
- A ativação exige ao menos 1 step, e cada step com ao menos 1 task — ambas
  as condições são verificadas antes de persistir a transição.
- Archived é estado terminal. Uma vez arquivado, o template não pode ser reativado.
- A versão começa em "v1.0" e avança conforme `societario-template-versioning`.

### Step

- Título de step único por template, case-insensitive com trim.
  Duplicidade → `StepTitleAlreadyExists`.
- Order é clamped entre 1 e `(total de steps + 1)` antes do insert-shift.
- Insert-shift ao inserir/atualizar para order N: se já existe step com
  order = N, todos os steps com order >= N recebem +1, então o novo item
  ocupa order = N.
- Remoção de step: remove em cascata step fields, tasks e task fields.

### Task

- Título de task único por step, case-insensitive com trim.
  Duplicidade → `TaskTitleAlreadyExists`.
- Order clamped entre 1 e `(total de tasks na step + 1)`.
- Insert-shift aplicado da mesma forma que em steps.
- Tipos de task disponíveis: Manual, Automatic, Approval.
- `configuration` é JSON string; valor padrão "{}".
- Remoção de task: remove em cascata todos os task fields.

### Field (step ou task)

- Label de step field único por step, case-insensitive com trim.
  Duplicidade → `StepFieldLabelAlreadyExists`.
- Label de task field único por task, case-insensitive com trim.
  Duplicidade → `TaskFieldLabelAlreadyExists`.
- Order clamped entre 1 e `(total de fields no escopo + 1)`.
- Insert-shift aplicado da mesma forma.
- `fieldType = Select` → `options` obrigatório, JSON string não vazio.
- `fieldType ≠ Select` → `options` deve ser nulo.

---

### Insert-shift — exemplo numérico

**Exemplo 1 — inserção no meio:**

Steps existentes com orders `[1, 2, 3]`. SystemAdmin adiciona nova step com
`order = 2`.

1. Clamp: total = 3, máx = 4. order 2 está entre 1 e 4 → sem ajuste.
2. Conflito: já existe step com order = 2.
3. Deslocamento: steps com order >= 2 recebem +1 → orders ficam `[1, 3, 4]`.
4. Inserção: nova step recebe order = 2.
5. Resultado final: `[1, 2, 3, 4]`.

**Exemplo 2 — order acima do máximo (clamping):**

Steps existentes com orders `[1, 2, 3]`. SystemAdmin tenta adicionar com
`order = 10`.

1. Clamp: total = 3, máx = 4. order 10 > 4 → clamped para 4.
2. Conflito: não existe step com order = 4.
3. Inserção direta: nova step recebe order = 4.
4. Resultado final: `[1, 2, 3, 4]`.

**Exemplo 3 — inserção no início:**

Steps existentes com orders `[1, 2, 3]`. SystemAdmin adiciona com `order = 1`.

1. Clamp: máx = 4. order 1 está entre 1 e 4 → sem ajuste.
2. Conflito: já existe step com order = 1.
3. Deslocamento: steps com order >= 1 recebem +1 → orders ficam `[2, 3, 4]`.
4. Inserção: nova step recebe order = 1.
5. Resultado final: `[1, 2, 3, 4]`.

---

### Filtragem por role na listagem

| Role | Vê Draft | Vê Active | Vê Archived |
|---|---|---|---|
| SystemAdmin | Apenas com `includeDraft=true` | Sempre | Apenas com `includeArchive=true` |
| AccountancyAdmin | Nunca | Sempre | Nunca |
| AccountancyEmployee | Nunca | Sempre | Nunca |

Comportamento padrão do SystemAdmin sem query params: retorna apenas Active.
Os query params `includeDraft` e `includeArchive` são ignorados para roles que
não são SystemAdmin.

---

### Tabela de erros

| Código | Trigger |
|---|---|
| `TemplateNotEditable` | Qualquer operação de escrita (step, task ou field) em template que não está em Draft |
| `ActivationRequiresAtLeastOneStep` | Tentativa de ativar template sem nenhuma step |
| `ActivationRequiresAllStepsToHaveTask` | Tentativa de ativar template com ao menos uma step sem nenhuma task |
| `StepTitleAlreadyExists` | Step com título duplicado (case-insensitive) dentro do mesmo template |
| `TaskTitleAlreadyExists` | Task com título duplicado (case-insensitive) dentro da mesma step |
| `StepFieldLabelAlreadyExists` | Field com label duplicado (case-insensitive) dentro da mesma step |
| `TaskFieldLabelAlreadyExists` | Field com label duplicado (case-insensitive) dentro da mesma task |
| `NotFoundError` | Template, step, task ou field não encontrado (inclui templates em status invisível para o role do usuário) |

---

## 4. Pontos de integração

```
Frontend precisa saber:
  - Status do template (Draft / Active / Archived) para determinar quais ações
    estão disponíveis: editar steps/tasks/fields apenas em Draft; ativar apenas
    em Draft; arquivar apenas em Active; nenhuma edição em Archived
  - Lista ordenada de steps (por order), com tasks (por order) e fields
    (por order), para renderizar o editor de template corretamente
  - Tipos de task disponíveis — Manual, Automatic, Approval — para o seletor
    na criação e edição de task
  - Tipos de field disponíveis — Text, Boolean, Integer, Select, Date — para
    o seletor na criação e edição de field
  - Que fieldType = Select torna o campo options obrigatório — para validação
    e exibição condicional do campo de opções na interface
  - Que após qualquer operação de criação ou atualização com order informado,
    os orders de outros itens do mesmo escopo podem ter mudado — o frontend
    deve recarregar a lista após essas operações
  - Que a ativação pode falhar com erros de validação específicos
    (ActivationRequiresAtLeastOneStep, ActivationRequiresAllStepsToHaveTask)
    — exibir mensagem contextualizada por erro
  - Que erros de unicidade (StepTitleAlreadyExists, TaskTitleAlreadyExists,
    StepFieldLabelAlreadyExists, TaskFieldLabelAlreadyExists) devem ser
    exibidos no campo correto, sem validação local prévia

Backend precisa garantir:
  - Que toda operação de escrita em steps/tasks/fields verifica o status do
    template antes de processar — rejeita com TemplateNotEditable se não Draft
  - Que o insert-shift é aplicado atomicamente: clamp do order, deslocamento
    dos existentes e inserção em uma única operação
  - Que as validações de unicidade são case-insensitive com trim
  - Que a ativação valida as duas condições antes de persistir a transição
  - Que a remoção de step e task aplica cascata completa descrita nas regras
  - Que o campo options é validado contra fieldType na criação e na atualização
  - Que a listagem respeita includeDraft e includeArchive apenas para
    SystemAdmin, retornando sempre apenas Active para outros roles
  - Que o response de detalhe do template inclui steps, tasks e fields
    ordenados por order crescente
  - Que templates em Draft ou Archived retornam NotFoundError para roles
    que não são SystemAdmin, mesmo que o id exista
```

---

## 5. Casos de borda

**Reordenamento em escopo com um único item**
Ao adicionar um item com `order = 2` em um escopo com apenas 1 item: clamp
reduz para máx = 2. Não há conflito (o item existente tem order = 1). O novo
item recebe order = 2 diretamente. O item existente permanece em order = 1.

**Atualizar order de um item para o mesmo valor que ele já possui**
O item ocupa order = N; nenhum outro item tem order = N. Não há conflito e
nenhum deslocamento ocorre. A operação é idempotente.

**Ativar template com steps que possuem apenas tasks opcionais**
As regras de ativação exigem apenas que cada step tenha ao menos 1 task,
independente de `isOptional`. Um template com todas as tasks marcadas como
opcionais pode ser ativado normalmente. O comportamento de steps sem tasks
obrigatórias no fluxo de execução é tratado na spec do WorkflowProcess.

**Dois ou mais templates ativos simultaneamente**
O WorkflowTemplate não limita o número de templates em status Active. Restrições
sobre quantos templates ativos coexistem para uma contabilidade são gerenciadas
pelo módulo `societario-template-versioning`.

**Template arquivado enquanto há processos em andamento**
A permissão ou bloqueio de archivamento em função de processos ativos é
responsabilidade do módulo `societario-template-versioning`, via a interface
`IWorkflowProcessDataService`. O WorkflowTemplate executa o archivamento quando
a requisição chega — a guarda é feita antes, externamente.

**Acesso de AccountancyAdmin a template em Draft por id direto**
Mesmo que o AccountancyAdmin conheça o id de um template Draft, a requisição
retorna `NotFoundError`. O status Draft é tratado como inexistente para esse
perfil — não há distinção de mensagem entre "não existe" e "não está visível".
