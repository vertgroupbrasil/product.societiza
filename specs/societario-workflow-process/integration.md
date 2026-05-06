---
status: approved
spec: societario-workflow-process
domain: societario
created_at: 2026-05-06
approved_at: 2026-05-06
---

# Contrato de Integração — WorkflowProcess

## 1. Identificação da feature

- **Nome da feature**: WorkflowProcess (Processos Societários)
- **Módulo backend responsável**: WorkflowProcess
- **Status atual**: existente
- **Data da última atualização**: 2026-05-06
- **Responsável técnico**: não identificado (confirmar com o tech lead do módulo)

A feature está implementada e fornece a camada de operação real do setor societário.
Ela complementa a modelagem administrativa dada por `WorkflowTemplate` da plataforma, copiando os modelos estáticos imutavelmente num fluxo "instanciável".

---

## 2. Objetivo de negócio da feature

Esta feature resolve a dor operacional direta da equipe contábil onde, anteriormente, o andamento de um processo (como Abertura de Filial, Baixa, Alteração Contratual) perdia-se em planilhas ou "na cabeça do colaborador".
Ao ser amarrada por `WorkflowProcess`, cada atividade de uma contabilidade torna-se rastreável ativamente via backend, transitando em um fluxo fixo sem margem de erro.

Ela beneficia o `AccountingAdmin` e o `AccountingEmployee`. O sistema admin da plataforma tem nula atuação visível na API, e o target "Cliente pessoa física" não terá visibilidade aqui na feature MVP. O foco absoluto do frontend deve ser na renderização do Kanban dinâmico para os olhos do Backoffice e na UI de manipulação de sub-tarefas por card.

---

## 3. Escopo atual versus escopo futuro

**O que já está implementado no backend hoje:**
- `POST /workflow/processes` — Inicialização instanciado via Snapshot (copiando todas as tarefas de `appliesTo`).
- `GET /workflow/processes` — Retorno da collection `ProcessBoardItemResponse` servido via Projections para ordenações rápidas no front.
- `GET /workflow/processes/{id}` — Retorno rico (`ProcessDetailResponse`), agregando sub-coleções de `Steps`, e estes as repectivas Tasks/Fields.
- Rotas command atômicas de interação que **modificam estado do todo**: Completar Tarefa e Pular Tarefa (`Complete/Skip`).
- Preenchimentos mutáveis de tipologia puramente livres de amarras complexas, permitindo escrita de `"string Value"` tanto em atributos da Etapa Global, quanto da sub-tarefa. (`FillProcessTaskFieldCommand` e StepFieldCommand).

**O que ainda não está implementado (decisões declaradas na logic.md):**
- Nenhum usuário pode retroceder steps no processo explicitamente nem pelo command. O Status flui apenas para frente.
- Falta rastreabilidade de autoria de quem clicou na feature como um payload explícito (através de endpoint manual); a autoria deve ser registrada passivamente pelo backend a partir do JWT Token.
- Rotas do cliente final monitorar os Steps estão de fora.

**O que existe no domínio mas não está exposto por endpoint:**
- A projeção de board não armazena processos já concluídos (`"Concluído" é terminal. Processo concluído sai do board...`). Para recuperar, precisa consultar uma nova feature (`societario-process-history`).
- Edição do campo text de TargetClient (nome do cliente livre), que se originou vazio, no MVP a route update inexiste no código lido.

---

## 4. Contexto de produto e semântica da feature

O usuário usará a feature puramente como "Kanban visual interativo, mas de estado backend-driven". O front não envia comando de mover card.
1. O usuário vê a Tela Global Kanban. Todos os steps dinamicamente agrupados por colunas que o Backend retornou.
2. O usuário clica e detalha em modal/Drawer a Visão Única do processo.
3. No Card (Processo), o Front aponta quais Tarefas do sub-Step `InProgress` requerem interação obrigatoriamente (flag "IsOptional"=false).
4. O Front executa a requisição `POST .../complete`.
5. Se esta for a última task, o Board se reordena internamente. **A UI deverá reler (`invalidateQuery`) toda a tela, percebendo que a Stage em andamento mudou.** O NextStep estará em progresso, ou o Fluxo todo cairá para `Completed`.

