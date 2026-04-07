# Rastreamento do Processo Societário

> Status: aprovada
> Criado em: 2026-04-07

---

## 1. Visão da feature

Quando um cliente contrata uma contabilidade para abrir sua empresa, ele entra
num processo que pode durar semanas. Durante esse tempo, ele não sabe o que está
acontecendo. Não sabe em que etapa está. Não sabe o que falta. Não sabe se o
contador esqueceu. Essa incerteza gera ansiedade, e a ansiedade gera mensagens
— no WhatsApp, por e-mail, por ligação.

O rastreamento do processo societário resolve isso com um link. No momento em
que o contador cria o processo, um link de rastreamento é gerado automaticamente.
O contador manda esse link para o cliente. O cliente abre o link e vê, de forma
clara e visual, todas as etapas pelas quais a abertura da sua empresa vai passar
— em que etapa está agora, o que já foi concluído, o que vem a seguir.

Sempre que o processo avança de etapa, o cliente recebe um e-mail de notificação.
Sem precisar pedir. Sem precisar perguntar. O contador não precisa lembrar de
avisar. O processo fala por si.

A experiência para o cliente é parecida com o rastreamento de uma encomenda dos
Correios: clara, progressiva, sem exigir login, sem exigir conhecimento técnico.
A diferença é que o objeto rastreado é a empresa que ele está abrindo.

---

## 2. Problema que resolve

O cliente da contabilidade (o empresário que está abrindo empresa) não tem
nenhuma janela de visibilidade sobre o andamento do processo. Toda informação
chega por iniciativa do contador — quando o contador lembra de avisar. Quando
não lembra, o cliente entra em contato.

Esse contato acontece por WhatsApp, ligação ou e-mail. O contador responde,
verifica no sistema, atualiza o cliente, volta para o trabalho. Esse ciclo se
repete múltiplas vezes por processo, por múltiplos clientes simultaneamente.

O problema é duplo:
- Para o cliente: ansiedade e sensação de falta de controle sobre algo importante
- Para o contador: interrupções constantes que consomem tempo operacional

Nenhum dos dois quer que isso aconteça. O cliente não quer ter que perguntar.
O contador não quer ter que parar para responder. A ausência de visibilidade
força esse comportamento.

---

## 3. Para quem é

**Beneficiário direto: o cliente da contabilidade** — o empresário (pessoa
física ou jurídica) que contratou a contabilidade para abrir uma empresa. Ele
não usa a Societiza. Ele nunca vai usar. Ele só recebe um link. Esse link
precisa ser autoexplicativo, sem nenhuma curva de aprendizado, sem nenhum
cadastro.

**Beneficiário indireto: o contador** — que para de ser interrompido por
perguntas de status. O alívio operacional acontece do lado do contador, mas
a feature é experienciada pelo cliente.

Perfil do cliente que recebe o link: em geral, um empreendedor abrindo seu
primeiro negócio. Não tem familiaridade com processos burocráticos. Não
conhece o vocabulário do registro de empresa. Pode estar ansioso porque abrir
a empresa é um passo importante na sua vida. A interface que ele vê precisa
ser simples o suficiente para que qualquer pessoa entenda na primeira leitura.

---

## 4. Como melhora a experiência atual

**Antes:**

O cliente manda mensagem para o contador perguntando "como está o andamento?"
O contador para o que está fazendo, abre o sistema, verifica em que etapa está
o processo, redige uma resposta, manda de volta. Isso acontece de 2 a 5 vezes
por processo. Em uma contabilidade com 30 processos abertos, isso é ruído
constante.

O cliente, mesmo depois de receber a resposta, não tem contexto do que vem a
seguir. Na próxima semana, pergunta de novo.

**Depois:**

O contador cria o processo e imediatamente tem um link para compartilhar. Manda
uma vez para o cliente — por WhatsApp, e-mail, onde preferir.

O cliente abre o link e vê um kanban com todas as etapas do processo — os nomes
das colunas, qual está ativa, quais já foram concluídas. É o mesmo template que
o contador configurou, exposto de forma legível. Não há tarefas internas, não há
detalhes operacionais. Só a sequência de etapas e onde o processo está agora.

Quando a etapa muda, o cliente recebe um e-mail. Não precisa perguntar. Não
precisa voltar ao link para checar. O processo o notifica.

O contador segue trabalhando sem interrupção.

---

## 5. Como se conecta com o produto existente

O societário é a fonte de verdade desta feature. Ele já tem tudo que é necessário:
etapas ordenadas, status de progresso, processo vinculado a uma contabilidade.

