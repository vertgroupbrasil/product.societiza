# Societiza — Contexto de Negócio

> **Última atualização:** 2026-04-06 por Gabriel Carvalho e Claude

## Como usar este documento

Este documento serve como base de contexto de produto para qualquer pessoa ou
IA que trabalhe na Societiza. Ele se destina igualmente a quem pensa produto, 
quem implementa frontend ou backend, quem planeja automações, quem discute 
evolução de features e quem integra no frontend uma feature já feita no backend.

O objetivo não é ensinar como implementar linha por linha. É explicar com
profundidade o que a Societiza é, que problema resolve, como as features se
conectam, o que já existe, o que é visão futura, e quais obstáculos estruturais
precisam ser considerados em qualquer planejamento. Ele é um índice que aponta 
para documentos detalhados dentro desta pasta (`docs/`).

**Regras de uso:**

- Não trate como existente uma feature que ainda é futura.
- Pode detalhar como uma feature futura deveria funcionar, desde que fique claro
  que é visão futura e não comportamento já entregue.
- Quando houver conflito entre narrativa de produto e estado técnico atual, explicite
  o gap em vez de fingir que ele não existe.
- Ao planejar algo novo, preserve a coerência com o núcleo do produto.

---

## 1. O que é a Societiza

A Societiza nasceu para mudar a maneira como o Brasil abre empresas. Hoje, a 
operação societária das contabilidades é extremamente fragmentada. Os dados de
um mesmo processo ficam despejados em planilhas, sites de prefeitura, portais de 
viabilidade, documentos espalhados, conversas no WhatsApp e em outros softwares 
que não compartilham contexto entre si. Isso gera retrabalho, perda de informação, 
falta de previsibilidade, dependência excessiva de memória humana e baixa capacidade 
de escalar a operação com qualidade.

A Societiza existe para centralizar essa operação e reduzir a burocracia
percebida pelo contador. O objetivo não é apenas organizar tarefas.
O objetivo é transformar a abertura de empresas em uma operação estruturada, 
clara, configurável, colaborativa e progressivamente automatizável.

A plataforma deve concentrar toda a regra de negócio da abertura de empresas:
desde a proposta ao cliente, coleta dos dados iniciais, execução operacional, 
acompanhamento do andamento, notificações, documentos, pendências e conclusão 
da abertura.

A visão é se tornar o sistema operacional da abertura de empresas no Brasil 
para contabilidades. A missão é descomplicar a vida do contador.

> Detalhes completos: [`product/product-vision.md`](product/product-vision.md)

---

## 2. ICP e proposta de valor

O ICP são contabilidades de pequeno e médio porte e contadores autônomos. A
Societiza é um SaaS B2B pensado como software de time — um usuário principal
cria a conta e convida a equipe.

A proposta de valor central: dar ao contador visibilidade e controle sobre toda
a operação societária em um único lugar, substituindo planilhas e comunicações
fragmentadas por um fluxo estruturado, rastreável e configurável.

> Detalhes completos: [`product/product-vision.md`](product/product-vision.md)

---

## 3. Times, usuários e permissões

A plataforma é estruturada para times — cada escritório de contabilidade opera
com sua própria equipe, seus próprios dados e seu próprio ambiente de trabalho.
O acesso é inteiramente controlado por convite, com perfis de permissão
distintos definidos no momento do convite.

O multi-tenancy é organizado por escritório de contabilidade: cada contabilidade
tem sua própria instância de dados, processos, templates e membros de equipe.
O aggregate `Accountancy` é o discriminante central dessa separação.

A plataforma opera com três perfis de usuário distintos, com escopos de acesso
bem delimitados:

- **SystemAdmin** é o perfil dos donos e operadores da própria Societiza. Tem
  acesso global à plataforma: cadastra escritórios de contabilidade, gerencia
  workflow templates (criação, edição e configuração), e convida usuários de
  qualquer perfil para qualquer tenant. Não pertence a nenhum escritório
  específico — opera acima do tenant.

- **AccountingAdmin** é o administrador de um escritório de contabilidade. Entra
  na plataforma exclusivamente por convite do SystemAdmin. Gerencia os membros da
  sua equipe (convidar, editar e remover) e tem acesso completo à operação de
  processos societários.

- **AccountingEmployee** é o funcionário operacional da contabilidade. Entra por
  convite do AccountingAdmin ou do SystemAdmin. Seu escopo é restrito à operação:
  cria processos, edita, acompanha o andamento e executa tarefas. Não gerencia
  usuários nem workflow templates.

O fluxo de acesso é inteiramente controlado por convite — não existe registro
direto. Todo usuário entra via `UserInvite`, com o perfil definido no momento
do convite.

