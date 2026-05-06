---
status: approved
domain: societario
created_at: 2026-05-06
approved_at: 2026-05-06
---

# WorkflowTemplate — Modelos de processo societário

## 1. Visão da feature

O WorkflowTemplate é o módulo onde o administrador da Societiza cria e gerencia
os modelos reutilizáveis de processo. Um template define, de forma abstrata e
estruturada, quais etapas compõem um processo societário, quais tarefas existem
em cada etapa e quais campos de informação devem ser preenchidos ao longo do
processo. Uma vez publicado, o template se torna o molde que as contabilidades
usam como base para executar processos reais dos seus clientes.

O template existe em três momentos distintos: em elaboração (Rascunho), onde
pode ser construído e ajustado livremente; publicado e ativo (Ativo), onde está
disponível para as contabilidades; e arquivado (Arquivado), quando deixa de ser
utilizado. A publicação é uma ação deliberada do administrador — o template nunca
fica disponível para uso sem aprovação explícita.

O SystemAdmin é o único que cria e gerencia templates. Contadores
(AccountancyAdmin e AccountancyEmployee) visualizam apenas os templates ativos
da plataforma, sem acesso às versões em rascunho ou arquivadas.

---

## 2. Problema que resolve

Antes do WorkflowTemplate, cada processo societário existia apenas na cabeça do
contador ou em planilhas avulsas — sem uma estrutura definida que garantisse
que todos os passos fossem seguidos de forma consistente entre os escritórios.
A Societiza precisava de um lugar central onde o administrador da plataforma
pudesse criar os modelos canônicos de processo e distribuí-los para todas as
contabilidades de forma padronizada.

Sem templates, os contadores precisariam montar cada processo do zero, sem uma
referência que indicasse quais etapas são necessárias, quais tarefas existem em
cada etapa e quais informações precisam ser coletadas. O resultado seria processos
inconsistentes entre escritórios, etapas esquecidas e informações faltando. O
workaround seria confiar na memória e na experiência individual de cada contador
— frágil, não escalável e impossível de auditar.

---

## 3. Para quem é

**Beneficiário direto:** o SystemAdmin da Societiza, que é a única pessoa que
cria, edita, ativa e arquiva templates. O SystemAdmin configura os modelos de
processo; esses modelos se tornam o padrão operacional para todas as
contabilidades da plataforma.

**Beneficiário indireto:** o contador (AccountancyAdmin ou AccountancyEmployee),
que ganha uma estrutura clara e padronizada para executar o processo real do
cliente. O contador não configura o template — ele o usa como base quando o
WorkflowProcess é iniciado.

O SystemAdmin usa o módulo de templates na fase de setup e manutenção da
plataforma: ao lançar um novo tipo de processo, ao atualizar a estrutura de um
processo existente ou ao aposentar um modelo obsoleto.

---

## 4. Como melhora a experiência atual

**Antes:**
Não há estrutura central. O administrador da plataforma não tem onde definir
como um processo societário deve ser conduzido. Cada contabilidade interpreta o
processo à sua maneira. Informações coletadas variam entre escritórios. Nenhuma
consistência garantida entre a operação de diferentes clientes e diferentes times.

**Depois:**
O SystemAdmin acessa o módulo de templates, cria o modelo do processo com suas
etapas, tarefas e campos específicos, ajusta livremente enquanto está em
rascunho e publica quando está pronto. A partir daí, o template está disponível
para todas as contabilidades como o molde padrão. Atualizar o processo significa
criar uma nova versão do template — as operações em andamento não são afetadas.
Aposentar um modelo obsoleto é arquivá-lo.

---

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**

- **Identity** — a autenticação garante que apenas o SystemAdmin gerencie
  templates. Contadores só têm acesso de leitura ao que está ativo.
- **Accountancy** — na listagem, contadores de qualquer contabilidade veem os
  templates ativos disponíveis na plataforma. Os templates não são exclusivos
  de uma contabilidade.

**O que esta feature habilita no futuro:**

