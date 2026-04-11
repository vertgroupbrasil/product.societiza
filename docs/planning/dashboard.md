# Dashboard — Próxima Feature Mandatória

> **ARQUIVADO POR TEMPO INDETERMINADO (desde 2026-04-11).** A partir de
> 2026-04-11, a única task ativa do repositório é `societario-workflow-process`.
> O planejamento do dashboard permanece válido como visão de produto, mas não
> deve ser iniciado, detalhado em spec nem distribuído enquanto o arquivamento
> não for explicitamente revertido pelo humano.

O dashboard ainda não existe como feature entregue. Ele é a próxima grande
prioridade após as melhorias no societário.

## Visão de produto

Do ponto de vista de produto, o dashboard deve ser a camada de leitura da
operação, enquanto o societário continua sendo a camada de execução. Ele não
pode ser passivo — cada número, card ou gráfico que represente um conjunto
operacional de dados deve, idealmente, permitir navegação para a área
operacional correspondente.

O mental model obrigatório: se existir um gráfico com os processos mais
parados, ao clicar em um segmento desse gráfico, o usuário deve ir para o
societário com um filtro aplicado, vendo exatamente os processos ligados a
esse insight. Insight deve virar ação.

## Dados promissores

Os dados mais promissores para esse dashboard incluem: total de processos
ativos, total de processos concluídos em um período, distribuição de processos
por etapa, processos mais parados, etapas com maior acumulação, processos
aguardando cliente, tarefas pendentes, movimentações recentes, alertas e
riscos, gargalos operacionais.

## Dependências técnicas

Do ponto de vista técnico, o dashboard depende do workflow (fonte dos dados de
processo) e da Accountancy (filtro multi-tenant por `accountancyId`). Ele não deve
nascer como módulo analítico isolado. Deve resumir a operação real e levar o
usuário de volta ao trabalho com contexto e filtros.

## Estado

A definição exata das métricas, visualizações e nível de interatividade ainda
não está fechada. Qualquer planejamento dessa feature deve combinar a visão de
produto com a realidade operacional do societário.