No roadmap, está prevista a introdução de novos perfis de usuário dentro do
contexto da contabilidade, com escopos de acesso mais específicos. Quais perfis
existirão e o que cada um poderá fazer ainda não está definido nem implementado.

**Freemium (planejado para o MVP):** o time está desenhando um fluxo de entrada
diferente do descrito acima, contemplando um plano gratuito com acesso
self-service. Os detalhes ainda estão sendo definidos e serão documentados
separadamente quando consolidados.

---

## 4. Features existentes

### Societário (workflow-template + workflow-process)

O societário é o núcleo do produto. Um board kanban com colunas ordenadas que
representam etapas de abertura de empresa. Cada card é um workflow process. O workflow
template é a camada de configuração — etapas, tarefas (Manual, Automatic e Approval) e 
campos (Text, Boolean, Integer, Select e Date) configuráveis.

> Detalhes completos: [`context/societario-context.md`](context/societario-context.md)

### Accountancy

Representa o escritório de contabilidade na plataforma. É o discriminante
multi-tenant: convites, processos, templates e todas as demais funcionalidades 
partem do contexto da contabilidade.

> Detalhes completos: [`context/accountancy-context.md`](context/accountancy-context.md)

### Autenticação

Login com e-mail e senha, logout, refresh de token e reset de senha implementados.
Endpoints ainda sem proteção aplicada. Fluxo de acesso em reestruturação por
conta do plano freemium planejado para o MVP.

> Detalhes completos: [`context/authentication-context.md`](context/authentication-context.md)

---

## 5. Próximas prioridades

O roadmap segue uma lógica de fora para dentro: primeiro consolidar o núcleo
operacional, depois ampliar leitura e acompanhamento, depois reduzir fricção de
entrada, e por fim expandir o produto para além da abertura de empresa.

| Prioridade | Feature | Status |
|---|---|---|
| P1 | Fortalecer o societário | Em andamento |
| P2 | Dashboard operacional | Não iniciado |
| P3 | Notificações e acompanhamento | Futuro |
| P4 | Automação, importação e IA | Futuro |
| P5 | Álvarás, certificados e vida regulatória | Futuro |

**P1** cobre melhorias de qualidade, consistência e capacidade de configuração
do societário e do workflow template — base para tudo que vem depois.

**P2** é a próxima feature grande e mandatória: um dashboard de leitura
operacional interativo, conectado ao societário. Cada insight deve permitir
navegação direta para a área operacional correspondente.

**P3** amplia a capacidade de acompanhamento: notificações por e-mail para a
equipe e para o cliente final, alertas de processos parados e gargalos
operacionais.

**P4** reduz a fricção de entrada e aumenta automação: importação de processos,
exportação, automação de tarefas repetitivas e MCP Server para IA assistiva.

**P5** expande o produto da abertura de empresa para a continuidade regulatória
do cliente: controle de álvarás, certificados digitais em nuvem e portal do
cliente.

> Semântica completa de cada prioridade: [`product/product-priorities.md`](product/product-priorities.md)
> Dashboard: [`planning/dashboard.md`](planning/dashboard.md)
> Features futuras detalhadas: [`planning/future-features.md`](planning/future-features.md)

---

## 6. Gaps estruturais

Cada gap está classificado por severidade:

- **Bloqueante MVP** — precisa ser resolvido antes do lançamento
- **Risco** — não bloqueia o MVP, mas pode causar problemas sérios se ignorado
- **Dívida aceitável** — reconhecido, mas tolerável no curto prazo

### Migração de dados para dentro do sistema `Dívida aceitável`

Este é um dos maiores desafios de produto. Os dados atuais dos usuários estão
espalhados em planilhas, WhatsApp, e-mails, cadernos operacionais, ERPs e sites
públicos. Trazer isso para dentro da plataforma de forma confiável é uma dor
grande, e qualquer funcionalidade de importação precisa reconhecer que a origem
será, em muitos casos, despadronizada e sem semântica limpa. Não existe hoje
nenhuma funcionalidade de importação disponível. Isso é um bloqueio real para
contabilidades que querem adotar o produto mas têm processos em andamento em
outros sistemas.

### Variação de regra de negócio por cidade, estado e prefeitura `Dívida aceitável`

O Brasil não tem um fluxo único de abertura de empresa. Cada estado, cidade e,
em muitos casos, cada prefeitura relevante pode impor regras, sequências,
exigências, portais, nomenclaturas e limitações diferentes. Isso significa que
qualquer automação, template pronto ou fluxo considerado "padrão" precisa ser
tratado com cuidado. O produto precisa ser suficientemente configurável para
acomodar essas diferenças, sem assumir que existe um processo nacional
totalmente uniforme. Hoje o sistema de templates é flexível, mas não tem
suporte explícito a localidade ou perfil regional.

