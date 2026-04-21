---
status: approved
spec: tooling-integration-agent
created_at: 2026-04-20
---

# Lógica de Negócio — Agente Tech-Lead de Integração

---

## 1. Fluxo por perfil

Esta feature não serve usuários finais da Societiza. Os perfis que interagem com ela são exclusivamente do time de desenvolvimento interno.

---

### Backend Developer (quem aciona)

O desenvolvedor que concluiu a implementação de uma feature em `api.societiza` é quem aciona o agente.

1. O dev verifica que a feature está implementada e que os pré-requisitos estão atendidos: spec aprovada (`spec.md` com `status: approved`) e logic doc revisado (`logic.md` com `status: reviewed`) em `product.societiza`.
2. O dev aciona o agente passando o identificador da feature no formato `[domain]-[concept]` (exemplo: `societario-tracking`).
3. O agente confirma que as pré-condições estão atendidas. Se não estiverem, interrompe a execução e informa qual artefato está pendente — sem gerar nada.
4. Se as pré-condições estiverem atendidas, o agente inicia o processo: lê a spec aprovada, lê o logic doc revisado, e analisa o código implementado em `api.societiza`.
5. O agente gera o `integration.md` seguindo o template canônico de `product.societiza/templates/integration.md`.
6. O agente cria uma branch em `product.societiza` com o nome `integration/[domain]-[concept]`, commita o arquivo em `specs/[domain]-[concept]/integration.md` e abre um PR para revisão.
7. O agente reporta ao dev a URL do PR criado. O trabalho do dev nessa execução termina aqui.

Se a criação do PR falhar por problema de acesso ao GitHub, o agente salva o `integration.md` localmente dentro de `api.societiza` e instrui o dev a criar o PR manualmente — sem perder o conteúdo gerado.

---

### Tech Lead / Revisor (quem aprova)

O responsável por revisar o PR criado em `product.societiza`. Pode ser o tech lead de backend, o tech lead de frontend, ou ambos.

1. O revisor recebe a notificação do PR aberto (via GitHub).
2. O revisor lê o `integration.md` gerado e verifica se o contrato está correto, completo e coerente com a spec aprovada e o logic doc.
3. Se o documento estiver correto: o revisor aprova e mergea o PR. O `integration.md` passa a ser o artefato canônico de integração daquela feature.
4. Se o documento precisar de ajustes: o revisor solicita mudanças no PR. O dev re-aciona o agente para gerar uma versão corrigida.
5. O revisor nunca edita o `integration.md` diretamente no PR — correções são feitas re-acionando o agente ou editando manualmente e commitando na mesma branch.

O revisor não tem acesso ao código de `api.societiza` para essa revisão — ele valida o documento em `product.societiza` contra a spec e o logic doc, que ele já conhece.

---

## 2. Estados e transições

### Processo de documentação de uma feature

**Aguardando acionamento**
A feature foi implementada em `api.societiza`. O `integration.md` ainda não existe em `product.societiza`. Nenhum PR está aberto para essa feature.

**Em execução**
O agente foi acionado e está lendo os artefatos de produto, analisando o código e gerando o `integration.md`. Nenhuma interação humana ocorre neste estado.

**PR aberto**
O `integration.md` foi gerado e o PR está aberto em `product.societiza` aguardando revisão. O documento existe em uma branch — ainda não é canônico.

**Com mudanças solicitadas**
O revisor identificou problemas no documento e solicitou ajustes no PR. O `integration.md` existe mas não representa o contrato correto ainda.

**Concluído**
O PR foi aprovado e mergeado. O `integration.md` é o artefato canônico de integração da feature. O frontend pode consumi-lo com segurança.

---

**Transições:**

- `Aguardando acionamento` → `Em execução`: dev aciona o agente com o identificador correto e todas as pré-condições atendidas.
- `Em execução` → `PR aberto`: agente conclui a geração e cria o PR com sucesso.
- `Em execução` → `Aguardando acionamento` (falha de acesso): GitHub API inacessível — agente salva localmente e para. O estado retorna ao início, com o integration.md disponível apenas localmente.
- `PR aberto` → `Com mudanças solicitadas`: revisor abre review com mudanças pedidas.
- `Com mudanças solicitadas` → `Em execução`: dev re-aciona o agente para gerar versão corrigida.
- `PR aberto` → `Concluído`: revisor aprova e mergea.

O que bloqueia cada transição:
- `Aguardando` → `Em execução`: spec não aprovada, ou logic doc não revisado.
- `PR aberto` → `Concluído`: PR com mudanças solicitadas pendentes, ou sem aprovação do revisor.

---

## 3. Regras de negócio

