# Societiza — Contexto de Negócio

## Como usar este documento

Este documento serve como base de contexto de produto para qualquer pessoa ou
IA que trabalhe na Societiza. Ele se destina igualmente a quem pensa produto,
quem implementa frontend ou backend, quem planeja automações, quem discute
evolução de features e quem integra no frontend uma feature já feita no backend.

O objetivo não é ensinar como implementar linha por linha. É explicar com
profundidade o que a Societiza é, que problema resolve, como as features se
conectam, o que já existe, o que é visão futura, e quais obstáculos estruturais
precisam ser considerados em qualquer planejamento.

Regras de uso:
- Não trate como existente uma feature que ainda é futura.
- Pode detalhar como uma feature futura deveria funcionar, desde que fique claro
  que é visão futura e não comportamento já entregue.
- Quando houver conflito entre narrativa de produto e estado técnico atual,
  explicite o gap em vez de fingir que ele não existe.
- Ao planejar algo novo, preserve a coerência com o núcleo do produto.

---

## Quando o objetivo for integrar uma feature no frontend

Se a tarefa é integrar no frontend uma feature que já foi implementada no
backend, o fluxo correto é:

1. Procurar um arquivo `.md` dentro da pasta da feature no código-fonte para
   entender a regra de negócio daquela feature e como ela se encaixa no produto.
2. Procurar em `docs/integration/` pela documentação de integração específica
   daquela feature — formato descrito em `docs/integration/TEMPLATE.md`.
3. Se o nome correto da feature não estiver claro nos documentos, perguntar
   antes de assumir.
4. A integração deve sempre se basear nesses documentos antes de assumir
   qualquer contrato.

---

## 1. O que é a Societiza

A Societiza nasceu para mudar a maneira como o Brasil abre empresas. Hoje, a
operação societária das contabilidades é extremamente fragmentada. Os dados de
um mesmo processo ficam despejados em planilhas, sites de prefeitura, portais
de viabilidade, documentos espalhados, conversas no WhatsApp e em outros
softwares que não compartilham contexto entre si. Isso gera retrabalho, perda
de informação, falta de previsibilidade, dependência excessiva de memória
humana e baixa capacidade de escalar a operação com qualidade.

A Societiza existe para centralizar essa operação e reduzir a burocracia
percebida pelo contador. O objetivo não é apenas organizar tarefas. O objetivo
é transformar a abertura de empresas em uma operação estruturada, clara,
configurável, colaborativa e progressivamente automatizável.

A plataforma deve concentrar toda a regra de negócio da abertura de empresas:
desde a proposta ao cliente, coleta dos dados iniciais, execução operacional,
acompanhamento do andamento, notificações, documentos, pendências e conclusão
da abertura.

### Visão, missão e valores

A visão da Societiza é se tornar o sistema operacional da abertura de empresas
no Brasil para contabilidades. A plataforma não deve ser vista como um conjunto
de telas soltas, nem como apenas um kanban. Ela deve ser a camada central da
operação societária.

A missão é descomplicar a vida do contador. Isso significa reduzir burocracia,
reduzir caos operacional, reduzir dependência de controles improvisados e dar
mais clareza, velocidade e previsibilidade para a equipe da contabilidade e
para o cliente final.

Os valores que devem orientar qualquer decisão de produto são:

- centralização de informação;
- clareza operacional;
- configurabilidade real sem perder simplicidade;
- automação com propósito;
- colaboração em equipe;
- confiabilidade;
- continuidade entre features;
- melhor experiência para contador e cliente;
- escalabilidade operacional para pequenos e médios escritórios.

---

## 2. ICP e proposta de valor

O ICP atual da Societiza são contabilidades de pequeno e médio porte e
contadores autônomos. São operações que normalmente sofrem com controle manual,
uso excessivo de planilhas, dependência de WhatsApp, dificuldade para
padronizar processos e baixa visibilidade sobre gargalos.

A Societiza é um SaaS B2B. O modelo comercial ainda não está totalmente
fechado, mas a direção atual é trabalhar com um plano principal por assinatura
e um freemium. As limitações do freemium ainda não estão definidas, mas hoje a
linha de pensamento está centrada principalmente em quantidade de processos
rodados ou quantidade de processos ativos.

