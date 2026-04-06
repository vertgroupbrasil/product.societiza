# Relatório 3 — Gaps para o MVP e Sugestões de Correção nos Documentos

**Data:** 2026-04-06
**Escopo:** O que falta estruturar (documentação) e o que falta desenvolver (produto/engenharia) para o lançamento do MVP. Sugestões concretas de correção nos documentos de planejamento.

---

## Parte A — O que falta estruturar (documentação e planejamento)

### A.1 Não existe definição formal de escopo MVP

**Problema:** Nenhum documento responde "o que precisa estar pronto para lançar o MVP". As prioridades P1-P5 indicam uma direção, mas não definem uma linha de corte. O leitor precisa inferir que P1 + P2 + auth consolidado + freemium = MVP, mas isso nunca está escrito.

**Impacto:** Sem escopo fechado, qualquer discussão de prazo é especulativa. A equipe pode trabalhar em melhorias de P1 indefinidamente sem critério de "pronto o suficiente".

**Ação:** Criar `docs/planning/mvp-scope.md` com:
- Lista explícita de features e capacidades obrigatórias para o lançamento
- Lista explícita do que fica de fora (pós-MVP)
- Critérios de aceite por feature (não precisa ser granular — "dashboard com pelo menos X métricas funcionais" já serve)
- Dependências de infra/ops (deploy, domínio, monitoramento, suporte)

---

### A.2 P1 não tem breakdown rastreável

**Problema:** "Fortalecer o societário" é um objetivo, não um escopo. O `product-priorities.md` define P1 como "melhorar societário, workflow-template, consistência de uso, qualidade da operação, clareza da interface, confiabilidade e capacidade de configuração". São 7 dimensões sem nenhuma melhoria específica listada.

**Impacto:** Impossível saber quando P1 está "completo o suficiente" para começar P2.

**Ação:** Expandir `product-priorities.md` com uma sub-tabela de P1 listando melhorias concretas e seus status. Exemplos que deveriam estar lá (baseado no que os documentos de contexto revelam):
- Proteção de endpoints com auth middleware
- Edição segura de templates com processos em andamento
- Melhoria de UX do board (filtros, busca, indicadores visuais)
- Personalização de templates pela contabilidade
- Campos configuráveis consolidados

---

### A.3 Freemium não tem documento

**Problema:** O freemium é mencionado em `business-context.md` §3 e em `product-vision.md` como planejado para o MVP, mas não existe nenhum documento que descreva o modelo, as limitações, o fluxo de entrada self-service ou o impacto na autenticação.

**Impacto:** O freemium impacta pelo menos 4 áreas — autenticação (novo fluxo de registro), permissões (novo perfil ou adaptação?), onboarding (self-service vs. convite), e comercial (limites de processos). Sem documentação, essas áreas estão sendo planejadas no escuro.

**Ação:** Criar `docs/planning/freemium.md` com pelo menos:
- Modelo de limitação (processos ativos? processos por mês? features?)
- Fluxo de registro self-service vs. fluxo por convite
- Impacto na autenticação e no módulo Identity
- Impacto nos perfis de permissão
- O que o usuário free pode e não pode fazer

---

### A.4 Documentação de integração está vazia

**Problema:** A pasta `docs/integration/` contém apenas o `TEMPLATE.md`. Nenhuma feature existente (societário, accountancy, auth) tem documentação de integração publicada.

**Impacto:** O frontend não tem contrato formal para consumir nenhuma feature. Toda integração atual foi feita por comunicação informal ou leitura direta do código backend.

**Ação:** Gerar documentação de integração para as três features existentes:
- `docs/integration/workflow.md`
- `docs/integration/accountancy.md`
- `docs/integration/authentication.md`

---

### A.5 Specs e contracts estão vazios

**Problema:** `features/specs/` e `contracts/api/` estão vazios. O workflow definido no CLAUDE.md (spec → aprovação → contract → implementação) nunca foi executado.

**Impacto:** O processo de governança de features não tem histórico. Quando começar a ser usado, não haverá referência de como um spec aprovado se parece.

**Ação:** Retroativamente documentar pelo menos uma feature existente (societário) como spec aprovado, para servir de exemplo. Para novas features (dashboard), seguir o fluxo completo.

---

## Parte B — O que falta desenvolver (produto/engenharia)

### B.1 Proteção de endpoints (BLOQUEANTE)

