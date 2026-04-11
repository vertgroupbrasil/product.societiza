# Societiza — Mapa de Features e Integrações

> **Estado de execução (desde 2026-04-11):** A única task ativa do repositório
> de produto é `societario-workflow-process` (spec aprovada em 2026-04-11, ver
> `specs/societario-workflow-process/spec.md`). Todas as demais features
> listadas abaixo — inclusive `dashboard`, `notifications`, `portal-do-cliente`,
> `importacao-exportacao`, `automacao`, `formulario-nativo`, `mcp-server`,
> `alvarás` e `certificados` — estão ARQUIVADAS POR TEMPO INDETERMINADO. O
> conteúdo deste mapa permanece como referência de visão, dependências e
> acoplamento, mas nenhuma dessas features deve avançar para spec, impact-map
> ou distribuição enquanto o arquivamento não for explicitamente revertido
> pelo humano.

Este documento descreve as features da plataforma, suas dependências, os dados que compartilham e os pontos de acoplamento críticos. É a referência principal para entender o que existe, o que está planejado para o MVP e o que vem depois.

---

## Estado atual (implementado)

### identity
Gerencia autenticação e convites. É a base de toda a plataforma — toda feature protegida depende dela para validar o usuário e obter o `accountancyId`.

### accountancy
Representa a contabilidade como unidade organizacional (multi-tenant). Toda feature da plataforma opera dentro do contexto de uma `accountancy`. Depende de `identity` para criação e autenticação.

### workflow-template
Define a estrutura de processos societários disponibilizados pela Societiza. Templates são padrões criados e mantidos pela própria plataforma — a contabilidade não configura templates, ela os utiliza. Atualmente, o único template disponível reflete o processo societário do estado de Santa Catarina. Depende de `accountancy` para escopo organizacional. Será consumido pelo `workflow-process` no MVP.

---

## MVP — próximo a ser implementado

### workflow-process
O board operacional da contabilidade. Cada processo é criado a partir de um `workflow-template` e executa dentro de uma `accountancy`. É o núcleo do MVP e habilita tudo que vem depois.

**Depende de:** `accountancy`, `workflow-template`

---

## Pós-MVP (planejado, ordem de prioridade)

### dashboard
Painel de análise e navegação filtrada dos processos em andamento. Depende dos eventos e dados gerados pelo `workflow-process`.

**Depende de:** `workflow-process`, `accountancy`

### notifications
Sistema de notificações baseado em eventos do `workflow-process`. Pré-requisito para o portal do cliente.

**Depende de:** `workflow-process`  
**Habilita:** `portal-do-cliente`

### portal-do-cliente
Interface de colaboração direta com o empresário. Permite que o cliente acompanhe e interaja com seus processos societários sem acessar o painel interno da contabilidade.

**Depende de:** `workflow-process`, `notifications`, `accountancy`

### importacao-exportacao
Permite que contabilidades com histórico existente migrem dados para a plataforma ou exportem relatórios.

**Depende de:** `workflow-process`, `workflow-template`

### automacao
Execução automática de tarefas recorrentes definidas nos templates, reduzindo trabalho manual operacional.

**Depende de:** `workflow-template`

### formulario-nativo
Formulário estruturado para captura de dados do cliente, integrado aos campos configurados no `workflow-template`. Alimenta o processo automaticamente sem necessidade de entrada manual pela contabilidade.

**Depende de:** `workflow-template` (campos), `accountancy` (branding)

### mcp-server
Servidor MCP que expõe o contexto real da operação para modelos de linguagem, permitindo IA com conhecimento do estado atual da contabilidade.

**Depende de:** `workflow-process`, `accountancy`

### alvarás
Produto futuro da Societiza para gestão do ciclo de vida de alvarás. Pode ter relação com o processo societário, mas ainda não há previsão de desenvolvimento. Está entre as últimas prioridades do pós-MVP.

### certificados
Produto futuro para gestão de certidões e documentos com validade vinculados ao cliente. Também sem previsão de desenvolvimento. Está entre as últimas prioridades do pós-MVP, junto com `alvarás`.

---

## Dados compartilhados entre features

Estas chaves circulam entre features e devem ser tratadas como contratos estáveis.

**`accountancyId`** — discriminante multi-tenant. Presente em usuários, convites e templates. É o contorno organizacional de toda a plataforma.

**`templateId`** — vincula um processo ao template usado na sua criação. Define quais etapas e tarefas o processo vai seguir.

**`processId`** — chave futura que conectará `workflow-process` → `alvarás` → `certificados` → `notifications` → `portal-do-cliente`. Não existe ainda.

**`stepId`** — identifica uma etapa dentro de um template. A ordem (`step.order`) é semanticamente significativa para a operação.

**`taskId`** — identifica uma tarefa dentro de uma etapa. A ordem (`task.order`) também é semântica.

**`fieldId`** — identifica um campo dentro de uma tarefa ou etapa.

---

## Pontos de acoplamento críticos

### templateId como configuração imutável do processo

Quando um processo é criado, ele é associado a um template. Esse template define as etapas e tarefas que o processo vai seguir. Alterar o template depois que processos estão em andamento é uma operação sensível — o sistema atualmente não tem mecanismo de migração para processos já criados com versões anteriores do template.

### accountancyId como contorno de tudo

Toda feature opera dentro do escopo de uma `accountancy`. Qualquer feature que crie, liste ou altere dados precisa filtrar por `accountancyId`. Isso é o que garante o isolamento entre contabilidades diferentes.