O produto deve ser pensado como software de time. Ele não é apenas um painel
pessoal. Quem compra é um usuário principal, que representa a contabilidade, e
depois convida outras pessoas para a equipe.

A proposta de valor central é: dar ao contador visibilidade e controle sobre
toda a operação societária em um único lugar, substituindo planilhas e
comunicações fragmentadas por um fluxo estruturado, rastreável e configurável.

---

## 3. Times, usuários e permissões

Hoje o produto é vendido como uma estrutura de equipe. Um usuário principal
cria a conta da contabilidade e pode convidar outros membros. No MVP atual,
todos os convidados operam na prática como administradores: podem editar
templates de workflow, criar processos, editar processos e usar a ferramenta de
forma ampla.

O multi-tenancy é organizado por escritório de contabilidade. Isso significa
que cada contabilidade tem sua própria instância de dados, seus próprios
processos, seus próprios templates e seus próprios membros de equipe. A
entidade `accountancy` é o discriminante central dessa separação.

No futuro, faz sentido existir cargos e permissões mais granulares no ato de
convidar usuários, com papéis como administrador, operador, gestor ou
visualizador. Porém isso ainda não está consolidado como comportamento atual do
produto. **Este gap de permissões granulares é um obstáculo atual.**

---

## 4. Autenticação: estado atual e gaps

Do ponto de vista de produto, a autenticação foi concebida com email e senha,
refresh token em cookies e access token em cache, com middleware no backend
validando o access token e, quando necessário, usando o refresh token para
renovar a sessão. No futuro, a intenção é migrar para login apenas com Google,
para evitar armazenar senhas no banco e simplificar a experiência.

Entretanto, é importante registrar um gap técnico atual: o backend existente
no repositório hoje não expõe publicamente esse fluxo completo como contrato
observável de API do mesmo jeito que a narrativa de produto descreve. O backend
atual possui um módulo `Identity`, mas o que está exposto hoje de forma clara
na API são fluxos de convite e aceite de convite, além de um serviço JWT
voltado a password reset. Isso significa que qualquer planejamento envolvendo
autenticação precisa distinguir explicitamente:

- a história de produto desejada;
- o que o frontend hoje espera em alguns pontos;
- e o que o backend efetivamente expõe hoje.

Esse desalinhamento não deve ser escondido. Ele deve ser tratado como gap
arquitetural e de integração.

---

## 5. O societário como feature central

O societário é o núcleo do produto. Tudo na Societiza orbita em torno do fluxo
de abertura de empresa. O societário é um quadro kanban sem drag-and-drop no
uso operacional do workflow, com colunas ordenadas que representam as etapas
pelas quais um processo passa desde o início até a conclusão.

Cada processo representa uma abertura de empresa ou um caso semelhante dentro
da contabilidade. O processo contém dados básicos da empresa sendo aberta,
vinculação com um escritório de contabilidade (accountancy), e uma série de
tarefas configuradas pelo template do workflow. Dentro do processo, existem
tarefas e campos configuráveis que precisam ser suficientemente claros para que
qualquer membro da equipe entenda em que ponto o caso está, o que falta, o que
está travado e o que precisa ser feito em seguida.

Do ponto de vista de experiência do usuário, o societário deve transmitir
controle, previsibilidade e continuidade. O contador não deve sentir que está
apenas preenchendo software. Ele deve sentir que está tocando a operação real
da contabilidade em um único lugar. O board precisa mostrar rapidamente o
estado de cada processo, os responsáveis, as pendências e os gargalos.

O societário se conecta diretamente com o workflow template (que define as
etapas e tarefas), com a accountancy (que identifica a qual escritório o
processo pertence), e será a fonte primária de dados para o dashboard e para
futuras features de notificação.

---

## 6. Workflow e Workflow Template

### O board (workflow)

O workflow é a camada onde o usuário acompanha e opera processos. É um board
kanban com colunas que representam etapas. Cada card é um processo de abertura
de empresa. O usuário pode filtrar por contabilidade, responsável e etapa,
abrir um drawer com os detalhes do processo, mover processos de etapa e
interagir com as tarefas de cada processo.

