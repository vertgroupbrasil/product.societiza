---
status: approved
domain: accountancy
created_at: 2026-04-21
approved_at: 2026-04-21
---

# Accountancy — Unidade Organizacional

## 1. Visão da feature

A Accountancy é a representação digital de um escritório de contabilidade
dentro da Societiza. Cada contabilidade cadastrada na plataforma tem uma
identidade própria — nome, CNPJ, endereço, contato — e opera como um ambiente
completamente isolado dos demais. Nada do que acontece dentro de uma
contabilidade é visível para outra.

Do ponto de vista do contador que usa a plataforma, a Accountancy é "minha
empresa". É o contexto em que ele trabalha. Todos os processos que ele abre,
os templates que ele usa e a equipe que ele integra pertencem à sua
contabilidade. Ao entrar no sistema, ele vê a identidade da firma em "Minha
Contabilidade" — é o lugar onde ele sente que está dentro do próprio
escritório.

Do ponto de vista da Societiza, a Accountancy é a unidade de provisionamento
da plataforma. É ela quem define a fronteira do tenant. Criar uma
contabilidade é o ato que habilita uma firma a operar na plataforma — antes
disso, nada existe para aquela empresa.

## 2. Problema que resolve

Sem uma unidade organizacional formal, a plataforma não tem como separar os
dados de firmas diferentes. Processos, templates e usuários de contabilidades
distintas se misturariam — o que é inaceitável em um produto B2B
multi-tenant. A contabilidade cadastrada é o que ancora toda a separação
lógica entre clientes da Societiza.

Sem um cadastro estruturado da contabilidade, o onboarding também não tem
ponto de partida. O link de convite para o primeiro administrador de uma
firma precisa estar vinculado a uma contabilidade específica antes de ser
gerado. Sem a contabilidade cadastrada antes, não há como iniciar o ciclo
de uso — o provisionamento fica bloqueado na origem.

Há ainda um problema de identidade operacional: o contador precisa saber, ao
acessar a plataforma, em qual ambiente está trabalhando. Sem uma tela que
mostre com clareza os dados da sua firma, o sistema se torna abstrato — uma
coleção de telas sem contexto. A Accountancy resolve isso materializando a
firma como um objeto visível dentro do produto.

## 3. Para quem é

**Beneficiários diretos:**

- **Time interno da Societiza (SystemAdmin)** — é quem cria e gerencia o
  cadastro das contabilidades. Opera a plataforma como um back-office,
  provisionando novos clientes e mantendo os dados cadastrais atualizados.
  É o perfil que vive dentro das telas administrativas deste módulo.

- **Dono, sócio ou gestor da contabilidade (AccountancyAdmin)** — não cria o
  próprio cadastro, mas é o primeiro beneficiário depois que ele existe. Tem
  acesso à identidade da sua firma via "Minha Contabilidade" e é responsável
  por expandir o time por convite. É a pessoa para quem "Minha Contabilidade"
  representa a porta de entrada simbólica ao produto.

- **Colaborador da contabilidade (AccountancyEmployee)** — beneficiário
  operacional. Está dentro do escopo da contabilidade e consulta os dados
  dela, mas não gerencia nada.

**Beneficiários indiretos:**

- **Time de suporte e operação da Societiza** — usa o cadastro para
  identificar clientes em conversas e chamados.
- **Módulos futuros da plataforma** — societário, dashboard, notificações,
  portal do cliente — todos dependem da contabilidade como discriminante
  de dados. Sem ela, nenhum outro módulo tem como separar conteúdo por
  cliente da Societiza.

**Momento da jornada em que a feature entra:** o cadastro da contabilidade é
o primeiro ato. Vem antes de qualquer convite, antes de qualquer usuário,
antes de qualquer processo. "Minha Contabilidade" entra em cena no primeiro
login do AccountancyAdmin convidado, como a primeira confirmação visual de
que a firma dele existe dentro da Societiza.

## 4. Como melhora a experiência atual

