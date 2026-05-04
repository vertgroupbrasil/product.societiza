---
status: approved
spec: societario-template-versioning
created_at: 2026-05-04
approved_at: 2026-05-04
---

# Lógica de Negócio — Versionamento e Ciclo de Vida do WorkflowTemplate

---

## 1. Fluxo por perfil

### SystemAdmin

**Criar e publicar um template**

O SystemAdmin acessa a área de templates e cria um novo template. Ele nasce em Rascunho — estrutura totalmente editável: steps, tasks, fields, nome e descrição. Quando considera o template pronto, publica. O template passa para Ativo. A partir desse momento, a estrutura (steps, tasks, fields) fica somente-leitura. Apenas nome e descrição permanecem editáveis.

**Editar nome ou descrição de um template Ativo**

O SystemAdmin abre um template Ativo e edita nome ou descrição diretamente. Não envolve fluxo de versionamento — é uma edição cosmética que não afeta processos em andamento.

**Criar nova versão (fork)**

O SystemAdmin abre um template Ativo e aciona "Criar nova versão". O sistema cria uma cópia completa do template — todos os steps, tasks e fields — em estado Rascunho, com referência ao template de origem registrada. O template original permanece Ativo e continua aceitando novos processos normalmente. O SystemAdmin edita o Rascunho com liberdade total.

Se o SystemAdmin tentar acionar "Criar nova versão" em um template que não está Ativo (Rascunho, Em transição ou Arquivado), o sistema recusa. Fork fora do estado Ativo é inválido — regra de domínio, não apenas restrição de interface.

**Ativar nova versão**

O SystemAdmin abre o Rascunho derivado do fork e aciona "Ativar". O sistema executa uma transição atômica:

- O Rascunho passa para Ativo.
- O template original (que estava Ativo) passa para Em transição.

Essas duas transições acontecem na mesma operação. Não existe momento em que nenhuma versão está Ativa, nem em que duas estão Ativas simultaneamente.

Se no momento da ativação o template original não tem processos em andamento, ele é arquivado imediatamente — salta o estado Em transição e vai direto para Arquivado na mesma operação.

A partir desse momento, novos processos nascem exclusivamente da nova versão Ativa. O template Em transição não aceita novos processos.

Se o template-pai do Rascunho já não está mais Ativo no momento da ativação (porque outra versão foi ativada antes), o sistema recusa a operação com mensagem explicando que o template de origem não é mais a versão Ativa corrente.

**Arquivar manualmente um template**

Disponível para templates em estado Ativo ou Em transição. O sistema verifica se há processos em andamento associados ao template. Se houver, a operação é bloqueada com mensagem informativa indicando quantos processos estão impedindo o arquivamento. Se não houver, o template é arquivado imediatamente.

**O que o SystemAdmin não pode fazer:**
- Editar estrutura de um template Ativo, Em transição ou Arquivado.
- Editar nome ou descrição de um template Em transição ou Arquivado.
- Fazer fork de um template que não está Ativo.
- Arquivar um template com processos em andamento.
- Reverter um template Em transição para Ativo.
- Criar processos (operação exclusiva das contabilidades).

---

### AccountingAdmin e AccountingEmployee (contadores)

Os contadores não interagem com versionamento em nenhum momento. Não veem estados de template, não veem referências a versões e não recebem notificações sobre transições.

**Criar um processo**

O contador seleciona o tipo de processo que deseja criar. O sistema associa automaticamente o novo processo à versão Ativa mais recente da linhagem correspondente. O contador não escolhe versão — a resolução é invisível. O processo nasce com a estrutura materializada da versão Ativa no momento da criação: steps, tasks e fields ficam registrados no próprio processo e não mudam mais, independente de qualquer evolução futura do template.

**Visualizar o board**

O board exibe colunas que representam os steps dos processos em andamento. As colunas visíveis são a união dos steps da versão Ativa com os steps atualmente ocupados por processos de versões Em transição. Cada processo aparece na coluna correspondente ao seu step atual. Processos de versões anteriores não aparecem em colunas de steps que não existiam na sua versão — simplesmente não transitam por elas. Conforme esses processos concluem, as colunas exclusivas de versões anteriores esvaziam e desaparecem. Nenhum rótulo de versão é exibido em nenhum momento.

**O que o contador não pode fazer:**
- Ver ou acessar qualquer informação sobre versões de template.
- Escolher qual versão de template usar ao criar um processo.
- Criar processos a partir de templates Em transição ou Arquivados.