A experiência esperada é de visão operacional imediata: ao abrir o board, o
contador vê exatamente onde cada processo está, o que está parado, o que
precisa de atenção e o que está avançando. A ausência de drag-and-drop é uma
decisão atual; a movimentação de etapa acontece de forma explícita.

### O template de workflow

O workflow template é a camada de configuração do societário. Ele permite que
a contabilidade defina suas etapas, tarefas e campos de acordo com o processo
operacional que já utiliza no mundo real. Se a contabilidade já possui um
processo consolidado, a plataforma deve permitir reproduzi-lo. Se o usuário
não quiser configurar tudo do zero, ele deve conseguir usar um template pronto.

O template é composto por:
- **Steps (etapas)**: cada etapa tem um título, uma ordem e pode ter campos
  próprios (step fields) que se aplicam a toda a etapa, não a uma tarefa
  específica. A ordem das etapas carrega semântica operacional.
- **Tasks (tarefas)**: cada etapa pode ter múltiplas tarefas. As tarefas têm
  título, tipo, indicação de opcionalidade, uma ordem e campos configuráveis.
  Os tipos de tarefa existentes são: Form, Checklist, Document e Approval.
- **Fields (campos)**: cada tarefa pode ter campos configuráveis. Os tipos de
  campo suportados hoje são Text e Select. Campos Select têm uma lista de
  opções configurável. A lista de opções chega do backend como JSON string e
  é parseada na camada de serviço do frontend.

O template precisa refletir o modelo mental do uso real do workflow. Ordem de
etapas, ordem de tarefas e ordem de campos importam. A estrutura não é
decorativa; ela carrega semântica operacional.

### Template ativo versus draft

Um template pode estar em estado Ativo, Inativo ou Arquivado. O ativo
representa o fluxo em uso real. Um draft representa uma versão editável para
alterações que ainda não foram publicadas. Planejamentos e implementações
nessa área não podem confundir essas duas coisas.

A versão do template é uma string (exemplo: "1.0"). Ela não muda
automaticamente ao editar — representa a versão publicada e aprovada.

### Campos dentro de tarefas e etapas — atenção ao contexto

Hoje existe no produto a ideia de inputs de texto, caixas de seleção e outros
campos configuráveis dentro de etapas e tarefas do template do workflow. Esses
campos são o mecanismo atual para capturar e estruturar parte das informações
operacionais necessárias durante a execução do processo.

**O produto ainda não possui, como feature consolidada de MVP, um formulário
nativo completo de abertura de empresa como módulo principal e oficial do
produto.** O que existe hoje, do ponto de vista de regra de negócio
consolidada, são campos configuráveis dentro do workflow-template. A direção
futura é ter um formulário nativo com branding da contabilidade onde o cliente
preenche os dados que alimentam o processo, mas isso ainda não existe como
capacidade entregue. Não trate a feature futura de formulário como se já fosse
uma capacidade principal do MVP.

---

## 7. Accountancy

A feature de accountancy representa a contabilidade dentro da plataforma. Ela
é o contorno organizacional do produto — o escritório de contabilidade que
compra e usa a Societiza. A accountancy guarda dados básicos do escritório
(nome legal, nome fantasia, CNPJ, telefone, e-mail) e é o ponto de partida
para a criação de times e para a separação de dados entre clientes diferentes.

A importância da accountancy vai além de ser apenas um cadastro. Ela é o
discriminante multi-tenant do sistema: convites, processos, templates e futuras
funcionalidades de branding partem do contexto da contabilidade. Mesmo quando
uma feature parecer focada em workflow, deve-se lembrar que a entidade da
contabilidade é o contorno organizacional do produto.

O que está implementado hoje: criação, edição, listagem de contabilidades,
vinculação com processos no board via filtro. O que ainda é gap: branding
personalizado por contabilidade, configurações avançadas por escritório,
multi-contabilidade gerenciada por um único admin.

---

## 8. Dashboard — próxima feature mandatória

O dashboard ainda não existe como feature entregue. Ele é a próxima grande
prioridade após as melhorias no societário.