**Antes (sem esta camada):**

Não existe estado "antes" dentro da Societiza — a Accountancy é a fundação
do modelo multi-tenant. Sem ela, nenhuma outra feature consegue existir com
dados separados por cliente. O contraste relevante é com o que aconteceria
se a plataforma tentasse operar sem uma unidade organizacional formal:
dados misturados, impossibilidade de provisionar um cliente novo sem
ambiguidade, e nenhum ponto de ancoragem visual para o contador.

**Depois (com esta camada):**

- **Provisionamento previsível.** A Societiza cria a contabilidade uma vez,
  gera o link de convite para o primeiro AccountancyAdmin, e a firma está
  habilitada a operar. O ato de criar a contabilidade é o gatilho de tudo o
  que vem depois.
- **Identidade visível para o contador.** Ao entrar na plataforma, o
  AccountancyAdmin encontra "Minha Contabilidade" como tela que mostra os
  dados da sua firma. O sistema deixa de ser abstrato e passa a dizer "você
  está aqui, dentro da sua empresa".
- **Fronteira multi-tenant garantida.** Cada processo, cada template, cada
  usuário criado dentro da plataforma pertence a uma contabilidade
  específica. Não há vazamento possível entre clientes — a separação é
  estrutural, não convencional.
- **Back-office operacional para a Societiza.** O SystemAdmin tem uma
  listagem clara das contabilidades cadastradas, com a possibilidade de
  consultar detalhes e atualizar dados quando necessário.

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**

- **Identity** — a contabilidade só ganha vida quando há pelo menos um
  AccountancyAdmin associado a ela. O vínculo entre usuário e contabilidade
  é feito via Identity, que carrega o contexto do tenant nas sessões
  autenticadas.
- **Links de Convite** — o fluxo de onboarding de uma contabilidade
  depende do link de convite que o SystemAdmin gera apontando para a
  `AccountancyId` recém-criada. É esse link que permite que o primeiro
  AccountancyAdmin entre na plataforma já vinculado à firma.

**O que esta feature habilita no futuro:**

- **Todos os módulos de operação** — societário, dashboard, notificações,
  portal do cliente. Nenhum deles existe sem uma unidade organizacional
  para ancorar seus dados. Esta feature é a fundação multi-tenant sobre
  a qual o resto do produto é construído.
- **Edição self-service dos dados cadastrais** — hoje o AccountancyAdmin
  não edita os dados da própria firma. Quando essa necessidade se tornar
  relevante (mudança de endereço, atualização de telefone, rebranding),
  a spec de evolução do perfil da contabilidade será aberta sobre esta
  base.
- **Busca e filtros na listagem de SystemAdmin** — à medida que a base
  de clientes cresce, a listagem simples paginada se torna insuficiente.
  A evolução para busca textual e filtros operacionais é habilitada por
  esta spec, mas fica fora do escopo desta entrega.
- **Múltiplas contabilidades por usuário** — um contador que opera mais
  de uma firma. Essa visão está documentada em `specs/accountancy-offices/`
  e se apoia na noção de Accountancy já consolidada aqui.

**Posição no produto:**

Esta é uma feature **fundacional do módulo Accountancy**. Não é
complementar, não é derivada. Ela define a unidade de tenant da plataforma
inteira — sem ela, nenhuma outra feature de operação faz sentido. A
qualidade desta spec estabelece a fronteira multi-tenant do produto.

## 6. O que não é (escopo negativo)

- **Não inclui** self-service de cadastro pelo AccountancyAdmin. A criação
  e a edição dos dados cadastrais da contabilidade são operações exclusivas
  do SystemAdmin neste estágio. A evolução para edição self-service é
  horizonte futuro, não escopo atual.
- **Não inclui** ciclo de vida da contabilidade. Não existe ativar,
  inativar, suspender ou arquivar uma contabilidade. A contabilidade,
  uma vez criada, permanece ativa indefinidamente.