- O agente só inicia se `specs/[domain]-[concept]/spec.md` existir com `status: approved` — sem exceção.
- O agente só inicia se `specs/[domain]-[concept]/logic.md` existir com `status: reviewed` — sem exceção.
- Se qualquer pré-condição não for atendida, o agente para imediatamente, informa qual artefato está pendente e não gera nenhum arquivo.
- O agente sempre lê o template `templates/integration.md` de `product.societiza` no momento da execução — nunca usa uma cópia em cache.
- O agente commita o `integration.md` exclusivamente em `specs/[domain]-[concept]/integration.md` dentro de `product.societiza`.
- O agente nunca commita diretamente na branch `main` de `product.societiza` — sempre via branch e PR.
- O agente nunca modifica `spec.md`, `logic.md` ou qualquer outro artefato existente em `product.societiza`.
- O nome da branch criada segue obrigatoriamente o padrão `integration/[domain]-[concept]`.
- Cada execução documenta exatamente uma feature — um identificador `[domain]-[concept]` por vez.
- O agente documenta o que o código diz — não o que a spec diz. Divergências entre spec e implementação aparecem no documento e cabem ao revisor identificar e tratar.
- O `integration.md` só é considerado canônico após o PR ser aprovado e mergeado por um humano.

---

## 4. Pontos de integração

**O dev (quem aciona) precisa saber:**
- O formato exato do identificador da feature para acionar o agente: `[domain]-[concept]` em kebab-case
- Que as pré-condições (spec aprovada + logic doc revisado) devem estar atendidas antes de acionar
- A URL do PR criado ao final de cada execução bem-sucedida
- Que, em caso de falha de acesso ao GitHub, o `integration.md` foi salvo localmente e onde encontrá-lo

**O agente precisa garantir:**
- Verificação explícita das pré-condições antes de qualquer geração
- Acesso de leitura ao repositório `product.societiza` via MCP do GitHub (spec, logic doc, template)
- Acesso de leitura ao código de `api.societiza` para análise do que foi implementado
- Acesso de escrita ao repositório `product.societiza` via GitHub API para criar branches e abrir PRs
- Que o conteúdo gerado segue a estrutura do template canônico seção por seção
- Que o PR inclui no título o nome da feature e o identificador `[domain]-[concept]`
- Que o PR inclui no corpo uma referência à spec aprovada e ao logic doc revisado

**O frontend (web.societiza) precisa saber:**
- Que o `integration.md` só é confiável após o PR estar mergeado em `product.societiza`
- Que o documento fica em `specs/[domain]-[concept]/integration.md` e pode ser consumido via MCP do GitHub

---

## 5. Casos de borda

**Spec não aprovada no momento do acionamento**
O dev aciona o agente antes de a spec ter sido formalmente aprovada (por exemplo, ainda com `status: draft`). O agente recusa, informa que a spec não está aprovada e instrui o dev a obter aprovação antes de re-acionar. Nenhum arquivo é gerado.

**Logic doc ausente ou não revisado**
O dev aciona o agente sem que o `logic.md` exista ou esteja com `status: reviewed`. O agente recusa com mensagem específica sobre qual artefato está pendente. Nenhum arquivo é gerado.

**`integration.md` já existe em `product.societiza`**
A feature já foi documentada anteriormente. O agente precisa decidir: sobrescreve o arquivo existente na mesma branch, cria uma branch com sufixo de versão, ou força atualização do PR anterior. `[pergunta aberta]` — esta decisão impacta o histórico de revisões e precisa ser acordada antes da implementação do agente.

**PR já aberto para a mesma feature**
Existe um PR com a branch `integration/[domain]-[concept]` já aberto e não mergeado. O agente re-acionado deve atualizar a branch existente (novo commit) ou fechar o PR anterior e abrir um novo? `[pergunta aberta]` — a escolha afeta o histórico de revisão e a rastreabilidade das versões do documento.

**Falha de autenticação ou acesso ao GitHub API**
O agente não consegue criar a branch ou o PR. O agente salva o `integration.md` gerado localmente em `api.societiza` (em um caminho definido pelo agente, com log claro) e informa ao dev a URL que deveria ter sido usada para criar o PR manualmente. O conteúdo não é perdido.

**Spec e código divergem**
O agente detecta que o comportamento implementado no código diverge do que a spec aprovada descreve. O agente não decide qual está correto — documenta o que o código faz e registra a divergência como um gap explícito na seção de limitações do `integration.md`. A responsabilidade de resolver pertence ao revisor e ao time de produto.

**Acionamento prematuro (feature não totalmente implementada)**
O dev aciona o agente antes de a implementação estar completa. O agente não tem mecanismo para detectar isso — ele documenta o estado atual do código. O `integration.md` gerado refletirá uma implementação parcial. A responsabilidade de acionar apenas quando a implementação estiver completa é inteiramente do dev.