Do ponto de vista de produto, o dashboard deve ser a camada de leitura da
operação, enquanto o societário continua sendo a camada de execução. Ele não
pode ser passivo — cada número, card ou gráfico que represente um conjunto
operacional de dados deve, idealmente, permitir navegação para a área
operacional correspondente.

O mental model obrigatório: se existir um gráfico com os processos mais
parados, ao clicar em um segmento desse gráfico, o usuário deve ir para o
societário com um filtro aplicado, vendo exatamente os processos ligados a
esse insight. Insight deve virar ação.

Os dados mais promissores para esse dashboard incluem: total de processos
ativos, total de processos concluídos em um período, distribuição de processos
por etapa, processos mais parados, etapas com maior acumulação, processos
aguardando cliente, tarefas pendentes, movimentações recentes, alertas e
riscos, gargalos operacionais.

Do ponto de vista técnico, o dashboard depende do workflow (fonte dos dados de
processo) e da accountancy (filtro multi-tenant por escritório). Ele não deve
nascer como módulo analítico isolado. Deve resumir a operação real e levar o
usuário de volta ao trabalho com contexto e filtros.

A definição exata das métricas, visualizações e nível de interatividade ainda
não está fechada. Qualquer planejamento dessa feature deve combinar a visão de
produto com a realidade operacional do societário.

---

## 9. Features futuras planejadas

### 9.1. Automação como extensão do workflow

Um dos potenciais mais fortes da Societiza está nas tarefas automatizáveis.
Hoje o contador perde muito tempo consultando repetidamente sites de prefeitura
para verificar se determinada resposta saiu, como viabilidade, alvará ou outra
informação pública. No futuro, essas verificações repetitivas devem poder virar
tarefas automatizadas do próprio workflow.

Essa automação não deve nascer como módulo desconectado. Ela deve ser entendida
como extensão do processo já configurado no workflow — substituindo trabalho
manual repetitivo dentro do fluxo real da contabilidade. Essa automação não
faz parte do MVP atual consolidado.

### 9.2. Importação e exportação de processos

Importação e exportação são estrategicamente importantes porque a adoção do
produto depende de o usuário conseguir migrar parte da operação sem sentir que
precisa recomeçar do zero.

Hoje existem duas direções principais imaginadas: importação por planilha e
importação assistida por IA via MCP Server. A importação por planilha tem
riscos reais — exige colunas perfeitamente padronizadas, não representa bem
estrutura de tarefas e campos, e dificilmente reconstrói com inteligência o
estado real do processo. A importação por IA é mais promissora, mas está fora
das prioridades imediatas do MVP.

Exportação também é importante para confiança do usuário e para eventual
relatório, auditoria, backup ou migração futura.

### 9.3. Notificações

A Societiza deve ter notificações por email e, futuramente, por WhatsApp. Essas
notificações devem atender tanto o cliente final quanto a equipe interna da
contabilidade. Para o cliente, faz sentido notificar movimentação de etapa,
mudanças importantes no andamento e necessidades de ação. Para a contabilidade,
faz sentido alertar processos parados, gargalos, etapas travadas e risco de
atraso.

Essa feature deve ser pensada como extensão do acompanhamento operacional e
não como um canal de mensagens isolado. Ela se conecta diretamente com o
workflow (eventos de mudança de etapa, tarefas vencidas) e com a accountancy
(contexto do escritório). Notificações também habilitam o portal do cliente no
futuro.

**Esta feature ainda não existe.**

### 9.4. Formulário nativo de abertura de empresa

Hoje muitas contabilidades usam ferramentas externas de formulário para coletar
informações como dados administrativos, quadro societário, cotas, nomes, email,
profissão e outras respostas necessárias para abrir a empresa. No futuro,
queremos que a Societiza permita criar esse formulário de forma nativa, com
branding da contabilidade, e que as respostas alimentem diretamente o processo
do cliente dentro da plataforma.

É crucial manter a distinção: isso é uma feature futura de produto. Não deve
ser tratada como capacidade consolidada do MVP atual. O que existe hoje como
verdade operacional são campos configuráveis dentro do workflow-template.

**Esta feature ainda não existe.**