**Estado:** Autenticação implementada (login, logout, refresh, reset), mas nenhum endpoint exige token. Qualquer requisição é aceita sem autenticação.

**Por que é bloqueante:** Um SaaS B2B multi-tenant não pode lançar com endpoints abertos. Dados de uma contabilidade ficariam acessíveis a qualquer requisição.

**Escopo estimado:** Middleware de autenticação + aplicação em todas as rotas protegidas + testes.

---

### B.2 Fluxo de entrada do usuário (BLOQUEANTE)

**Estado:** O fluxo atual é 100% por convite. O freemium prevê self-service. Nenhum dos dois está consolidado.

**Por que é bloqueante:** Sem fluxo de entrada funcional, ninguém consegue usar o produto. Se o MVP vai ter freemium, o registro self-service precisa existir. Se não vai, o fluxo por convite precisa estar completo e testado de ponta a ponta.

**Decisão necessária:** O MVP lança com freemium ou apenas com convite? Essa decisão desbloqueia engenharia.

---

### B.3 Dashboard (MANDATÓRIO para P2)

**Estado:** Nenhuma implementação. Métricas não definidas. Visualizações não definidas. Interatividade não definida.

**Por que é mandatório:** O `product-priorities.md` e o `business-context.md` classificam o dashboard como a próxima feature grande após P1. Sem o dashboard, a plataforma oferece execução (board) mas não oferece leitura gerencial — o que limita severamente o valor percebido por administradores de contabilidade.

**Pré-requisitos:** P1 precisa estar minimamente consolidado. O workflow precisa expor dados suficientes (contagens por etapa, datas de movimentação, processos parados).

---

### B.4 Versionamento/migração de templates (RISCO)

**Estado:** O `feature-map.md` documenta explicitamente: "o sistema atualmente não tem mecanismo de migração para processos já criados com versões anteriores do template".

**Por que é risco:** Se um template ativo for editado, processos em andamento podem ficar inconsistentes. Esse problema se agrava conforme mais contabilidades usam o sistema com processos reais.

**Escopo:** Definir política de versionamento (snapshot no momento da criação do processo? migração automática? bloqueio de edição enquanto houver processos ativos?).

---

### B.5 Personalização de templates pela contabilidade (PLANEJADO MVP)

**Estado:** O `societario-context.md` diz que "permitir que a contabilidade personalize seus próprios templates é algo que está sendo mapeado para implementação no MVP". Não está implementado.

**Por que importa:** Se a contabilidade não pode adaptar o fluxo à sua realidade, o produto depende 100% dos templates fornecidos pela Societiza — o que limita a proposta de valor de "configurabilidade real".

---

### B.6 Permissões granulares (DÍVIDA ACEITÁVEL para MVP)

**Estado:** Três perfis existentes (SystemAdmin, AccountingAdmin, AccountingEmployee). Não há granularidade dentro do contexto da contabilidade.

**Para o MVP:** Aceitável lançar com os três perfis atuais, desde que o modelo de dados suporte extensão futura sem breaking change.

---

## Parte C — Sugestões de correção nos documentos

### C.1 product-priorities.md

**Problema:** Documento puramente conceitual. Não lista features concretas nem status.

**Correção sugerida:**

Cada seção P1-P5 deveria ter uma sub-tabela:

```markdown
## Prioridade 1 — Fortalecer o núcleo operacional atual

Melhorar societário, workflow-template, consistência de uso, qualidade da
operação, clareza da interface, confiabilidade e capacidade de configuração.

### Itens de P1

| Item | Status | Escopo MVP? |
|---|---|---|
| Auth middleware aplicado em todos os endpoints | Não iniciado | Sim |
| Edição segura de templates com processos ativos | Não iniciado | Sim |
| Personalização de templates pela contabilidade | Não iniciado | Sim |
| Melhoria de filtros e busca no board | Não iniciado | A definir |
| Indicadores visuais de gargalo no board | Não iniciado | A definir |
```

Repetir para P2-P5 com as features já nomeadas em `future-features.md`.

---

### C.2 feature-map.md

**Problema 1:** Não indica prioridade (P1-P5) nem escopo (MVP / pós-MVP).

**Correção sugerida:** Adicionar duas colunas à tabela de status:

```markdown
| Feature | Status | Prioridade | MVP? | Descrição curta |
|---|---|---|---|---|
| workflow | Existe hoje | P1 | Sim | Board kanban de processos |
| dashboard | Não existe | P2 | Sim | Camada de leitura analítica |
| notifications | Não existe | P3 | Não | Alertas e acompanhamento |
```

