# Escritório e Planos

> Status: ARQUIVADA POR TEMPO INDETERMINADO (desde 2026-04-11)
> Status anterior: aguardando aprovação
> Criado em: 2026-04-07
>
> **Arquivamento:** a partir de 2026-04-11, a única task ativa do repositório
> é `societario-workflow-process`. Esta spec não deve ser aprovada nem avançar
> no ciclo enquanto o arquivamento não for explicitamente revertido pelo humano.

---

## 1. Visão da feature

Um Escritório é o espaço de trabalho de um escritório de contabilidade dentro
da Societiza. Quando alguém cria uma conta na plataforma, o primeiro passo é
criar — ou entrar em — um Escritório. É dentro dele que tudo acontece: os
processos de abertura de empresa, os templates de workflow configurados pela
equipe, os membros da contabilidade, e o plano de assinatura.

Cada Escritório tem uma identidade real: CNPJ, nome legal, nome fantasia,
endereço, telefone, e-mail, foto de perfil e banner. Não é um workspace
genérico — é a representação digital do escritório de contabilidade que o
contador já conhece no mundo físico.

Um usuário pode fazer parte de mais de um Escritório ao mesmo tempo — como
acontece no Notion ou no Canva. Um contador que trabalha em dois escritórios
diferentes usa a mesma conta e alterna entre os dois contextos. Ao trocar de
Escritório, tudo o que ele vê muda: os processos são daquele escritório, os
templates são daquele escritório, a equipe é daquele escritório. Os dados nunca
se misturam.

O plano de assinatura pertence ao usuário, não ao Escritório. Quando o contador
que administra um Escritório assina o plano Escrivaninha ou Executivo, os
Escritórios que ele criou se tornam premium — e todos os membros dentro deles
passam a operar com as capacidades do plano pago, sem precisar pagar
separadamente.

---

## 2. Problema que resolve

Hoje a Societiza não tem planos implementados. Qualquer conta criada tem acesso
irrestrito a tudo, sem distinção entre quem paga e quem não paga. Isso torna o
modelo comercial inviável: não há como limitar o uso gratuito, não há como
cobrar pelo crescimento da equipe, e não há razão concreta para um escritório
fazer upgrade.

Além disso, um contador não consegue fazer parte de mais de um escritório na
mesma conta. Se ele atua em dois lugares, precisa criar contas separadas —
perdendo a centralização que é a proposta central do produto.

Por fim, não havia um conceito de "Escritório" claro e gerenciável pelo próprio
usuário. O único registro de uma contabilidade no sistema era uma entidade
administrativa da Societiza, não algo que o contador criasse, configurasse e
sentisse como seu.

---

## 3. Para quem é

- **Owner do Escritório**: o profissional que cria o Escritório na Societiza.
  É o responsável pelo espaço — gerencia membros, configurações e, quando
  aplicável, o plano de assinatura. O plano que ele assina determina as
  capacidades dos Escritórios que ele criou.
- **Membros do Escritório**: contadores e colaboradores convidados pelo owner
  para operar dentro do Escritório. Se beneficiam do plano pago sem precisar
  assinar individualmente.
- **Contadores com múltiplos vínculos**: profissionais que atuam em mais de um
  escritório e precisam trocar de contexto dentro da mesma conta, sem gerenciar
  múltiplos acessos.

---

## 4. Como melhora a experiência atual

**Antes:**
- Criar conta = ter acesso total sem nenhuma configuração de contexto
- Um usuário = um único contexto de trabalho (e mesmo isso sem clareza)
- Sem planos, sem limites, sem razão para pagar
- Sem identidade real do escritório na plataforma

**Depois:**
- Criar conta → criar um Escritório com identidade real: CNPJ, nome, foto,
  banner
- Trocar de Escritório em segundos — cada um com seus próprios processos,
  templates e equipe
- Plano Free com limitações claras e visíveis → quando o escritório cresce,
  há uma razão concreta para fazer upgrade
- O owner gerencia equipe e plano em um único lugar

---

## 5. Os planos

### Grátis

Para contadores que estão começando ou querem experimentar a plataforma. Não
é um produto incompleto — é o produto inteiro para um contador que trabalha
sozinho e está construindo sua base de clientes.

**Capacidades:**
- 1 Escritório próprio (Free)
- Entrada em mais 1 Escritório de outra pessoa (qualquer plano)
- Total de Escritórios acessíveis: **2**
- **5 processos criados** no Escritório próprio (qualquer processo criado conta,
  independente de status ou conclusão)
- **1 membro** no Escritório (só o próprio owner)
- Templates sem limite de criação — o contador pode criar quantos quiser
- **1 template ativo por vez** — para ativar outro, o atual é congelado junto
  com os processos vinculados a ele (não destruído, não recuperável como "ativo"
  sem reativar)
