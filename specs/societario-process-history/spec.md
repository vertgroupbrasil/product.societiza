---
status: approved
domain: societario
created_at: 2026-05-01
approved_at: 2026-05-03
---

# Histórico de Processos Concluídos

## 1. Visão da feature

O board operacional do societário existe para mostrar o que está
acontecendo agora. O histórico existe para mostrar o que aconteceu. São
duas perguntas diferentes, e elas merecem duas vistas diferentes.

Esta feature separa formalmente o histórico de processos concluídos do
board ativo. O board passa a mostrar apenas processos InProgress,
organizados pelas etapas do template em uso. O histórico passa a ser uma
vista própria — uma lista de processos concluídos que o contador acessa
quando quer reconstruir a relação com um cliente, conferir o que foi feito
em uma operação passada ou filtrar por período. No histórico, o contador
filtra por template (incluindo templates já arquivados), por versão do
template, por tipo de processo (abertura, alteração, baixa) e por
intervalo de datas.

A decisão de produto que sustenta tudo isso é simples mas crítica:
**processos concluídos guardam dentro de si tudo que precisam para serem
renderizados, para sempre.** Cada processo concluído carrega seu próprio
snapshot completo da estrutura do template (steps, tasks, fields), todos
os valores que foram preenchidos, o tipo, o nome e a versão do template
no momento da conclusão. Isso significa que o histórico continua
funcionando mesmo se o template original for arquivado ou substituído por
versões mais novas. A memória operacional da contabilidade fica
preservada de forma autônoma.

---

## 2. Problema que resolve

A spec aprovada do board operacional (`societario-workflow-process`)
deixou explícito que processos concluídos não podem ser apagados —
eles são a única memória estável da relação com o cliente-alvo, que é
texto livre e não tem entidade própria. Mas ela também deixou em aberto
uma pergunta: **como evitar que a coluna "concluído" do board polua a
visão operacional do contador conforme os meses passam?**

Essa pergunta não é teórica. Em uma contabilidade ativa, a quantidade de
processos concluídos cresce de forma cumulativa, enquanto a quantidade
de processos em andamento se mantém aproximadamente estável. Em poucos
meses, o board deixa de ser uma ferramenta operacional e vira uma lista
em que o que importa (o que está rodando agora) some no meio do que já
foi feito.

Há também um problema correlato que aparece quando o versionamento de
templates entra em jogo (ver spec
`societario-template-versioning`). Templates são arquivados ao longo do
tempo. Se o histórico de um processo concluído depende do template
ainda existir como ele era no momento da conclusão, qualquer operação
de manutenção em templates corrompe o histórico. Renomear, arquivar ou
evoluir um template não pode ter efeito retroativo sobre o histórico.

O workaround atual seria filtrar visualmente o board ou esperar uma
solução genérica de paginação. Nenhum dos dois resolve o problema: o
contador continua precisando consultar processos passados, e o board
continua misturando dois tipos de uso muito diferentes — operação ativa
e consulta de histórico.

---

## 3. Para quem é

**Beneficiário direto:** o contador (AccountingAdmin ou
AccountingEmployee) que precisa consultar processos passados — para
responder a um cliente sobre o que foi feito, conferir uma data de
conclusão, recuperar um valor preenchido em um campo, ou simplesmente
auditar a operação histórica da contabilidade.

**Beneficiário indireto:** a contabilidade como organização, que ganha
uma memória operacional duradoura e independente da evolução dos
templates. O contexto societário do cliente fica preservado mesmo que
o template usado para conduzir aquele processo deixe de existir.

O contexto de uso é distinto do uso do board. O board é consultado várias
vezes por dia, durante a operação. O histórico é consultado de forma
pontual: quando uma pergunta específica precisa ser respondida ou quando
uma operação precisa ser reconstruída. A interface deve refletir essa
diferença — o histórico não precisa ser tão denso operacionalmente
quanto o board, mas precisa ser fácil de filtrar.

---

## 4. Como melhora a experiência atual

**Antes:**

O contador opera no board e os processos concluídos ficam ali, misturados
com os InProgress. Conforme o tempo passa, a coluna "concluído" cresce
indefinidamente. Quando o contador precisa achar um processo antigo, ele
rola o board ou usa filtros visuais que não foram pensados para consulta
histórica. A relação entre o processo concluído e o template que o
originou fica frágil — se o template for alterado ou arquivado, a leitura
do processo antigo pode ficar inconsistente ou simplesmente parar de
fazer sentido.

**Depois:**

