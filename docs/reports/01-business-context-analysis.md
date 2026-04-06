# Relatório 1 — Análise Estrutural do business-context.md

**Data:** 2026-04-06
**Escopo:** Avaliação do documento `docs/business-context.md` como índice central de contexto de produto.

---

## 1. O que o documento é

O `business-context.md` funciona como **índice narrativo** da Societiza. Ele não é uma spec técnica, não é um backlog e não é documentação de API. Ele é o ponto de entrada para qualquer pessoa ou agente de IA que precise entender o produto antes de tomar decisões.

O documento cumpre três papéis simultâneos:

1. **Onboarding de contexto** — explica o que é a Societiza, para quem serve e qual problema resolve.
2. **Mapa de navegação** — aponta para documentos satélites que detalham cada área (visão, contexto de features, planejamento, integração).
3. **Registro de gaps** — lista explicitamente os problemas estruturais que qualquer planejamento deve considerar.

---

## 2. Como está estruturado

| Seção | Conteúdo | Papel |
|---|---|---|
| §1 O que é a Societiza | Visão, missão, proposta geral | Contexto fundacional |
| §2 ICP e proposta de valor | Público-alvo e modelo B2B | Enquadramento comercial |
| §3 Times, usuários e permissões | Perfis (SystemAdmin, AccountingAdmin, AccountingEmployee), convites, multi-tenancy | Modelo de acesso |
| §4 Features existentes | Societário, Accountancy, Autenticação | Estado atual do produto |
| §5 Próximas prioridades | Tabela P1-P5 com status | Roadmap resumido |
| §6 Gaps estruturais | 7 gaps explícitos | Riscos e dívidas |
| §7 Mapa de features | Ponteiro para `feature-map.md` | Navegação |
| §8 Documentação de integração | Ponteiro para `TEMPLATE.md` | Navegação |
| Documentos satélites | Tabela com 8 links | Índice de navegação |

---

## 3. Pontos fortes

- **Honestidade sobre gaps.** O documento não esconde problemas — lista explicitamente endpoints desprotegidos, ausência de importação, variação regulatória, permissões pouco granulares e falta de colaboração com o cliente. Isso é raro e valioso.
- **Distinção clara entre existente e futuro.** As regras de uso no topo reforçam que features futuras não devem ser tratadas como entregues.
- **Estrutura de índice funcional.** Um leitor consegue, a partir deste documento, navegar para qualquer área do produto com contexto suficiente.
- **Seção de permissões completa.** A descrição dos três perfis é precisa, com escopos bem delimitados e fluxo de convite documentado.

---

## 4. Pendências e problemas identificados

### 4.1 Ausência de versionamento ou data

O documento não tem data de última atualização nem indicação de versão. Como ele é o ponto de entrada canônico, qualquer leitor deveria saber se está lendo algo de uma semana ou de seis meses atrás. **Sugestão:** adicionar campo de data e changelog resumido no topo.

### 4.2 Seção de features existentes incompleta

A §4 lista três features (societário, accountancy, autenticação), mas não menciona **user management** como feature separada. O convite de usuários, a gestão de membros por AccountingAdmin e o fluxo de aceite são funcionalidades distintas da autenticação pura. Hoje isso está parcialmente coberto em `authentication-context.md`, mas o business-context deveria reconhecer essa separação.

### 4.3 Freemium mencionado sem documento de referência

A §3 menciona que o freemium está sendo desenhado e "será documentado separadamente quando consolidado". Não há ponteiro para nenhum documento de rascunho, nem prazo esperado. Se houver qualquer discussão em andamento, deveria haver ao menos um link para um documento de status, mesmo que vazio.

### 4.4 Gaps sem priorização

A §6 lista 7 gaps, mas todos têm o mesmo peso visual. Não há indicação de quais são bloqueantes para o MVP e quais são dívidas aceitáveis a curto prazo. **Sugestão:** classificar cada gap como `bloqueante para MVP`, `dívida aceitável` ou `pós-MVP`.

### 4.5 Ausência de link para specs e contracts

O documento não menciona as pastas `features/specs/` e `contracts/api/`. Ambas estão vazias hoje, mas fazem parte do workflow definido no CLAUDE.md. Quando specs e contracts começarem a ser gerados, o business-context deveria apontar para eles como artefatos de saída do processo de planejamento.

### 4.6 Dashboard descrito como "próxima feature grande" sem qualificação

A §5 e a §6 mencionam o dashboard como próxima prioridade, mas a informação está espalhada. Na tabela de prioridades aparece como "Não iniciado"; na §6 aparece como "sem definição final de métricas". O leitor precisa cruzar três fontes (business-context, product-priorities, dashboard.md) para entender o real estado. **Sugestão:** consolidar o status do dashboard em uma frase única na §5.

---

## 5. Como o documento deve ser utilizado

### Para produto/planejamento
Ler primeiro business-context.md inteiro. Depois navegar para o documento satélite relevante. Antes de propor qualquer feature, verificar a §6 (gaps) e a §5 (prioridades) para não criar conflito com o roadmap.

### Para implementação frontend
Ler business-context.md para entender o papel de cada feature no produto. Depois ir para `docs/integration/` para o contrato técnico. Nunca integrar baseado apenas no business-context — ele é contexto, não contrato.

### Para implementação backend
Usar como referência de regra de negócio e semântica de domínio. As descrições de perfis de usuário, multi-tenancy e workflow template são autoritativas para decisões de domínio.

### Para IA/agentes
É o primeiro documento a ser lido em qualquer sessão. As regras de uso no topo são obrigatórias — especialmente a distinção entre existente e futuro. Qualquer output de IA que viole essa distinção está desalinhado.

---

## 6. Próximos passos recomendados

| Ação | Prioridade | Esforço |
|---|---|---|
| Adicionar data de última atualização e changelog | Alta | Baixo |
| Separar user management de autenticação na §4 | Média | Baixo |
| Classificar gaps da §6 por severidade (bloqueante MVP / dívida / pós-MVP) | Alta | Médio |
| Criar placeholder doc para freemium e linkar na §3 | Média | Baixo |
| Adicionar referência a `features/specs/` e `contracts/api/` | Baixa | Baixo |
| Consolidar status do dashboard em frase única na §5 | Média | Baixo |