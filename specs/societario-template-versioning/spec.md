---
status: approved
domain: societario
created_at: 2026-05-01
approved_at: 2026-05-03
---

# Versionamento e Ciclo de Vida do WorkflowTemplate

## 1. Visão da feature

Hoje, um WorkflowTemplate em uso pode ser alterado ou arquivado a qualquer
momento, sem que o sistema saiba o que isso significa para os processos que
estão rodando em cima dele. Quando isso acontece, o board do contador fica
incoerente — colunas mudam de nome, somem, ou não correspondem mais ao que
processos antigos estão executando. O template é um contrato vivo, e hoje
esse contrato pode ser quebrado silenciosamente.

Esta feature introduz um ciclo de vida explícito para o template e um
mecanismo de versionamento por bifurcação ("fork"). Templates publicados
deixam de aceitar edições estruturais. Para evoluir um template, o
SystemAdmin cria uma nova versão a partir do existente — uma cópia em
rascunho que pode ser editada livremente. No momento em que essa nova
versão é ativada, ela passa a ser usada para novos processos, enquanto a
versão anterior fica em estado de transição, segurando os processos que já
estavam em andamento até que terminem. Quando o último processo daquela
versão termina, ela é arquivada automaticamente.

O resultado, do ponto de vista do contador, é que o board nunca mais fica
incoerente. Todo processo continua rodando exatamente sobre a estrutura
com que nasceu, e o histórico passa a ter valor real porque sabe-se com
qual versão do template cada processo foi conduzido.

---

## 2. Problema que resolve

Hoje o WorkflowTemplate tem três estados — Draft, Active e Archived — e
nenhum deles protege os processos em andamento. O SystemAdmin pode editar
estrutura (steps, tasks, fields) de um template Active sem que o sistema
saiba o impacto disso, e pode arquivar um template em uso sem que processos
ativos sejam considerados.

Isso acontece toda vez que a Societiza identifica que um template precisa
evoluir — novas exigências regulatórias, mudanças no fluxo de uma junta
comercial, refinamento operacional após uso real. A frequência tende a ser
baixa em volume absoluto, mas a severidade é alta: cada edição estrutural
no template ativo introduz inconsistência em um número desconhecido de
processos.

Hoje não há workaround dentro do sistema. O workaround operacional é "evitar
editar o template enquanto há processos rodando", e isso depende inteiramente
da memória e disciplina do SystemAdmin. Não escala, não é auditável e não
sobrevive ao crescimento da operação. Conforme a Societiza adiciona mais
contabilidades e mais processos por contabilidade, a chance de uma edição
acidental causar dano silencioso aumenta.

A camada de execução (WorkflowProcess, já especificada e aprovada) materializa
a estrutura do template no momento da criação do processo — então o processo
em si não é corrompido por uma edição posterior. O dano fica no board: as
colunas do kanban são derivadas do template ativo, e processos antigos com
estruturas diferentes ficam visualmente incoerentes. Sem versionamento, o
board não consegue representar honestamente o que está acontecendo.

---

## 3. Para quem é

**Beneficiário direto:** o SystemAdmin da Societiza, que opera os templates
disponibilizados às contabilidades. É ele quem hoje carrega a
responsabilidade de "não quebrar nada" ao editar um template, e é ele quem
precisa de um mecanismo que torne essa operação segura por design.

**Beneficiário indireto:** toda a contabilidade operando processos. Ela
nunca interage com a interface de versionamento, mas é ela que sofre quando
um template é editado de forma inconsistente. Com versionamento, o contador
ganha estabilidade — processos não mudam de forma sob seus pés.

O contexto de uso é o ciclo de evolução dos templates da Societiza. Quando
um aprendizado operacional aparece (uma etapa precisa ser dividida, uma
tarefa precisa virar opcional, um campo novo precisa ser capturado), o
SystemAdmin entra na interface de templates e decide entre uma edição
cosmética (nome/descrição) ou uma evolução estrutural (criar nova versão).

---

## 4. Como melhora a experiência atual

**Antes:**

O SystemAdmin abre um template Active, edita uma estrutura (adiciona um
step, remove uma task, troca um field) e salva. O sistema aceita
silenciosamente. Processos em andamento agora rodam com uma estrutura
diferente da que foi mostrada no momento da criação. O board mistura
processos com estruturas diferentes na mesma coluna, ou pior, mostra
colunas que não existem mais para alguns processos. O SystemAdmin não tem
sinal nenhum de que algo quebrou. O contador descobre quando reclama —
geralmente tarde, geralmente sem entender o que aconteceu.

Arquivar um template é igualmente perigoso. O SystemAdmin pode arquivar um
template que tem 15 processos InProgress e o sistema permite. Esses
processos continuam rodando, mas o template "oficial" deles deixou de
existir como ativo, e qualquer ferramenta futura que assuma "se está
arquivado, ninguém usa" toma decisão errada.

**Depois:**