---

## 2. Estados e transições

### Template

**Rascunho**
Template em construção. Totalmente editável — steps, tasks, fields, nome e descrição. Não aceita criação de processos. Não é visível para contadores.

- → Ativo: SystemAdmin publica. Sem pré-condições além do estado atual ser Rascunho.
- → [descarte]: ver caso de borda.

**Ativo**
Template publicado e em uso. Estrutura somente-leitura. Nome e descrição editáveis. Aceita novos processos. Elegível para fork.

- → Em transição: nova versão derivada deste template é ativada (transição atômica). Não pode ser bloqueado — acontece independente da quantidade de processos em andamento.
- → Arquivado: arquivamento manual, bloqueado se há processos em andamento. Ou imediatamente, se no momento da ativação da nova versão não há processos em andamento.

**Em transição**
Template que teve uma versão mais nova ativada. Completamente imutável — nem estrutura, nem nome, nem descrição. Não aceita novos processos. Mantém os processos em andamento até sua conclusão.

- → Arquivado: automático, quando o último processo em andamento associado a este template é concluído. Ou manual, se não há processos em andamento (SystemAdmin arquiva explicitamente).
- Bloqueio do arquivamento manual: presença de processos em andamento.

**Arquivado**
Ciclo encerrado. Completamente imutável. Não aceita novos processos. Não elegível para fork. Mantido para referência histórica e rastreabilidade da linhagem. Estado final — sem transições de saída.

---

## 3. Regras de negócio

- Template Rascunho → estrutura, nome e descrição são editáveis.
- Template Ativo → apenas nome e descrição são editáveis; estrutura (steps, tasks, fields) é somente-leitura.
- Template Em transição → completamente imutável; nenhum campo editável.
- Template Arquivado → completamente imutável; nenhum campo editável.
- Fork disponível apenas em templates Ativos → regra de domínio; backend rejeita fork em qualquer outro estado.
- Ativação de nova versão → transição atômica; novo Rascunho passa para Ativo e template original Ativo passa para Em transição na mesma operação.
- Template original sem processos em andamento no momento da ativação → transita diretamente para Arquivado, saltando Em transição.
- Template Em transição → não aceita criação de novos processos.
- Novos processos → sempre associados à versão Ativa mais recente da linhagem, resolvida automaticamente pelo sistema.
- Arquivamento manual → disponível apenas em Ativo e Em transição; bloqueado se há processos em andamento.
- Arquivamento automático → disparado quando o último processo em andamento de um template Em transição é concluído.
- No máximo uma versão Ativa por linhagem → invariante de sistema; garantido pela atomicidade da transição de ativação.
- Múltiplas versões Em transição podem coexistir na mesma linhagem (forks consecutivos com processos pendentes em cada um).
- Reversão de transição → não existe; template Em transição nunca volta para Ativo.
- Estrutura do processo → materializada no momento da criação; imutável após isso, independente de evoluções futuras do template.
- Colunas do board → união dos steps da versão Ativa com os steps atualmente ocupados por processos de versões Em transição.
- Processos de versões anteriores → não aparecem em colunas de steps inexistentes na sua versão.
- Versionamento → 100% invisível para AccountingAdmin e AccountingEmployee; nenhum termo de versão aparece em interfaces operacionais.
- Apenas SystemAdmin opera criação de versão, publicação, fork, ativação e arquivamento de templates.

---

## 4. Pontos de integração

**Frontend precisa saber:**
- Estado atual de cada template (Rascunho, Ativo, Em transição, Arquivado) para exibir os controles corretos.
- Quais campos são editáveis em cada estado: todos em Rascunho; nome e descrição em Ativo; nenhum em Em transição e Arquivado.
- Se a ação de fork está disponível (somente em Ativo).
- Se a ação de ativar está disponível (somente em Rascunho derivado de fork — identificado por ter `DerivedFromTemplateId` preenchido).
- Se a ação de arquivar manualmente está disponível (Ativo e Em transição) e se está bloqueada, com a contagem de processos em andamento que justifica o bloqueio.
- Para o board: step atual de cada processo em andamento, para determinar em qual coluna o card aparece.
- Conjunto de steps da versão Ativa da linhagem, para montar as colunas base do board.
- Conjunto de steps atualmente ocupados por processos de versões Em transição, para complementar as colunas do board.
- Identificador da versão do template em que cada processo foi criado (para resolução da lógica de colunas, mesmo que não exibido ao contador).