- **Não inclui** exclusão de contabilidade. Não há mecanismo de delete.
  Se a Societiza precisar encerrar o relacionamento com um cliente, a
  operação é tratada fora da plataforma neste estágio.
- **Não inclui** busca textual ou filtros na listagem de SystemAdmin.
  A listagem é paginada e ordenada por nome legal. Busca operacional é
  evolução futura — não escopo atual.
- **Não inclui** billing, planos ou cobrança. O MVP é freemium. Não há
  tier de plano exposto no perfil da contabilidade, nem status comercial,
  nem limites de uso vinculados a plano de assinatura.
- **Não inclui** gestão de membros da contabilidade. Listar, remover ou
  alterar papéis de usuários vinculados à contabilidade pertence ao
  módulo Identity — não a este.
- **Não inclui** dashboard ou métricas sobre a contabilidade. Nenhum dado
  analítico é exposto nesta feature.
- **Não inclui** suporte a múltiplas contabilidades por usuário. Um
  usuário pertence a exatamente uma contabilidade. A visão multi-office
  tem spec própria (`specs/accountancy-offices/spec.md`) e não compromete
  este escopo.
- **Não inclui** identidade visual customizável (logo, cores, tema). O
  perfil da contabilidade é apenas dados cadastrais neste estágio.
  Branding por contabilidade é evolução futura, ligada à experiência do
  portal do cliente.

## 7. Custos adicionais

**Nenhum custo externo identificado.**

A feature opera inteiramente dentro da infraestrutura da própria Societiza.
O cadastro, a listagem, a consulta e a exibição de "Minha Contabilidade"
não dependem de nenhum provedor externo, gateway de pagamento, serviço de
validação de CNPJ em tempo real, ou integração com APIs governamentais.
A validação de CNPJ é estrutural (algoritmo local), não externa.

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Quem cria a contabilidade | Exclusivamente SystemAdmin. AccountancyAdmin não cria nem edita os dados cadastrais da própria firma neste estágio. |
| Quem edita a contabilidade | Exclusivamente SystemAdmin. Edição self-service é horizonte futuro, não escopo atual. |
| Tela "Minha Contabilidade" | Disponível para AccountancyAdmin e AccountancyEmployee em modo somente leitura. É o ponto de ancoragem da identidade organizacional no produto. |
| Ciclo de vida | Não existe. Contabilidade criada permanece ativa indefinidamente. Sem status de ativo, inativo, suspenso ou arquivado. |
| Exclusão | Não existe. Nenhuma contabilidade é apagada da plataforma. |
| Busca e filtros na listagem | Fora do escopo atual. Listagem paginada e ordenada por nome legal. Busca é evolução futura. |
| Dados obrigatórios | CNPJ, nome legal, endereço, cidade, estado, CEP e telefone são obrigatórios. Nome fantasia e e-mail são opcionais. |
| Unicidade | CNPJ é único no sistema. Tentar criar uma segunda contabilidade com o mesmo CNPJ é bloqueado. |
| Fonte de verdade | Backend. Toda validação de dados cadastrais e de autorização por perfil é feita no servidor. |
| Formatação de dados | Dados cadastrais como CNPJ, CEP e telefone são armazenados sem formatação (apenas dígitos). A apresentação formatada para o usuário é responsabilidade da camada de exibição. |
| Status da implementação | Backend já implementado em `api.societiza`. Esta spec documenta o que foi construído para servir de base ao `logic.md` e ao `integration.md` subsequentes. |

## 9. Perguntas abertas

Nenhuma. Todas as decisões de produto estão fechadas. O comportamento das
telas de SystemAdmin e da experiência "Minha Contabilidade" foi definido
consistentemente com o que já está registrado em `accountancy-context.md`
e com o padrão estabelecido pelas specs de Identity aprovadas. As decisões
remanescentes — contratos de endpoint, códigos de erro, estados de carga
por tela, pontos de integração entre frontend e backend — são internas à
camada de lógica e pertencem ao `logic.md`, não à spec de produto.