O rastreamento não cria uma estrutura nova. Ele expõe de forma pública e legível
o que o societário já representa internamente. Cada vez que um processo avança
de etapa no board do contador, essa mudança é refletida automaticamente na página
de rastreamento do cliente.

**Dependências obrigatórias:**
- `workflow` — é a origem de todos os dados: etapas, status, progresso do processo
- `workflow-template` — define a sequência completa de etapas que o cliente vai ver

**O que esta feature abre para o futuro:**

O link de rastreamento é o primeiro ponto de contato digital entre a Societiza
e o cliente do contador. É a semente do portal do cliente. No futuro, esse
mesmo link pode evoluir para:
- Coleta de documentos do cliente via link público
- Aprovações do cliente em etapas que exigem assinatura ou confirmação
- Acesso autenticado ao portal completo do cliente

Esta feature não exige o portal do cliente para existir. Ela antecipa o
problema que o portal vai resolver, de forma mais simples e imediata.

**Posição no roadmap:**

Esta feature é uma versão focada que toca em P3 (notificações e
acompanhamento) sem exigir a infraestrutura completa do portal do cliente (P5).
Ela pode ser construída antes do portal, funciona de forma independente, e
prepara o terreno para ele.

---

## 6. O que não é (escopo negativo)

**Não é um portal do cliente.** O cliente não faz login. Não tem conta. Não
pode interagir com o processo. Não pode enviar documentos. Não pode escrever
comentários. O link é somente leitura.

**Não é uma interface de comunicação.** Não há chat. Não há formulário de
contato. Não há botão de "falar com o contador". Se o cliente quiser falar com
o contador, ele usa os canais que já usa.

**Não é um relatório de progresso interno.** A página de rastreamento é para o
cliente, não para o contador. O contador tem o board do societário. O cliente
tem o link. São interfaces diferentes, com propósitos diferentes.

**Não é um sistema de tarefas visíveis.** O cliente vê etapas — não tarefas
internas. As tarefas do workflow são operacionais, da equipe da contabilidade.
O cliente não precisa saber que existe uma tarefa "Enviar documentação para a
Junta Comercial". Ele precisa saber que o processo está na etapa "Registro na
Junta Comercial".

**Não expõe dados sensíveis de outros clientes ou processos.** Cada link dá
acesso a exatamente um processo. Não há listagem de outros clientes. Não há
informações da contabilidade além do que for necessário para contextualizar o
processo.

**Não inclui notificação por WhatsApp nesta versão.** A notificação desta
versão é por e-mail. WhatsApp exige integração com provedores externos (Twilio,
Zenvia, Z-API), tem custo recorrente e requer aprovação de template de mensagem.
É uma evolução natural, mas não é escopo desta entrega.

---

## 7. Custos adicionais

**Infraestrutura de e-mail própria** — o envio de e-mail é operado pela própria
Societiza, sem depender de provedores externos visíveis para o cliente. Os
e-mails são enviados como se fossem da própria contabilidade: com o nome dela,
a identidade visual dela e o e-mail dela como remetente. O cliente não sabe que
a Societiza existe. Para o cliente, é a contabilidade que está se comunicando.

Isso exige que a Societiza gerencie a reputação de envio, autenticação de
domínio (SPF, DKIM) e escalabilidade da infraestrutura de e-mail. É um
investimento de infraestrutura, não uma dependência externa de custo variável
por cliente.

**Plano:** link de rastreamento e notificações por e-mail estão disponíveis em
todos os planos, incluindo o gratuito. Não há restrição de plano para esta
feature.

---

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| O que o cliente vê | Kanban com os nomes das etapas do template do contador. Nenhuma tarefa interna, nenhum detalhe operacional. Apenas etapas e a posição atual do processo. |
| Expiração do link | O link permanece ativo durante todo o processo. Expira 30 dias após a conclusão do processo. |
| Identidade dos e-mails | Os e-mails são enviados como se fossem da própria contabilidade — nome, identidade visual e e-mail remetente dela. O cliente não tem contato visual com a Societiza. |
| Infraestrutura de e-mail | Própria da Societiza. Sem dependência de provedor externo visível. |
| E-mail do cliente | Não é coletado nesta versão. A notificação por e-mail será habilitada futuramente, quando o campo de e-mail do cliente for introduzido no processo. |
| Plano | Disponível em todos os planos, incluindo gratuito. |
| Como o link é compartilhado | Dentro da sheet que o contador usa para avançar o processo. É o contexto natural: o contador opera o processo e, a partir dali, copia o link para compartilhar com o cliente. |
| Rastreamento de acesso do cliente | Fora de escopo. O contador não recebe nenhum sinal sobre se o cliente abriu o link. |

---

## 9. Perguntas abertas

Nenhuma. Todas as decisões de produto estão fechadas.