**Backend precisa garantir:**
- Rejeição de fork em qualquer estado que não seja Ativo, com erro descritivo.
- Atomicidade total na ativação da nova versão: all-or-nothing; nenhuma falha parcial pode resultar em dois templates Ativos ou em zero Ativos na linhagem.
- Bloqueio de ativação quando o template-pai do Rascunho já não é mais o Ativo corrente da linhagem.
- Arquivamento automático ao receber o evento de conclusão do último processo em andamento de um template Em transição; idempotente — se o template já estiver Arquivado, o evento é ignorado sem erro.
- Exposição da contagem de processos em andamento por template, para que o frontend informe o bloqueio do arquivamento manual corretamente.
- Garantia do invariante de no máximo uma versão Ativa por linhagem, inclusive sob concorrência.
- Registro de `DerivedFromTemplateId` no fork, para que a cadeia de linhagem seja reconstruível.
- Materialização completa da estrutura do template no processo no momento da sua criação: steps, tasks e fields copiados para o processo, não referenciados dinamicamente.
- Resolução automática da versão Ativa no momento de criação de processo — o contador não informa versão; o sistema resolve e associa.

---

## 5. Casos de borda

**Ativação com template-pai fora do estado Ativo**

Dois SystemAdmins fazem fork do mesmo template Ativo quase ao mesmo tempo. O primeiro ativa sua nova versão — o template-pai passa para Em transição. O segundo tenta ativar a sua. O template-pai já não está Ativo.

Decisão: a ativação é bloqueada. O sistema recusa com mensagem indicando que o template de origem não é mais a versão Ativa corrente. O segundo SystemAdmin precisa criar um novo fork a partir da versão Ativa atual e incorporar suas mudanças manualmente.

---

**Dois Rascunhos derivados da mesma linhagem coexistindo**

O SystemAdmin cria um fork (Rascunho A), não ativa, e cria um segundo fork do mesmo template Ativo (Rascunho B). Dois Rascunhos da mesma linhagem existem simultaneamente.

Decisão: permitido. Cada Rascunho é independente. O primeiro a ser ativado "ganha" — o template-pai passa para Em transição. O segundo fica bloqueado para ativação (caso anterior). O destino do segundo Rascunho depende da resolução do caso de descarte abaixo.

---

**Descarte de um Rascunho**

O SystemAdmin cria um fork (Rascunho) e decide que não quer mais essa nova versão antes de ativá-la.

`[pergunta aberta]` — A spec define arquivamento manual apenas para Ativo e Em transição. O comportamento de descarte de um Rascunho não está especificado. Precisa ser decidido: o SystemAdmin pode descartar (excluir) um Rascunho? Ou ele permanece como Rascunho indefinidamente? A decisão afeta a UX da lista de templates e o destino dos Rascunhos bloqueados pelo caso anterior.

---

**Evento de conclusão de processo chegando em template já Arquivado**

O sistema arquiva manualmente um template Em transição com 0 processos em andamento. Paralelamente, um evento de conclusão de processo chega com referência a esse template (por race condition extrema entre a conclusão do último processo e o arquivamento manual).

Decisão: o arquivamento automático disparado pelo evento deve ser idempotente — se o template já está Arquivado, o evento é processado sem erro e sem efeito. A regra de bloqueio do arquivamento manual (presença de processos em andamento) torna esse cenário praticamente impossível por construção, mas o backend deve tratá-lo graciosamente mesmo assim.

---

**Template Ativo nunca utilizado arquivado manualmente**

O SystemAdmin publica um template (Ativo) e, antes de qualquer processo ser criado, decide arquivá-lo diretamente.

Decisão: permitido. Não há processos em andamento, portanto o bloqueio não se aplica. O template transita de Ativo para Arquivado imediatamente.

---

**Step renomeado entre versões — renderização do board**

A versão Em transição tem um step chamado "Análise". A versão Ativa renomeou esse step para "Análise Detalhada". No board, "Análise" e "Análise Detalhada" são tratados como colunas distintas — não há mesclagem por similaridade de nome. Processos da versão antiga aparecem em "Análise"; processos da versão nova aparecem em "Análise Detalhada". Quando os processos da versão antiga concluem, a coluna "Análise" desaparece naturalmente.

Decisão: colunas são identificadas pelo nome exato do step. Não há inferência de equivalência entre nomes. O board exibe ambas durante a transição.