O contador abre o board e vê só o que está rodando. O board é mais leve,
mais focado, mais útil para a operação do dia. Quando precisa consultar o
passado, ele entra em uma vista separada de histórico. Lá, escolhe um
recorte — por exemplo, "todos os processos de abertura concluídos no
trimestre passado", ou "todos os processos do template X versão 1.0", ou
"tudo que foi concluído em janeiro". Os filtros incluem templates já
arquivados, porque o histórico precisa enxergar o passado mesmo quando o
presente já evoluiu.

Ao abrir um processo concluído, o contador vê exatamente como ele foi —
com a estrutura de etapas e tarefas que valia naquele momento, com os
valores que foram preenchidos, com o nome e a versão do template como
estavam quando o processo terminou. Mesmo que o template tenha sido
arquivado ou substituído, a leitura continua íntegra.

---

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**

- **WorkflowProcess** — esta feature consome o estado de processos
  concluídos que a camada de execução já produz. Os campos do processo
  (estrutura materializada, valores preenchidos, tipo, timestamps) já
  estão definidos pela spec aprovada do board operacional. Esta feature
  formaliza como esses dados são apresentados como histórico e adiciona
  os atributos necessários para que o processo concluído seja
  auto-suficiente.
- **WorkflowTemplate** — o histórico filtra por template e por versão.
  Filtrar por template inclui templates arquivados (a vista de filtro
  precisa enxergar o passado, não apenas o presente operacional). A
  filtragem por versão depende do mecanismo de versionamento descrito na
  spec `societario-template-versioning`.
- **Accountancy** — todo processo pertence a uma contabilidade, e o
  histórico exibe apenas processos da contabilidade ativa.

**O que esta feature habilita no futuro:**

- **Dashboard operacional (P2)** — métricas históricas (tempo médio de
  conclusão, volume por período, distribuição por tipo) precisam de uma
  base de processos concluídos consultável. Esta feature é a base.
- **Exportação de relatórios (P4)** — exportar histórico exige que o
  histórico exista como vista filtrável em primeiro lugar.
- **Portal do cliente (futuro)** — quando o cliente final ganhar acesso
  ao próprio histórico, o snapshot auto-suficiente do processo
  concluído é o que permite mostrar a ele exatamente como o processo
  rodou, mesmo após qualquer evolução de template.
- **Outras features que reagem à conclusão de processo** — a publicação
  da conclusão de processo como evento integrável habilita, em paralelo,
  o arquivamento automático de templates Superseded (ver spec
  `societario-template-versioning`) e qualquer feature futura que
  precise reagir ao fim de um processo (notificações de conclusão para
  o cliente, métricas em tempo real do dashboard, ações automáticas
  pós-conclusão).

**Posição no produto:**

Feature core de fortalecimento do societário (P1). Fecha uma pergunta
aberta da spec do board operacional (poluição do kanban com concluídos),
e cria a base de dados duradoura sobre a qual o dashboard (P2) e
funcionalidades de exportação e relatório futuras vão operar. Não é uma
adição ao produto — é a separação correta de duas vistas que hoje estão
indevidamente colapsadas em uma só.

---

## 6. O que não é (escopo negativo)

- **Não inclui** edição de processos concluídos. O histórico é
  somente-leitura. Uma vez concluído, o processo é imutável.
- **Não inclui** exclusão de processos concluídos. Já decidido na spec
  do board operacional: processos concluídos permanecem armazenados
  permanentemente. Esta feature reforça essa decisão e a torna mais
  visível ao colocar o histórico em vista própria.
- **Não inclui** desconclusão (reabrir um processo concluído). Se um
  processo foi concluído por engano, o tratamento operacional é criar
  um novo processo. Reabrir é um vetor de inconsistência que o produto
  rejeita.
- **Não inclui** dashboard de métricas agregadas no histórico. Contagens,
  médias, gráficos e cruzamentos por período pertencem ao dashboard
  (P2). O histórico desta feature é uma lista filtrável, não um painel
  analítico.
- **Não inclui** exportação do histórico para CSV, PDF ou outros
  formatos. Exportação é capacidade futura (P4).
- **Não inclui** comparação entre processos. O contador abre um processo
  por vez. Comparar dois processos lado a lado fica fora.
- **Não inclui** reconciliação retroativa de processos concluídos com
  versões mais novas do template. Cada processo concluído carrega seu
  próprio snapshot e essa é a verdade dele para sempre. Não há
  "atualizar processos antigos para a nova estrutura".