- Sem automações de workflow
- Sem convite de membros
- Identidade completa do Escritório: CNPJ, nome, foto de perfil, banner,
  endereço, telefone, e-mail
- Rastreamento de processo (link público para o cliente) incluso

**O argumento do Free em uma frase:**
O contador que trabalha sozinho com até 5 clientes em abertura tem tudo que
precisa. O dia que precisar de mais, o produto já provou que merece ser pago.

---

### Escrivaninha

Para contadores que trabalham em equipe ou operam com mais volume.

- 1 Escritório próprio premium
- Entrada em mais 2 Escritórios de outras pessoas (qualquer plano)
- Total de Escritórios acessíveis: **3**
- **50 processos criados** no Escritório próprio
- **3 membros** no Escritório (incluindo o owner)
- Múltiplos templates ativos simultaneamente
- Automações de workflow
- Convite de membros por e-mail e por link
- Cobrança mensal ou anual (anual com desconto)

---

### Executivo

Para contadores que gerenciam múltiplos escritórios ou operações de maior escala.

- Até **3 Escritórios próprios premium**
- Entrada em mais **2 Escritórios** de outras pessoas
- Total de Escritórios acessíveis: **5**
- **200 processos criados** por Escritório próprio
- **8 membros** por Escritório (incluindo o owner)
- Múltiplos templates ativos simultaneamente
- Automações de workflow
- Convite de membros por e-mail e por link
- Cobrança mensal ou anual (anual com desconto)

---

## 6. Regras de negócio

### O plano é do usuário, herda para os Escritórios que ele criou

Quando um usuário assina um plano, os Escritórios que ele criou se tornam
premium. Os membros desses Escritórios se beneficiam das capacidades do plano
pago sem precisar pagar individualmente. Se o usuário cancelar, o plano dura
até o fim do ciclo já pago — depois, os Escritórios caem para Free.

### Limite de processos criados é por Escritório próprio

O limite (5, 50, 200) é por Escritório que o usuário é owner. Todo processo
criado conta no limite, independente de status ou conclusão. Não é uma cota
total entre todos os Escritórios.

### Downgrade = freeze

Quando um Escritório cai de plano, os dados existentes ficam intactos. Nenhum
processo é deletado, nenhum membro é removido automaticamente. O que muda:

- Se o número de **membros** excede o limite do novo plano: o Escritório
  congela para todos exceto o owner, que pode acessar e remover membros até
  atingir o limite permitido. Quando o headcount está dentro do limite, o
  Escritório descongela automaticamente.
- Se apenas o número de **processos** excede (mas membros estão dentro do
  limite): o Escritório não congela para membros — todos podem acessar e ajudar
  a deletar processos até atingir o limite. Não é possível criar novos processos
  até isso acontecer.
- Se ambos excedem: o comportamento de membros tem precedência — congela para
  todos exceto o owner, que resolve os membros primeiro.

### Downgrade com múltiplos Escritórios premium

Quando o usuário faz downgrade e possui mais Escritórios premium do que o novo
plano permite (ex: 3 Escritórios no Executivo → downgrade para Escrivaninha que
permite 1), o owner escolhe qual Escritório mantém o plano premium. Os demais
congelam seguindo a regra de freeze acima.

### Ownership — transferência voluntária

O owner pode transferir a ownership de um Escritório para qualquer membro, a
qualquer momento, através do gerenciamento do Escritório (mesma área onde
configura foto de perfil, banner, nome e membros).

Ao transferir:
- O Escritório mantém o plano vigente até o fim do ciclo pago pelo owner
  original. Após esse período, o Escritório assume o plano do novo owner.
- Se o novo owner já possui um Escritório premium no limite do seu plano (ex:
  tem Escrivaninha e já possui 1 Escritório premium), o Escritório recebido
  permanece Free. Para torná-lo premium, o novo owner precisa fazer upgrade para
  um plano que permita mais Escritórios próprios.
- O owner original perde o vínculo com o Escritório após a transferência.

### Ownership — transferência por deleção de conta

O owner não pode deletar sua conta sem antes transferir a ownership de todos os
Escritórios em que é owner e que possuem outros membros. Se todos os seus
Escritórios são solos (só ele), pode deletar a conta diretamente — os Escritórios
são excluídos junto, com aviso de irreversibilidade.

### Escritório solo — owner não pode sair

Um owner não pode sair de um Escritório do qual é o único membro. A única opção
é deletar o Escritório. Para deletar, o Escritório não pode ser o único da conta
do usuário — é necessário criar (ou entrar em) outro Escritório antes. Um usuário
não pode ficar sem nenhum Escritório.

### Deleção de Escritório

A deleção é permanente e irreversível. Todos os processos, templates e dados do
Escritório são perdidos. O sistema exige confirmação explícita com aviso claro
antes de executar.