### Dependência de sistemas externos instáveis `Dívida aceitável`

Boa parte da operação societária depende de sites públicos e sistemas externos
que podem ser lentos, confusos, instáveis, ter captcha, mudar layout sem aviso
ou ficar fora do ar. Isso impacta diretamente futuras automações e deve ser
considerado nas decisões de produto, engenharia e operação. Qualquer feature
de automação precisa ser projetada assumindo que o sistema externo pode falhar,
e o usuário precisa de visibilidade sobre o status dessas automações.

### Endpoints desprotegidos e contratos técnicos desalinhados `Bloqueante MVP`

Algumas histórias do produto já estão claras, mas o backend atual ainda não
implementa ou não expõe exatamente esses contratos da forma esperada. O caso
da autenticação é o exemplo mais importante. Isso significa que planejamento
estratégico e planejamento técnico precisam sempre explicitar o que já existe
de verdade, o que é comportamento esperado de produto, e o que ainda precisa
ser alinhado entre frontend, backend e infra.

### Fluxo de entrada do usuário (freemium vs. convite) `Bloqueante MVP`

O plano freemium planejado para o MVP muda fundamentalmente o fluxo de entrada
na plataforma. Hoje o acesso é inteiramente por convite, mas o modelo
self-service exige um fluxo de registro direto, onboarding, limites de plano e
lógica de upgrade. Sem essa definição fechada, não é possível lançar o MVP com
entrada aberta. O impacto atinge autenticação, permissões e a experiência
inicial do usuário.

### Versionamento de templates `Risco`

Hoje não existe mecanismo de versionamento de workflow templates. Se um
template é alterado enquanto processos estão em andamento, não há garantia de
que os processos existentes mantenham consistência com a versão do template em
que foram criados. Isso pode causar problemas silenciosos de integridade
conforme o uso escalar.

### Permissões ainda pouco granulares `Dívida aceitável`

Hoje o modelo de permissão cobre três perfis distintos, mas dentro do contexto
da contabilidade ainda não existe granularidade suficiente. Isso pode virar
gargalo conforme o produto evoluir para operações com mais pessoas e mais
sensibilidade de ação — por exemplo, quando um contador júnior não deve ter
poder de alterar templates ou arquivar processos.

### Colaboração com o cliente `Dívida aceitável`

Hoje o cliente (empresário) não tem acesso ao sistema. Toda comunicação é fora
da plataforma — por WhatsApp, e-mail ou telefone. O contador precisa sair do
sistema para falar com o cliente, pedir documentos ou atualizar sobre o
andamento. Isso é um gap de produto importante que só será resolvido com o
portal do cliente.

### Dashboard sem definição final de métricas `Bloqueante MVP`

Sabemos que o dashboard é obrigatório e estratégico, mas ainda não existe
definição fechada de métrica, fonte de verdade, visualização final ou
interatividade completa. O planejamento dessa feature precisa combinar visão de
produto com a realidade operacional do societário.

---

## 7. Mapa de features e integrações

O mapa completo de features (existentes e futuras), suas dependências, dados
compartilhados e pontos de acoplamento críticos está em:

> [`planning/feature-map.md`](planning/feature-map.md)

---

## 8. Documentação de integração

O template canônico para documentar features do backend para consumo do
frontend está em:

> [`integration/TEMPLATE.md`](integration/TEMPLATE.md)

---

## Documentos satélites

| Documento | Conteúdo |
|---|---|
| [`product/product-vision.md`](product/product-vision.md) | Identidade, ICP, valores, como features devem se comportar |
| [`product/product-priorities.md`](product/product-priorities.md) | Semântica de prioridade P1-P5 |
| [`context/societario-context.md`](context/societario-context.md) | WorkflowTemplate, WorkflowProcess, board, campos |
| [`context/accountancy-context.md`](context/accountancy-context.md) | Accountancy como tenant root |
| [`context/authentication-context.md`](context/authentication-context.md) | Autenticação: estado atual e gaps |
| [`planning/dashboard.md`](planning/dashboard.md) | Dashboard: visão, dados, dependências |
| [`planning/future-features.md`](planning/future-features.md) | Todas as features futuras planejadas |
| [`planning/feature-map.md`](planning/feature-map.md) | Status, dependências, dados compartilhados |
| [`integration/TEMPLATE.md`](integration/TEMPLATE.md) | Template de documentação backend → frontend |