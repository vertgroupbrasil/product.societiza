# Societiza — Mapa de Features e Integrações

## Diagrama de dependências

```
+-----------------------------------------------------------------------------+
|                              SOCIETIZA                                       |
|                                                                              |
|  +-------------+                                                             |
|  |  identity   | <- autenticação + convites (parcialmente implementado)      |
|  +------+------+                                                             |
|         |                                                                    |
|         v                                                                    |
|  +-----------------+     +------------------------+                          |
|  |   accountancy   |---->|   workflow-template    |                          |
|  | (accountancyId) |     | (etapas/tasks/campos)  |                          |
|  +-----------------+     +------------------------+                          |
|         ^                                                                    |
|         |                                                                    |
|         |   [ PREVISTO MVP / NÃO IMPLEMENTADO AINDA ]                      |
|         |                                                                    |
|         |              +------------------------------+                      |
|         |              |      workflow-process         |                     |
|         |              |  (board/processos, MVP)       |                     |
|         |              +----------+-----------+--------+                     |
|         |                         |  ^                                       |
|         |                         |  +---------------------------+           |
|         |              +----------v-----------+                  |           |
|         +--------------+       dashboard       |                 |           |
|                        |  (analytics, pós-MVP) |-----------------+           |
|                        +------------------------------+                      |
|                                    |                                         |
|                         +----------v----------+                              |
|                         |    notifications    | (futura)                     |
|                         +----------+----------+                              |
|                                    |                                         |
|                    +---------------v----------------+                        |
|                    |       portal-do-cliente         | (futura)              |
|                    +--------------------------------+                        |
|                                                                              |
|  +-------------+   +--------------+   +----------------------+              |
|  |   alvaras   |   | certificados |   |  formulario-nativo   |              |
|  |  (futuro)   |   |  (futuro)    |   |      (futuro)        |              |
|  +------+------+   +------+-------+   +----------------------+              |
|         |                 |                                                  |
|         +--------+--------+                                                  |
|                  v                                                            |
|          workflow-process (conecta a operação societária)                     |
+-----------------------------------------------------------------------------+
```

---

## Tabela de integrações por feature

### Features existentes

| Feature | Depende de | É consumida por | Dados compartilhados |
|---|---|---|---|
| `accountancy` | `identity` | `workflow-template`, `workflow-process` (MVP), `dashboard` (futuro) | `accountancyId` em todo template e convite |
| `identity` | — | Todas as features protegidas | Token de identidade do usuário, `AccountancyId` |
| `workflow-template` | `accountancy` | `workflow-process` (MVP) | `templateId`, `stepId`, `taskId`, `fieldId` |

### Features do MVP ainda não implementadas

| Feature | Depende de (obrigatório) | Habilita |
|---|---|---|
| `workflow-process` | `accountancy`, `workflow-template` | `dashboard`, `notifications`, `alvaras` |

### Features pós-MVP

| Feature | Depende de (obrigatório) | Habilita |
|---|---|---|
| `dashboard` | `workflow-process`, `accountancy` | Navegação filtrada de volta ao societário |
| `notifications` | `workflow-process` (eventos) | `portal-do-cliente` |
| `portal-do-cliente` | `workflow-process`, `notifications`, `accountancy` | Colaboração direta com o empresário |
| `formulario-nativo` | `workflow-template` (campos), `accountancy` (branding) | Alimentação automática de dados no processo |
| `alvaras` | `workflow-process` (processo societário) | `certificados`, `notifications` |
| `certificados` | `accountancy` (dados do cliente) | `notifications` (vencimento) |
| `automacao` | `workflow-template` (tarefas do template) | Redução de trabalho manual operacional |
| `importacao-exportacao` | `workflow-process`, `workflow-template` | Adoção por contabilidades com histórico existente |
| `mcp-server` | `workflow-process`, `accountancy` | IA com contexto real da operação |

---

## Dados compartilhados entre features

| Chave | Tipo | Papel no sistema |
|---|---|---|
| `accountancyId` | UUID | Discriminante multi-tenant. Presente em User, UserInvite e WorkflowTemplate. É o contorno organizacional da Societiza. |
| `templateId` | UUID | Vincula processo ao template usado na criação. Identifica qual configuração de etapas/tarefas está em uso. |
| `processId` | UUID | Chave futura — conectará workflow-process → alvarás → certificados → notificações → portal do cliente. Não existe hoje (WorkflowProcess não implementado). |
| `stepId` | UUID | Identifica uma etapa dentro de um template. A ordem (`step.order`) é semântica operacional. |
| `taskId` | UUID | Identifica uma tarefa dentro de uma etapa. A ordem (`task.order`) também é semântica. |
| `fieldId` | UUID | Identifica um campo dentro de uma tarefa ou etapa. |

---

## Pontos de acoplamento críticos

### templateId como configuração do processo

Quando um processo é criado, ele é associado a um template. Esse template define
as etapas e tarefas que o processo vai seguir. Alterar o template depois que
processos estão em andamento é uma operação sensível — o sistema atualmente não
tem mecanismo de migração para processos já criados com versões anteriores do
template.