### Usuário Free entrando em Escritórios pagos

O usuário Free pode ser convidado para qualquer Escritório, independente do
plano. Ao aceitar, ele ocupa sua única vaga de entrada. Se quiser entrar em
outro Escritório, precisa sair do atual. Ele não pode sair do Escritório próprio
— portanto, seu limite prático é: 1 Escritório próprio (criado por padrão) + 1
Escritório externo.

### Programa de indicação

Cada usuário tem um código de referral único embutido no link que ele compartilha.
O link não tem validade — se alguém clicar daqui a anos e assinar, a conversão
ainda conta.

Uma conversão é validada após **30 dias do pagamento sem cancelamento ou
chargeback**. O mesmo CPF ou CNPJ de pagador não pode ser contabilizado mais de
uma vez nas conversões de um mesmo link — isso previne abuso com contas
alternativas.

A cada **3 conversões validadas**, o usuário ganha **1 mês grátis** do plano que
já usa. O benefício é cumulativo.

---

## 7. Como se conecta com o produto existente

**Accountancy (substituída)** — O conceito anterior de Accountancy era uma
entidade administrativa da Societiza, não algo que o usuário criava ou
controlava. O Escritório substitui esse conceito: agora é o próprio contador
que cria, configura e gerencia seu espaço de trabalho. Os campos de dados são
os mesmos — o que muda é a propriedade e o propósito da entidade.

**UserInvite (existe hoje, parcialmente)** — O sistema de convite por e-mail
evolui para suportar convite por link também (Escrivaninha e Executivo). O
convite passa a ser contextualizado: é um convite para um Escritório específico.

**Workflow e WorkflowTemplate (existem hoje)** — Continuam funcionando. O
Escritório ativo é o discriminante: ao trocar de Escritório, o usuário vê os
processos e templates daquele Escritório. Os limites de processos criados e de
templates ativos passam a ser aplicados por Escritório.

**Dashboard (futuro)** — Deve operar no contexto do Escritório ativo.
Informações de uso do plano (processos vs limite, membros vs limite) são
candidatas naturais para aparecer nessa camada.

**Automações (futuro)** — Exclusivas de Escritórios premium (Escrivaninha e
Executivo).

**Notificações (futuro)** — Configuradas por Escritório, não por usuário.

---

## 8. O que não é

- **Não é a feature Accountancy renomeada.** Accountancy era gestão interna da
  Societiza. Escritório é uma entidade do usuário final — ele cria, gerencia e
  paga por ela.
- **Não é compartilhamento de dados entre Escritórios.** Cada Escritório é
  completamente isolado, mesmo que o mesmo usuário seja owner de ambos.
- **Não é um módulo de billing completo.** Histórico de faturas, gestão de
  chargeback, notas fiscais e relatórios financeiros detalhados estão fora do
  escopo.
- **Não é um sistema de permissões granulares por função.** No MVP, todos os
  membros de um Escritório operam com permissões amplas dentro daquele contexto.
  Controles finos são uma feature futura.
- **Não é uma biblioteca de templates da Societiza.** Os templates que um
  usuário vê estão no Escritório ativo — não em um catálogo geral da plataforma.

---

## 9. Custos adicionais

O plano Escrivaninha e o plano Executivo exigem integração com um processador
de pagamento:

- O processador cobra os Escritórios mensalmente ou anualmente
- Webhooks do processador atualizam o status de plano em tempo real
- A escolha do processador afeta taxas, suporte a PIX e boleto, e complexidade
  de integração
- Processadores compatíveis com o mercado brasileiro: Stripe (com suporte a
  BRL), Pagar.me, Iugu — decisão a tomar antes da implementação

E-mails de convite já são cobertos pelo Brevo, que já está implementado.

---

## 10. Decisões tomadas nas perguntas abertas

| Ponto | Decisão |
|---|---|
| Preços | A definir — em branco até decisão comercial |
| Trial | Não haverá período de trial |
| Vantagens pessoais do plano | O plano não dá benefícios ao perfil pessoal do usuário. Ele desbloqueia capacidades nos Escritórios que o usuário criou e abrirá acesso a novas ferramentas e produtos à medida que forem lançados — exatamente como Notion e Slack fazem com seus tiers |
| Org switcher | Segue o padrão Notion/Slack: o usuário troca de Escritório por um seletor fixo na navegação. A decisão de UI (header, sidebar, posição exata) está em aberto |

## 11. Perguntas ainda abertas

- **Preços**: valores dos planos Escrivaninha e Executivo a definir
- **Org switcher — UI**: posição e comportamento exato do seletor de Escritório na navegação
- **Roles no MVP**: owner e membro são suficientes para o primeiro lançamento, ou já existe necessidade de um nível de acesso mais restrito para certos membros?