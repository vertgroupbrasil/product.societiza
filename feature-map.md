# Societiza — Mapa de Features e Integrações

## Status das features

| Feature | Status | Descrição curta |
|---|---|---|
| `workflow` | **Existe hoje** | Board kanban de processos de abertura de empresa |
| `workflow-template` | **Existe hoje** | Configuração de etapas, tarefas e campos reutilizáveis |
| `accountancy` | **Existe hoje** | Gestão dos escritórios de contabilidade (multi-tenant) |
| `auth` | **Parcial** | Autenticação; gap entre narrativa de produto e contrato do backend |
| `dashboard` | **Não existe** | Camada de leitura analítica da operação societária |
| `notifications` | **Não existe** | Alertas de processo parado, tarefa vencida, mudança de etapa |
| `form` (formulário nativo) | **Não existe** | Formulário de coleta de dados com branding da contabilidade |
| `portal-do-cliente` | **Não existe** | Acesso do empresário ao status do seu processo |
| `alvaras` | **Não existe** | Controle de alvarás municipais e vencimentos |
| `certificados` | **Não existe** | Armazenamento seguro de certificados digitais em nuvem |
| `automacao` | **Não existe** | Tarefas automatizáveis no workflow (ex: consulta de viabilidade) |
| `importacao-exportacao` | **Não existe** | Migração de dados de planilhas ou sistemas externos |
| `mcp-server` | **Não existe** | IA operacional conectada ao contexto da plataforma |

---

## Diagrama de dependências

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              SOCIETIZA                                       │
│                                                                              │
│  ┌─────────────┐                                                             │
│  │    auth     │ ← base de identidade (parcialmente implementado)            │
│  └──────┬──────┘                                                             │
│         │                                                                    │
│         ▼                                                                    │
│  ┌─────────────────┐     ┌─────────────────────┐                            │
│  │   accountancy   │────►│      workflow        │                            │
│  │  (escritórios)  │     │   (board/processos)  │                            │
│  └─────────────────┘     └──────────┬───────────┘                           │
│         ▲                           │  ▲                                     │
│         │                           │  └─────────────────────────┐          │
│         │              ┌────────────▼───────────┐                │          │
│         │              │   workflow-template     │                │          │
│         │              │ (etapas/tasks/campos)   │                │          │
│         │              └────────────────────────┘                │          │
│         │                                                         │          │
│         │              ┌──────────────────────────────────┐      │          │
│         └──────────────│          dashboard               │      │          │
│                        │   (analytics, próxima feature)   │──────┘          │
│                        └──────────────────────────────────┘                 │
│                                    │                                         │
│                         ┌──────────▼──────────┐                             │
│                         │    notifications    │ (futura)                     │
│                         └──────────┬──────────┘                             │
│                                    │                                         │
│                    ┌───────────────▼────────────────┐                       │
│                    │       portal-do-cliente         │ (futura)              │
│                    └────────────────────────────────┘                       │
│                                                                              │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────────────┐             │
│  │   alvaras   │   │ certificados │   │  formulario-nativo   │             │
│  │  (futuro)   │   │  (futuro)    │   │      (futuro)        │             │
│  └──────┬──────┘   └──────┬───────┘   └──────────────────────┘             │
│         │                 │                                                  │
│         └────────┬────────┘                                                 │
│                  ▼                                                           │
│          workflow (conecta à operação societária)                            │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Tabela de integrações por feature

### Features existentes

| Feature | Depende de | É consumida por | Dados compartilhados |
|---|---|---|---|
| `accountancy` | `auth` | `workflow`, `dashboard` (futuro), `portal-do-cliente` (futuro) | `accountancyId` em todo processo |
| `workflow` | `accountancy`, `workflow-template` | `dashboard` (futuro), `notifications` (futuro), `alvaras` (futuro) | `processId`, `accountancyId`, `templateId` |
| `workflow-template` | — | `workflow` | `templateId`, `stepId`, `taskId`, `fieldId` |
| `auth` | — | Todas as features protegidas | Token de identidade do usuário |

### Features futuras e suas dependências

