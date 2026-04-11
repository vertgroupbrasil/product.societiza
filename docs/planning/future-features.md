# Features Futuras Planejadas

> **ARQUIVADO POR TEMPO INDETERMINADO (desde 2026-04-11).** A única task ativa
> do repositório é `societario-workflow-process`. Todas as features abaixo
> permanecem como visão futura documentada, mas não devem ser iniciadas,
> detalhadas em spec, nem distribuídas enquanto o arquivamento não for
> explicitamente revertido pelo humano.

Todas as features abaixo são visão futura. Nenhuma está implementada.

---

## Automação como extensão do workflow

Um dos potenciais mais fortes da Societiza está nas tarefas automatizáveis.
Hoje o contador perde muito tempo consultando repetidamente sites de prefeitura
para verificar se determinada resposta saiu, como viabilidade, alvará ou outra
informação pública. No futuro, essas verificações repetitivas devem poder virar
tarefas automatizadas do próprio workflow.

Essa automação não deve nascer como módulo desconectado. Ela deve ser entendida
como extensão do processo já configurado no workflow — substituindo trabalho
manual repetitivo dentro do fluxo real da contabilidade.

---

## Importação e exportação de processos

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

---

## Notificações

A Societiza deve ter notificações por e-mail e, futuramente, por WhatsApp. Essas
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

---

## Formulário nativo de abertura de empresa

Hoje muitas contabilidades usam ferramentas externas de formulário para coletar
informações como dados administrativos, quadro societário, cotas, nomes, e-mail,
profissão e outras respostas necessárias para abrir a empresa. No futuro,
queremos que a Societiza permita criar esse formulário de forma nativa, com
branding da contabilidade, e que as respostas alimentem diretamente o processo
do cliente dentro da plataforma.

É crucial manter a distinção: isso é uma feature futura de produto. Não deve
ser tratada como capacidade consolidada do MVP atual. O que existe hoje como
verdade operacional são campos configuráveis dentro do workflow-template.

---

## Controle de alvarás

Queremos uma feature de controle de alvarás para que a contabilidade consiga
gerenciar os alvarás dos clientes, acompanhar vencimentos e notificar tanto a
equipe interna quanto o cliente final. A modelagem da ligação com o societário
ainda não está fechada, mas existe uma direção clara: a Societiza pode evoluir
da abertura da empresa para a continuidade regulatória da vida desse cliente.

---

## Certificados digitais em nuvem

Queremos uma feature de controle de certificados digitais em nuvem
criptografados. O problema aqui é sério: muitas contabilidades não seguem
práticas adequadas para armazenar dados sensíveis dos clientes. A Societiza
quer oferecer um local seguro, confiável e prático para armazenar,
disponibilizar download e acompanhar vencimentos desses certificados. Essa
feature se conecta naturalmente com notificações, dados do cliente e, no
futuro, com uma visão mais ampla da vida operacional e regulatória do cliente.

---

## MCP Server e IA operacional

Existe uma visão de MCP Server para conectar uma IA que já rode na máquina do
usuário ou para servir como agente operacional no dia a dia do contador. Essa
IA poderia ajudar em importação, consulta de informações, perguntas sobre
processos e suporte operacional a partir de prompts. Ela precisa conhecer o
contexto real da operação — estado do workflow, tarefas e informações da
contabilidade — e não apenas responder de forma genérica.

---

## Portal do cliente

O portal do cliente permitirá que o empresário (cliente da contabilidade)
acompanhe o status do próprio processo de abertura, faça upload de documentos,
visualize pendências e, no futuro, assine digitalmente documentos. Hoje toda
comunicação com o cliente é fora da plataforma, o que é um gap operacional
importante. O portal depende de notificações para funcionar bem, e de uma
estrutura de autenticação para o cliente que ainda não existe.