- **WorkflowProcess** — os processos reais dos clientes são instanciados a
  partir dos templates ativos. Sem template ativo, não é possível criar nenhum
  processo.
- **Template Versioning** (`societario-template-versioning`) — a capacidade
  de publicar novas versões de um template sem interromper os processos em
  andamento. Depende inteiramente da estrutura de ciclo de vida que o
  WorkflowTemplate define.
- **Dashboard e métricas** — a estrutura de etapas e tarefas dos templates
  define as dimensões de análise disponíveis no futuro.

**Posição no produto:**

Feature fundacional do societário. É o pré-requisito de tudo que existe na
camada de execução. Sem template ativo, a Societiza não tem operação societária.

---

## 6. O que não é (escopo negativo)

- **Não é** a criação ou execução de processos reais dos clientes. O template
  é o molde; o WorkflowProcess é a execução.
- **Não inclui** versionamento de templates. Publicar uma nova versão de um
  template existente está descrito na spec `societario-template-versioning`.
- **Não inclui** personalização de templates por cada contabilidade. O template
  é gerenciado centralmente pela Societiza via SystemAdmin.
- **Não inclui** a lógica de execução de tarefas do tipo `Automatic`. O campo
  `type` e `configuration` são estruturados no template, mas a execução
  automática é resolvida na camada de processo.
- **Não inclui** o fluxo de aprovação de tarefas do tipo `Approval`. A
  configuração de quem aprova fica no campo `configuration`; o fluxo de
  aprovação acontece no WorkflowProcess.
- **Não inclui** exclusão de templates. Templates são apenas arquivados —
  nenhum dado é destruído.
- **Não inclui** limite de quantos templates podem estar ativos
  simultaneamente. Esse controle pertence ao módulo de versionamento.

---

## 7. Custos adicionais

Nenhum custo externo identificado. O WorkflowTemplate opera inteiramente dentro
da infraestrutura existente da Societiza, sem dependências de APIs de terceiros,
serviços externos ou recursos de infraestrutura adicionais.

---

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Quem gerencia templates | Exclusivamente o SystemAdmin. Contadores não criam nem editam templates. |
| Visibilidade por status | Draft e Archived são visíveis apenas para SystemAdmin (com query params explícitos). Contadores veem apenas Active. |
| Estado inicial do template | Todo template nasce em Draft. Nunca nasce como Active ou Archived. |
| Transições de estado | Draft → Active → Archived. Não há reversão de nenhuma transição. Archived é estado terminal. |
| Regras de ativação | Ao ativar, o template precisa ter ao menos uma etapa e cada etapa precisa ter ao menos uma tarefa. Se não, a ativação é recusada com erro específico. |
| Template ativo é imutável | Uma vez ativo, o template não pode ser editado. Para modificar, é necessário criar uma nova versão via `societario-template-versioning`. |
| Reordenamento dos subitens | Etapas, tarefas e campos usam order 1-based com lógica insert-shift: inserir no order N desloca os itens existentes com order >= N. O value de order é clamped entre 1 e (total de itens + 1). |
| Unicidade de nomes | Títulos de steps são únicos dentro do template; títulos de tasks dentro da step; labels de fields dentro do escopo (step ou task). Comparação case-insensitive com trim. |
| Remoção em cascata | Remover uma step remove todos os seus fields e todas as suas tasks (com seus fields). Remover uma task remove todos os seus fields. |
| Campo options do tipo Select | `options` é obrigatório e exclusivo para fields do tipo Select. Para os demais tipos, `options` deve ser nulo. |
| Versão do template | Começa em "v1.0". Avança conforme o ciclo de versionamento (`societario-template-versioning`). |
| Tipos de tarefa | Manual (execução humana), Automatic (execução pelo sistema), Approval (requer aprovação). O campo `configuration` armazena a configuração específica de cada tipo como JSON string. |
| Tipos de campo | Text, Boolean, Integer, Select, Date. |

---

## 9. Perguntas abertas

Nenhuma. Todas as decisões de produto estão fechadas. A feature está 100%
implementada — este documento registra o que foi construído.