### 9.5. Controle de alvarás

Queremos uma feature de controle de alvarás para que a contabilidade consiga
gerenciar os alvarás dos clientes, acompanhar vencimentos e notificar tanto a
equipe interna quanto o cliente final. A modelagem da ligação com o societário
ainda não está fechada, mas existe uma direção clara: a Societiza pode evoluir
da abertura da empresa para a continuidade regulatória da vida desse cliente.

**Esta feature ainda não existe.**

### 9.6. Certificados digitais em nuvem

Queremos uma feature de controle de certificados digitais em nuvem
criptografados. O problema aqui é sério: muitas contabilidades não seguem
práticas adequadas para armazenar dados sensíveis dos clientes. A Societiza
quer oferecer um local seguro, confiável e prático para armazenar,
disponibilizar download e acompanhar vencimentos desses certificados. Essa
feature se conecta naturalmente com notificações, dados do cliente e, no
futuro, com uma visão mais ampla da vida operacional e regulatória do cliente.

**Esta feature ainda não existe.**

### 9.7. MCP Server e IA operacional

Existe uma visão de MCP Server para conectar uma IA que já rode na máquina do
usuário ou para servir como agente operacional no dia a dia do contador. Essa
IA poderia ajudar em importação, consulta de informações, perguntas sobre
processos e suporte operacional a partir de prompts. Ela precisa conhecer o
contexto real da operação — estado do workflow, tarefas e informações da
contabilidade — e não apenas responder de forma genérica.

**Esta feature ainda não existe.**

### 9.8. Portal do cliente

O portal do cliente permitirá que o empresário (cliente da contabilidade)
acompanhe o status do próprio processo de abertura, faça upload de documentos,
visualize pendências e, no futuro, assine digitalmente documentos. Hoje toda
comunicação com o cliente é fora da plataforma, o que é um gap operacional
importante. O portal depende de notificações para funcionar bem, e de uma
estrutura de autenticação para o cliente que ainda não existe.

**Esta feature ainda não existe.**

---

## 10. Obstáculos atuais e gaps estruturais

### 10.1. Migração de dados para dentro do sistema

Este é um dos maiores desafios de produto. Os dados atuais dos usuários estão
espalhados em planilhas, WhatsApp, emails, cadernos operacionais, ERPs e sites
públicos. Trazer isso para dentro da plataforma de forma confiável é uma dor
grande, e qualquer funcionalidade de importação precisa reconhecer que a origem
será, em muitos casos, despadronizada e sem semântica limpa. Não existe hoje
nenhuma funcionalidade de importação disponível. Isso é um bloqueio real para
contabilidades que querem adotar o produto mas têm processos em andamento em
outros sistemas.

### 10.2. Variação de regra de negócio por cidade, estado e prefeitura

O Brasil não tem um fluxo único de abertura de empresa. Cada estado, cidade e,
em muitos casos, cada prefeitura relevante pode impor regras, sequências,
exigências, portais, nomenclaturas e limitações diferentes. Isso significa que
qualquer automação, template pronto ou fluxo considerado "padrão" precisa ser
tratado com cuidado. O produto precisa ser suficientemente configurável para
acomodar essas diferenças, sem assumir que existe um processo nacional
totalmente uniforme. Hoje o sistema de templates é flexível, mas não tem
suporte explícito a localidade ou perfil regional.

### 10.3. Dependência de sistemas externos instáveis

Boa parte da operação societária depende de sites públicos e sistemas externos
que podem ser lentos, confusos, instáveis, ter captcha, mudar layout sem aviso
ou ficar fora do ar. Isso impacta diretamente futuras automações e deve ser
considerado nas decisões de produto, engenharia e operação. Qualquer feature
de automação precisa ser projetada assumindo que o sistema externo pode falhar,
e o usuário precisa de visibilidade sobre o status dessas automações.

### 10.4. Gap entre narrativa de produto e contratos técnicos atuais

Algumas histórias do produto já estão claras, mas o backend atual ainda não
implementa ou não expõe exatamente esses contratos da forma esperada. O caso
da autenticação é o exemplo mais importante. Isso significa que planejamento
estratégico e planejamento técnico precisam sempre explicitar o que já existe
de verdade, o que é comportamento esperado de produto, e o que ainda precisa
ser alinhado entre frontend, backend e infra.