| Feature futura | Depende de (obrigatório) | Habilita |
|---|---|---|
| `dashboard` | `workflow`, `accountancy` | Navegação filtrada de volta ao societário |
| `notifications` | `workflow` (eventos) | `portal-do-cliente` |
| `portal-do-cliente` | `workflow`, `notifications`, `accountancy` | Colaboração direta com o empresário |
| `formulario-nativo` | `workflow-template` (campos), `accountancy` (branding) | Alimentação automática de dados no processo |
| `alvaras` | `workflow` (processo societário) | `certificados`, `notifications` |
| `certificados` | `accountancy` (dados do cliente) | `notifications` (vencimento) |
| `automacao` | `workflow` (tarefas do template) | Redução de trabalho manual operacional |
| `importacao-exportacao` | `workflow`, `workflow-template` | Adoção por contabilidades com histórico existente |
| `mcp-server` | `workflow`, `accountancy` | IA com contexto real da operação |

---

## Dados compartilhados entre features

| Chave | Tipo | Papel no sistema |
|---|---|---|
| `accountancyId` | UUID | Discriminante multi-tenant. Presente em todo processo e filtro. É o contorno organizacional da Societiza. |
| `templateId` | UUID | Vincula processo ao template usado na criação. Identifica qual configuração de etapas/tarefas está em uso. |
| `processId` | UUID | Chave que conecta workflow → alvarás → certificados → notificações → portal do cliente no futuro. |
| `stepId` | UUID | Identifica uma etapa dentro de um template. A ordem (`step.order`) é semântica operacional. |
| `taskId` | UUID | Identifica uma tarefa dentro de uma etapa. A ordem (`task.order`) também é semântica. |
| `fieldId` | UUID | Identifica um campo dentro de uma tarefa ou etapa. |

---

## Pontos de acoplamento críticos

### accountancyId como discriminante multi-tenant

Todo processo pertence a uma contabilidade. Todo filtro no board é contextualizado
pela contabilidade. Qualquer feature que liste, filtre ou agregue dados de processo
precisa considerar o `accountancyId` como contexto obrigatório. Ignorar isso gera
vazamento de dados entre escritórios diferentes.

### templateId como configuração do processo

Quando um processo é criado, ele é associado a um template. Esse template define
as etapas e tarefas que o processo vai seguir. Alterar o template depois que
processos estão em andamento é uma operação sensível — o sistema atualmente não
tem mecanismo de migração para processos já criados com versões anteriores do
template.

### A API antiga do workflow usa português

Os campos da API antiga do workflow usam nomes em português (`nome`, `contabilidade`,
`etapa`, etc.). A API nova usa inglês (`legalName`, `tradeName`, `accountancy`, etc.).
**Não renomear os campos da API antiga.** Qualquer integração nessa área precisa
respeitar essa distinção para não quebrar o contrato existente.

### order como semântica de domínio

Nos templates, a ordem de steps, tasks e fields não é apenas cosmética. Ela define
a sequência operacional que o contador vai seguir. A API retorna esses elementos
já ordenados. O frontend não deve reordenar localmente sem alinhamento explícito.

---

## Fluxo de integração: quando o backend já entregou uma feature

Quando uma feature já foi implementada no backend e precisa ser integrada no frontend:

1. **Verificar documentação da feature** — procurar pelo nome da feature dentro do
   `docs/business-context/` para entender a regra de negócio e contexto de produto.
2. **Verificar documentação do backend** — procurar em `docs/integration/[nome].md`
   pelo contrato completo: endpoints, contratos de request/response, estados, erros,
   impacto de UX e gaps conhecidos.
3. **Verificar documentação de como efetuar a integração** — Ler a skill /.agents/skills/societiza-operating-system/* e isso inclui todas as suas referências, para entender como o frontend opera. Com efeito, ler .agents/skills/societiza-code-rules/* para entender como o código deve ser escrito, e qual a ordem que se deve seguir para escrever código ao integrar uma feature.
4. **Usar o template de integração** — ver `docs/integration/TEMPLATE.md` para entender
   o formato esperado. Se a doc de integração não existir ainda, pedir ao backend que
   a preencha seguindo esse template antes de iniciar a integração.
5. **Não assumir contratos** — nunca inferir shape de request/response do nome da rota.
   Sempre basear na documentação de integração.