**Estados que importam:**

- Tarefas (`Status`): `"Pending" -> "Completed"` ou `"Pending" -> "Skipped"`.
- Etapas (`Status`): `"NotStarted" -> "InProgress" -> "Completed"`.
- Board / Card Global (`Status`): Inicializado instantaneamente como `InProgress`, terminando unicamente em `Completed`.

---

## 5. Dependências e relações com outras features

- **WorkflowTemplate:** É mandatório que haja um Template `Active` que seja repassado na `Create` via ID. Falhas aqui engatilharão a flag 404/409 (`WorkflowProcess.TemplateNotActiveOrNotFound`).
- **AccountancyId:** Assim como Feature de UserManagement, aqui o limite de tenanting é imposto globalmente. O Process deve casar com o dono tenant do Controller.

---

## 6. Rotas públicas

Base path real `UsePathBase("/api")` na `ApiVersion("1.0")` => `/api/v1/workflow/processes`.

| Método | Path | Tipo | Parametros | O que Backend Aguarda |
|---|---|---|---|---|
| POST | `/processes` | Create | `{}` | AccountancyId, TemplateId (Guid), Tipo ("abertura", etc) e Cli (String). |
| GET | `/processes` | List | (paged/tenant params) | Renderizar a prancheta de `ProcessBoardItemResponse`. |
| GET | `/processes/{id}` | Detail | `{id: Guid}` | Retorna todo ecossistema daquele Card com os Fields e Tasks para View. |
| POST | `/processes/{procId}/steps/{stepId}/tasks/{taskId}/complete` | Interact | `URL guids` | Assiduidade que a Task não fora feita ainda. |
| POST | `/processes/{procId}/steps/{stepId}/tasks/{taskId}/skip` | Interact | `URL guids` | Salto intencional, sem justificativa. |
| POST | `/processes/{procId}/steps/{stepId}/fields/{fieldId}/fill` | Interact | JSON: `{ "value": str }` | Preenchimento "automático saves" estilo notium. |
| POST | `/processes/{procId}/.../tasks/{taskId}/fields/{fieldId}/fill` | Interact | JSON: `{ "value": str }` | O mesmo para fields embutidos em Task. |

---

## 7. Contrato de request e transformações

**Criando Card -> CreateWorkflowProcessCommand:**
```json
{
  "AccountancyId": "UUID",
  "TemplateId": "UUID",
  "ProcessType": "String", 
  "TargetClient": "String" 
}
```
**Preenchendo text field (`FillProcessStepFieldCommand` / `Task`):**
```json
{
  "value": "Dado Livre preenchido."
}
```

*Importante:* o Guid nas rotas completas/skip (Path Params) são imutáveis após inicialização (os fields nunca perdem seus ids originais baseados nos Snapshots), então a reenderização deles é React-Key safe no front. 

---

## 8. Contrato de response

A Query de Detalhe `/processes/{id}` retorna a carga brutal aninhada:

```ts
type ProcessDetailResponse = {
  id: string; // Guid 
  templateId: string;
  templateVersion: string;
  processType: string;
  targetClient: string;
  status: string; // enum base
  createdAt: string; // ISODate
  completedAt?: string | null;
  steps: Array<{
    id: string;
    title: string;
    order: number;
    status: string; 
    completedAt?: string | null;
    tasks: Array<{
      id: string;
      title: string;
      order: number;
      isOptional: boolean; // IMPORTANTE! Bloqueia Front
      status: string;
      completedAt?: string | null;
      skippedAt?: string | null;
      fields: Array<ProcessStepTaskFieldInstanceResponse>
    }>;
    fields: Array<ProcessStepFieldInstanceResponse>;
  }>
}
```