### 10.5. Permissões ainda pouco granulares

Hoje o modelo de permissão é simples e todos os convidados acabam com poderes
amplos. Isso pode virar gargalo conforme o produto evoluir para operações com
mais pessoas e mais sensibilidade de ação — por exemplo, quando um contador
júnior não deve ter poder de alterar templates ou arquivar processos.

### 10.6. Colaboração com o cliente

Hoje o cliente (empresário) não tem acesso ao sistema. Toda comunicação é fora
da plataforma — por WhatsApp, email ou telefone. O contador precisa sair do
sistema para falar com o cliente, pedir documentos ou atualizar sobre o
andamento. Isso é um gap de produto importante que só será resolvido com o
portal do cliente.

### 10.7. Dashboard sem definição final de métricas

Sabemos que o dashboard é obrigatório e estratégico, mas ainda não existe
definição fechada de métrica, fonte de verdade, visualização final ou
interatividade completa. O planejamento dessa feature precisa combinar visão de
produto com a realidade operacional do societário.

### 10.8. Dívida técnica no código atual

O código atual tem algumas áreas de dívida conhecida:
- Componentes da feature `workflow/` contêm vários casts `any` e warnings de
  `exhaustive-deps`.
- `useProcessForm.tsx` tem cálculos computados fora de `useMemo`.
- `Board.tsx` tem referências potencialmente instáveis em `applyFilters` e
  `processes`.

---

## 11. Como novas features devem se comportar

Qualquer feature nova deve fortalecer o núcleo da plataforma. Ela não deve
nascer isolada, nem competir com o societário como centro operacional.

Perguntas obrigatórias ao planejar algo novo:
- Isso centraliza ou espalha ainda mais a informação?
- Isso reduz retrabalho real da contabilidade?
- Isso melhora a clareza operacional?
- Isso conversa com a equipe como unidade, e não apenas com um usuário isolado?
- Isso pode se conectar ao workflow ou ao dashboard?
- Isso melhora a experiência do cliente final ou da equipe interna?
- Isso respeita a diferença entre o que já existe e o que ainda é visão futura?

---

## 12. Semântica de prioridade

### Prioridade 1 — Fortalecer o núcleo operacional atual

Melhorar societário, workflow-template, consistência de uso, qualidade da
operação, clareza da interface, confiabilidade e capacidade de configuração.

### Prioridade 2 — Entregar o dashboard

O dashboard é a próxima feature grande e mandatória após as melhorias no
societário. Deve ser uma camada de leitura operacional e gerencial, interativa
e conectada ao societário.

### Prioridade 3 — Melhorar a capacidade de acompanhamento e continuidade

Aqui entram notificações, leituras de gargalo, visibilidade de parados e outras
funcionalidades que ampliem a capacidade de acompanhamento da operação.

### Prioridade 4 — Reduzir fricção de entrada e aumentar automação

Aqui entram importação, exportação, MCP Server, IA assistiva e automação de
tarefas repetitivas.

### Prioridade 5 — Expandir o produto para a vida operacional do cliente

Aqui entram controle de alvarás, certificados digitais e outros módulos que
expandem a plataforma da abertura para a manutenção da vida regulatória do
cliente.

---

## 13. Guardrails para qualquer IA ou time usando este contexto

- Não invente features atuais que ainda não existem.
- Não rebaixe nenhuma informação acima para uma frase superficial.
- Ao falar de uma feature existente, descreva seu papel no produto e sua
  experiência esperada, não apenas o nome da feature.
- Ao falar de uma feature futura, deixe claro que ela é futura.
- Preserve a Societiza como plataforma central da operação societária.
- Trate o societário como o núcleo atual do produto.
- Trate o dashboard como a próxima feature mandatária.
- Trate os campos em etapas e tarefas como a forma atual de captura de dados,
  sem confundir isso com um módulo oficial e consolidado de formulário de
  abertura no MVP.
- Explicite gaps técnicos quando existirem, em vez de mascarar esses gaps com
  linguagem vaga.