**Problema 2:** A seção "A API antiga do workflow usa português" é um detalhe de implementação que pertence a um documento de integração, não ao mapa de features.

**Correção sugerida:** Mover para `docs/integration/workflow.md` quando esse documento for criado. Manter no feature-map apenas uma nota resumida: "Atenção: coexistem contratos antigos (português) e novos (inglês). Ver docs/integration/workflow.md."

---

### C.3 future-features.md

**Problema 1:** O dashboard não aparece neste documento. É a feature futura mais importante (P2, mandatória) e está ausente.

**Correção sugerida:** Adicionar seção "Dashboard operacional" no topo do documento, antes de "Automação", com referência cruzada para `dashboard.md`:

```markdown
## Dashboard operacional

O dashboard é a próxima feature mandatória (P2). Deve ser a camada de
leitura da operação, enquanto o societário continua sendo a camada de
execução. Cada insight deve permitir navegação direta para a área
operacional correspondente.

> Documento dedicado: [`dashboard.md`](dashboard.md)
```

**Problema 2:** Nenhuma feature tem indicação de prioridade, dependências ou escopo MVP.

**Correção sugerida:** Adicionar bloco padronizado no final de cada seção:

```markdown
**Prioridade:** P4
**Depende de:** workflow (tarefas do template)
**Habilita:** Redução de trabalho manual operacional
**Escopo:** pós-MVP
```

**Problema 3:** A ordem das seções não segue a ordem de prioridade. Automação (P4) vem antes de Notificações (P3).

**Correção sugerida:** Reordenar as seções por prioridade:
1. Dashboard operacional (P2)
2. Notificações (P3)
3. Formulário nativo (P3-P4)
4. Automação (P4)
5. Importação e exportação (P4)
6. MCP Server e IA operacional (P4)
7. Controle de alvarás (P5)
8. Certificados digitais (P5)
9. Portal do cliente (P5)

---

### C.4 dashboard.md

**Problema:** Não referencia sua prioridade (P2), não indica pré-requisitos de P1, e não tem critérios de aceite mínimos.

**Correção sugerida:** Adicionar seções:

```markdown
## Prioridade e pré-requisitos

O dashboard é P2 no roadmap. Ele depende de P1 estar minimamente
consolidado — especificamente, o workflow precisa expor dados suficientes
para alimentar as métricas do dashboard.

Pré-requisitos técnicos:
- Endpoints protegidos (auth middleware)
- Contagens por etapa acessíveis via API
- Datas de movimentação de processos registradas

## Critérios mínimos de aceite para o MVP

- [ ] Total de processos ativos por contabilidade
- [ ] Distribuição de processos por etapa
- [ ] Processos parados há mais de X dias
- [ ] Navegação do insight para o board com filtro aplicado
```

---

### C.5 business-context.md

As correções deste documento estão detalhadas no Relatório 1 (seção 4). Resumo:

| Correção | Prioridade |
|---|---|
| Adicionar data e changelog | Alta |
| Separar user management de auth na §4 | Média |
| Classificar gaps por severidade na §6 | Alta |
| Criar link para doc de freemium na §3 | Média |
| Referenciar `features/specs/` e `contracts/api/` | Baixa |

---

## Parte D — Resumo executivo: bloqueantes para o MVP

| Item | Tipo | Bloqueante? | Quem resolve |
|---|---|---|---|
| Auth middleware em todos os endpoints | Engenharia | **Sim** | api.societiza |
| Decisão freemium vs. apenas convite | Produto | **Sim** | Decisão de produto |
| Documentação do fluxo de entrada | Planejamento | **Sim** | skills (este repo) |
| Dashboard — spec e contrato | Planejamento | **Sim** | skills → api.societiza → web.societiza |
| Versionamento de templates | Engenharia | **Risco** | api.societiza |
| Personalização de templates | Engenharia | **Planejado** | api.societiza |
| Docs de integração para features existentes | Documentação | **Alta** | skills + api.societiza |
| Definição formal de escopo MVP | Planejamento | **Sim** | skills (este repo) |
| Permissões granulares | Engenharia | Não (aceitável para MVP) | pós-MVP |
| Importação de dados | Engenharia | Não (aceitável para MVP) | pós-MVP |