O SystemAdmin abre um template Active. Os campos de estrutura (steps,
tasks, fields) estão em modo somente-leitura. Apenas nome e descrição
podem ser editados. Para mudar a estrutura, há uma ação explícita: "Criar
nova versão". Ao acionar, o sistema clona o template inteiro como Draft
e abre o rascunho para edição. O template original **permanece Active** —
continua recebendo novos processos enquanto a nova versão está sendo
preparada. Não há janela de indisponibilidade.

Quando o SystemAdmin termina de editar o rascunho e ativa a nova versão,
acontece uma **transição atômica**: a nova versão se torna Active e o
original se torna Superseded simultaneamente, na mesma operação. A partir
daí, novos processos nascem da nova versão. A anterior fica em Superseded
esperando os processos InProgress concluírem. Quando o último deles termina,
o sistema arquiva a versão antiga sozinho. Se no momento da ativação não
houver processos InProgress no original, ele é arquivado imediatamente.

Do ponto de vista do contador, a operação é invisível. Ele nunca sabe que
houve um fork, nunca vê referências a "versão anterior" ou "v1/v2", nunca
percebe a transição. Sempre existe um template Active da linhagem disponível
para criar novos processos.

Arquivar manualmente um template ativo ou superseded só funciona se não
houver processos InProgress nele. Se houver, o sistema bloqueia e diz por
quê. O SystemAdmin nunca consegue, por engano, deixar processos órfãos.

O resultado é que o template deixa de ser uma configuração editável e
passa a ser um contrato versionado — cada versão tem um ciclo de vida
fechado, e a transição entre versões é segura por construção.

---

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**

- **WorkflowTemplate** — a feature estende o ciclo de vida atual do
  template (Draft, Active, Archived) com um novo estado intermediário
  (Superseded) e regras explícitas de transição.
- **WorkflowProcess** — a feature precisa saber se há processos InProgress
  para um dado template antes de permitir arquivamento manual, e precisa
  reagir à conclusão do último processo InProgress de um template
  Superseded para arquivar automaticamente. Essa comunicação cross-feature
  acontece sem acoplar diretamente os módulos: a verificação "há processos
  InProgress?" é exposta como um serviço consultável, e o evento "processo
  concluído" é publicado como integration event para qualquer feature
  interessada — incluindo esta.

**O que esta feature habilita no futuro:**

- **Histórico fiel de processos concluídos** — saber em qual versão do
  template um processo concluído foi conduzido permite que o histórico
  filtre e renderize por versão, mesmo após arquivamento.
- **Personalização de templates pela contabilidade** — quando essa
  capacidade entrar no roadmap, o mecanismo de versionamento já estará no
  lugar para proteger os processos da contabilidade contra mudanças
  estruturais.
- **Rastreabilidade da evolução do produto** — a referência
  "DerivedFromTemplateId" cria uma cadeia auditável de versões,
  permitindo no futuro entender como um template chegou ao formato atual.

**Posição no produto:**

Feature core de fortalecimento do societário (P1). Não é uma nova
capacidade visível para o contador — é uma capacidade que protege a
operação dele de uma classe de bugs silenciosos. É exatamente o tipo de
trabalho que P1 cobre: melhorar consistência, confiabilidade e capacidade
de configuração do núcleo existente antes de avançar para P2 (dashboard).

---

## 6. O que não é (escopo negativo)

- **Não inclui** migração de processos existentes para uma nova versão do
  template. Quando uma nova versão é ativada, os processos antigos
  continuam executando a versão original até o fim. Migrar processos em
  andamento para uma estrutura diferente é uma classe de problema com
  consequências imprevisíveis e fica explicitamente fora do escopo.
- **Não inclui** comparação visual entre versões (diff de templates). O
  SystemAdmin pode ver as duas versões lado a lado abrindo cada uma, mas
  o sistema não destaca o que mudou.
- **Não inclui** retorno de Superseded para Active. Uma vez que uma nova
  versão é ativada, o caminho da anterior é só arquivamento (manual com
  pré-condições, ou automático ao último processo terminar).
- **Não inclui** múltiplas versões Active simultâneas. Em qualquer momento
  há, no máximo, uma versão Active de uma "linhagem" de template. A
  janela de transição é Superseded + Active, nunca Active + Active.
- **Não inclui** fork a partir de um template Draft ou Archived. O fork
  só faz sentido a partir de um template publicado e em uso (Active).
- **Não inclui** edição estrutural de um template Active mesmo com aviso.
  O bloqueio é absoluto — a única forma de evoluir estrutura é via fork.
- **Não inclui** desfazer um fork. Uma vez que a transição
  Active → Superseded acontece, ela não é revertida. Se o SystemAdmin
  quiser voltar atrás, ele cria mais uma versão.
- **Não inclui** notificação ao contador sobre transição de versão. Como
  o processo dele continua rodando exatamente como nasceu, não há
  necessidade operacional de avisar. Se isso virar uma necessidade, vira
  feature separada.

---