### 8.1. Formatos e formatação explícita UI
- Todos os Enums chegam em modo RAW `string`. Ou seja, o backend não enviará os subitens em PT-BR (ex `Pending` em Tarefas, `InProgress` no Processo). O front precisará desenhar badges com formatações locais ("Pendente", "Em Validação").
- Datas de Log (`SkippedAt` e afins) vêm com Offset. Nullibilidade representa `Ausência completa`. 
- No Grid Board (`GetProcessBoardQuery`), chega como `CurrentStepTitle`. É ali o agregador "groupby" das Arrays de colunas. Se nulo (já ocorreu algo estranho ou completo), filtrar em tela apartada do KanBan nativo.

---

## 9. Estados de negócio e transições

- `IsOptional=false` na lista `tasks` mapeada exige forte intervenção de UI: o backend vai concluir a Stage **SE E SOMENTE SE** essa Flag for saciada em **TODAS** daquele objeto daquele array. 
- Tarefas concluídas e etapas finalizadas são imutáveis (Terminal).

---

## 10. Erros esperados e edge cases

| Code HTTP / Tipo | Causador de Disparo | Ação do Frontend Esperada |
|---|---|---|
| `404 - WorkflowProcess.TemplateNotActiveOrNotFound` | Criou Crad contra base Template deletada/arquivada | Bloquear Form, exibir erro ("O Workflow deste card sofreu desativação master"). |
| `409 - WorkflowProcess.AlreadyCompleted` | Usuário engatilhou endpoint Complete Task numa Aba desatualizada da UI. | Limpar mutation local e invalidar Queries forçadamente. (Processo acabou em paralelo). |
| `409 - WorkflowProcess.TaskAlreadyResolved` | Clique acidental em duplo. | Silenciar alerta para ignorância limpa e refazer fetch. |
| `404 - WorkflowProcess.TaskNotFound` | ID quebrado / injetado de forma avulsa pela UI por mal state | Destruir UI Local. |

---

## 11. Semântica de autenticação e autorização

Tudo rola entorno da Accountancy (Contabilidade). Assim como Feature Identity, a restrição de Auth valida token JWT global por policies. Diferenças cruciais à lógica do Identity IdentityUser, na WorkflowProcess, o *AccountingAdmin* opera com poder destrutivo global, e *SystemAdmins* ficam totalmente impedidos de manipular as Tasks.

---

## 12. Impacto de UX que o frontend precisa saber

A ausência do drag & drop não significa que a UI deva ser estática. 
Toda vez que receber a Request `200` de um Complete de task, não apenas a tarefa deve dar seu `✅`, a Step parent pode mudar sua key visual para finalizada, trocando "accordions" em aberturas.

## 13. Impacto de dados no frontend (Reconciliação e Stale)

- Como o Update não altera as identidades GUID da array de Sub-Tasks (ex `id` na listagem de Fields), ele é um ID estático seguro de mapear em refs.
- **Risco Otimista:** Diferente da Role Atualização simples de nomes, concluir task aciona de modo indireto e cego para a UI a finalização do processo se for o "fechamento de conta". Desta maneira, é perigosíssimo realizar updates optimistas pesados locais de Stage. Mantenham updates optimistas estritamente em **texto digitais do FILL commands**, mas requeira Refetch de Queries totais em botões grandes (Completed / Skip).

---

## 16. Gaps, limitações e decisões abertas

1. **Client Em Branco e Edição**: Verificado no src que os Commands da Spec que ditam preenchimento textual obrigatório não explicitam se há Validation de target client na Criação. Assuma UI não vazia, e notem o fato da não possibilidade de renomeação.
2. **Obrigatoriedades em Conclusão**: O endpoint base não dita que a task pode estar "despreenchida de texts fields obrigatórios mas se permitir Concluir". O DTO de CompleteTask não processa validations com arrays de field states.
3. Não há endpoint "DeleteProcess" exposto. A lógica de soft/hard delete do Kanban ainda estaria trancada em eventuais scripts até ser priorizada na v2.