- **Não inclui** acesso do cliente final ao histórico. O cliente não vê
  nada disso. Portal do cliente é capacidade futura.
- **Não inclui** filtros por usuário responsável ou por autor do
  processo. Como decidido na spec do board, no MVP não há
  responsabilidade pessoal nem autoria registrada — então não há por
  onde filtrar.
- **Não inclui** filtro por valor preenchido em campo específico. O
  histórico filtra por metadados do processo (template, versão, tipo,
  período), não pelo conteúdo dos campos. Busca por conteúdo é problema
  diferente, fora desta versão.

---

## 7. Custos adicionais

Nenhum custo externo identificado.

A feature opera inteiramente dentro do ecossistema já existente da
Societiza. O snapshot auto-suficiente do processo concluído é dado já
materializado pela camada de execução do WorkflowProcess — esta feature
formaliza que esse snapshot deve ser persistido de forma a sobreviver à
evolução ou arquivamento do template original, mas não introduz nenhum
serviço externo, integração ou infraestrutura adicional. A publicação
da conclusão de processo como evento integrável usa a infraestrutura
de mensageria interna já prevista para o MVP.

---

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Separação entre board e histórico | Board mostra apenas processos InProgress. Histórico é uma vista própria, separada, dedicada a processos concluídos. |
| Auto-suficiência do processo concluído | Cada processo concluído carrega snapshot completo: estrutura materializada (steps, tasks, fields), todos os valores preenchidos, referência ao template (id e versão), nome do template como estava no momento da conclusão, tipo do processo, data de conclusão. Isso permite renderizar o processo concluído mesmo que o template original tenha sido arquivado ou superseded. |
| Imutabilidade do snapshot | O snapshot persistido no momento da conclusão não é alterado por evoluções posteriores do template (renomeação, fork, archive). Cada conclusão é uma fotografia definitiva. |
| Persistência permanente | Processos concluídos não são deletados nem arquivados destrutivamente. Continuam acessíveis pelo histórico para sempre. (Decisão herdada da spec do board operacional, reforçada aqui.) |
| Filtros do histórico | Por template (incluindo templates Archived), por versão do template, por tipo de processo (abertura, alteração, baixa), por intervalo de datas (data de conclusão). Os filtros podem combinar entre si. |
| Inclusão de templates arquivados nos filtros | Sim. A vista de filtro do histórico mostra todos os templates que já foram usados para criar processos concluídos, independentemente de estarem hoje Active, Superseded ou Archived. |
| Visualização de um processo concluído | Somente-leitura. Mostra a estrutura como estava no momento da conclusão, com os valores preenchidos. Sem opção de editar, reabrir ou alterar. |
| Conclusão de processo como evento | Quando um processo é concluído, isso gera um evento publicado no canal de mensageria interna. Outras features (arquivamento automático de templates Superseded, futuras notificações, futuro dashboard em tempo real) consomem esse evento sem acoplar ao módulo de processo. |
| Escopo da feature dentro do MVP | Faz parte do MVP. A separação entre board e histórico é decisão de produto que entra junto com o board, não depois. |

---

## 9. Perguntas abertas

- **Tratamento de filtros vazios.** Quando o contador entra na vista de
  histórico sem aplicar filtros, ela mostra todos os processos
  concluídos da contabilidade, ordenados por data de conclusão
  decrescente? Ou exige que pelo menos um filtro seja aplicado para
  evitar listas enormes? Decisão de UX a fechar com o desenho da tela.

- **Granularidade do filtro de período.** O filtro por intervalo de
  datas trabalha em qualquer granularidade (data inicial e final
  livres) ou oferece atalhos pré-definidos (este mês, mês passado,
  este trimestre, este ano)? Os dois caminhos são possíveis e a
  resposta depende de quão estruturada é a consulta típica dos
  contadores no dia a dia.

- **Acesso ao histórico por perfil.** No MVP, AccountingAdmin e
  AccountingEmployee operam o board com mesmo escopo. O histórico segue
  a mesma regra (qualquer membro da contabilidade vê o histórico
  inteiro) ou há razão para restringir alguma vista a apenas
  AccountingAdmin? A premissa atual é que segue a mesma regra do board,
  mas vale confirmar antes da implementação.

- **Visibilidade do histórico para SystemAdmin.** O SystemAdmin tem
  acesso global e pode precisar consultar processos concluídos de
  qualquer contabilidade para entender uso e evolução do produto. Isso
  é parte desta feature ou é uma capacidade administrativa separada,
  pertencente a outro módulo? Premissa atual: fora desta spec.