## 7. Custos adicionais

Nenhum custo externo identificado.

A feature opera inteiramente dentro do ecossistema já existente da
Societiza. Não depende de APIs de terceiros, serviços externos, provedores
de e-mail, integrações governamentais ou infraestrutura adicional além da
que já suporta o WorkflowTemplate e o WorkflowProcess. A comunicação
cross-feature entre WorkflowTemplate e WorkflowProcess (verificação de
processos InProgress e consumo do evento de conclusão) usa a mesma
infraestrutura de mensageria interna já prevista para outras features do
MVP.

---

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Novo estado no ciclo de vida | Adicionar `Superseded` entre `Active` e `Archived`. O ciclo passa a ser: Draft → Active → Superseded → Archived, com possibilidade de Active → Archived direto via arquivamento manual quando não há processos InProgress. |
| Edição em Active | Estrutura (steps, tasks, fields) fica somente-leitura. Apenas nome e descrição podem ser editados — são edições cosméticas que não afetam processos. |
| Mecanismo de evolução estrutural | Fork: criar nova versão a partir do template existente. O fork clona toda a estrutura para um novo template em Draft. O original **permanece Active** — não transiciona ainda. A transição só acontece quando a nova versão é ativada. |
| Comportamento de Superseded | Não aceita criação de novos processos. Mantém os processos InProgress já existentes até a conclusão. |
| Critério de uso para novos processos | Novos processos sempre nascem da versão Active mais recente da linhagem. Nunca de Superseded ou Draft. |
| Transição atômica na ativação | Quando a nova versão é ativada, a operação é atômica: nova versão = Active **e** original = Superseded simultaneamente. Garante que sempre existe um template Active da linhagem — não há janela de indisponibilidade entre o fork e a ativação. O contador nunca percebe o fork. |
| Invisibilidade ao contador | O versionamento é 100% invisível para o contador. Nenhum termo como "versão anterior", "v1/v2" ou "fork" aparece em qualquer interface operacional. O contador só vê processos nos seus steps. |
| Rastreio de origem | A nova versão registra referência ao template de origem via `DerivedFromTemplateId`. Permite reconstruir a cadeia de versões. |
| Archive manual | Disponível em Active e em Superseded. Bloqueado se houver qualquer processo com status InProgress associado ao template. O bloqueio é informativo: o sistema diz quantos processos estão impedindo. |
| Archive automático | Quando o último processo InProgress de um template Superseded é concluído, o template é arquivado automaticamente. Se no momento do fork não há processos InProgress no original, o original é arquivado imediatamente (transita Active → Superseded → Archived na mesma operação). |
| Versões simultâneas | No máximo uma versão Active por linhagem. Pode haver múltiplas Superseded simultâneas se houver forks consecutivos rápidos com processos pendentes em cada uma. |
| Reversão de fork | Não há. Uma vez Superseded, o template não volta para Active. Para voltar atrás, cria-se uma nova versão. |
| Quem opera versionamento | Apenas SystemAdmin. Contabilidades não criam, editam ou versionam templates no MVP. |
| Comunicação cross-feature | Verificação "há processos InProgress?" é exposta pela interface `IWorkflowProcessDataService` (declarada em WorkflowTemplate, implementada na Web layer pelo módulo WorkflowProcess). Usada para bloquear archive manual e disparar archive automático. Conclusão de processo é publicada como integration event `WorkflowProcessCompleted` (RabbitMQ/MassTransit), consumido por WorkflowTemplate. Os módulos não se acoplam diretamente. |
| Migração de processos existentes | Fora de escopo. Processos sempre executam até o fim na versão em que nasceram. |
| Edição em Superseded | Template Superseded é inteiramente imutável — nem nome nem descrição podem ser editados. Uma vez que a transição Active → Superseded acontece, o template congela completamente. |
| Restrição de fork por estado | Fork só é permitido a partir de um template Active. É regra de domínio (não apenas restrição de UI): o backend rejeita a tentativa de fork em qualquer outro estado. A UI não exibe a opção para Draft, Superseded ou Archived, e a API retorna erro se a tentativa ocorrer. |
| Renderização do board com versões mistas | O board exibe a união das colunas da versão Active com as colunas atualmente ocupadas por processos de versões Superseded. Cada processo aparece na coluna correspondente ao seu step atual. Processos de versões anteriores não passam por colunas que não existiam na sua versão — simplesmente não aparecem nelas. Conforme os processos da versão anterior concluem, suas colunas exclusivas esvaziam e desaparecem naturalmente. Nenhum rótulo de versão é exibido ao contador. |

---

## 9. Perguntas abertas

- **Indicador visual de versão para o SystemAdmin.** O ciclo de vida fica
  visível na interface de templates como um campo de estado, mas ainda
  não está decidido se a relação entre versões (linhagem, derivação) é
  exibida como uma árvore, uma lista plana ou apenas como referência
  textual em cada template. Decisão de UX a fechar com o desenho da tela.