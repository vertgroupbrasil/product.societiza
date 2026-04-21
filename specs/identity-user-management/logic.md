---
status: draft
spec: identity-user-management
created_at: 2026-04-17
---

# Lógica de Negócio — Gestão de Usuários

## 1. Fluxo por perfil

### SystemAdmin

O SystemAdmin opera em dois contextos distintos: o painel administrativo da
plataforma e o contexto de uma contabilidade específica.

**No painel administrativo:**

1. Acessa a área de administração da plataforma.
2. Visualiza a lista de todos os SystemAdmins ativos — nome, e-mail, perfil,
   data de entrada.
3. Pode acessar a visão de inativos (SystemAdmins removidos) a partir da mesma
   área, em tab ou seção separada.
4. Para remover um SystemAdmin: seleciona o usuário, aciona "Remover acesso",
   confirma a ação. O sistema bloqueia se o alvo for o último SystemAdmin ativo
   — exibe mensagem explicando que a plataforma ficaria sem administrador.
5. Para editar o próprio nome: acessa o menu de perfil, edita o campo Nome,
   salva. Nenhuma confirmação adicional.

**No contexto de uma contabilidade:**

1. Entra no contexto de uma contabilidade específica (via painel administrativo).
2. Visualiza a lista de AccountancyAdmins e AccountancyEmployees daquela
   contabilidade — nome, e-mail, perfil, data de entrada, referência ao convite
   de origem (quando houver).
3. Pode acessar a visão de inativos da contabilidade na mesma área.
4. Para remover um membro: seleciona o usuário, aciona "Remover acesso",
   confirma. O sistema bloqueia se o alvo for o último AccountancyAdmin da
   contabilidade — exibe mensagem orientando a promover outro membro antes.
5. Se a ação for bem-sucedida: o usuário removido perde a sessão imediatamente.
   Ele desaparece da listagem ativa e passa a aparecer na visão de inativos.

**O que o SystemAdmin não pode fazer:**

- Promover AccountancyEmployee a AccountancyAdmin em nenhuma contabilidade.
  Se tentar, a ação não está disponível na interface — e o backend rejeita caso
  a requisição chegue.
- Editar o nome de qualquer outro usuário. O campo de nome é editável apenas
  pelo próprio usuário.
- Agir em lote sobre múltiplas contabilidades simultaneamente.

---

### AccountancyAdmin

O AccountancyAdmin gerencia a equipe da própria contabilidade. É o perfil que
mais opera esta feature no dia a dia.

**Listagem:**

1. Acessa a área de Equipe da contabilidade.
2. Visualiza todos os membros ativos — AccountancyAdmins e AccountancyEmployees
   — com nome, e-mail, perfil, data de entrada e referência ao convite de origem.
3. Pode acessar a visão de membros inativos (removidos) na mesma área, em tab
   ou seção separada.

**Promover um membro:**

1. Na lista de membros ativos, localiza o AccountancyEmployee que deseja
   promover.
2. Aciona "Promover a administrador".
3. Confirma a ação no modal de confirmação.
4. O perfil do membro é atualizado imediatamente. A interface reflete a mudança
   no próximo carregamento de tela. O membro promovido ganha acesso às ações de
   AccountancyAdmin sem precisar fazer logout.

**Remover um membro:**

1. Na lista de membros ativos, localiza o usuário que deseja remover.
2. Aciona "Remover acesso".
3. Confirma a ação no modal de confirmação.
4. Se o alvo for o último AccountancyAdmin da contabilidade (incluindo o próprio
   AccountancyAdmin tentando se auto-remover): a ação é bloqueada. A interface
   exibe mensagem orientando a promover outro membro a administrador antes.
5. Se a ação for bem-sucedida: o usuário removido perde a sessão imediatamente.
   Ele desaparece da listagem ativa e aparece na visão de inativos.

**Editar o próprio nome:**

1. Acessa o menu de perfil.
2. Edita o campo Nome.
3. Salva. Nenhuma confirmação adicional — a ação é reversível pelo próprio
   usuário a qualquer momento.

**O que o AccountancyAdmin não pode fazer:**

- Agir sobre usuários de outras contabilidades. Toda ação é restrita ao
  `accountancyId` da própria contabilidade.
- Editar o nome de outro usuário. Nem de AccountancyEmployee, nem de outro
  AccountancyAdmin.
- Rebaixar um AccountancyAdmin para AccountancyEmployee. Promoção é
  unidirecional no MVP. Se precisar reverter, remove e reconvida.
- Se auto-remover como último AccountancyAdmin. O bloqueio é automático.

---

### AccountancyEmployee

O AccountancyEmployee tem acesso de leitura à equipe e controle apenas sobre
o próprio perfil.

**Listagem:**

1. Acessa a área de Equipe da contabilidade.
2. Visualiza os membros ativos da contabilidade — nome e perfil apenas. Sem
   e-mail, sem data de entrada, sem referência de convite.
3. Não tem acesso à visão de membros inativos.

**Editar o próprio nome:**

1. Acessa o menu de perfil.
2. Edita o campo Nome.
3. Salva. Nenhuma confirmação adicional.

**O que o AccountancyEmployee não pode fazer:**

- Promover ou remover qualquer membro. As ações não estão disponíveis na
  interface — e o backend rejeita caso a requisição chegue.
- Editar o nome de qualquer outro usuário.
- Ver dados completos dos colegas (e-mail, data de entrada, convite de origem).

---

## 2. Estados e transições

### Membro (presença na contabilidade ou na plataforma)

| Estado | Descrição |
|--------|-----------|
| **Ativo** | Usuário faz parte da equipe, tem sessão válida, aparece na listagem ativa. |
| **Inativo** | Usuário foi removido por um admin. Sessão invalidada. Não aparece na listagem ativa. Identidade e histórico preservados. |

**Transições:**

- **Ativo → Inativo:** AccountancyAdmin ou SystemAdmin executa "Remover acesso"
  e confirma. O sistema valida que o alvo não é o último administrador antes de
  executar.
- **Inativo → Ativo:** Não existe. Remoção é terminal. Para voltar, o admin
  gera novo convite e a pessoa entra como novo usuário.

**O que bloqueia a transição Ativo → Inativo:**
- O alvo é o último AccountancyAdmin da contabilidade.
- O alvo é o último SystemAdmin da plataforma.
- O solicitante tenta remover a si mesmo como último administrador.

---

### Perfil dentro de uma contabilidade

| Estado | Descrição |
|--------|-----------|
| **AccountancyEmployee** | Membro com acesso operacional, sem autoridade administrativa. |
| **AccountancyAdmin** | Membro com autoridade de gestão da equipe e da contabilidade. |

**Transições:**

- **AccountancyEmployee → AccountancyAdmin:** AccountancyAdmin da própria
  contabilidade executa "Promover a administrador" e confirma.
- **AccountancyAdmin → AccountancyEmployee:** Não existe no MVP. Para reverter,
  remover e reconvidar.

**O que bloqueia a transição AccountancyEmployee → AccountancyAdmin:**
- O solicitante não é AccountancyAdmin da mesma contabilidade.
- O alvo não pertence à mesma contabilidade do solicitante.

**Efeito imediato da promoção:** as permissões de AccountancyAdmin são aplicadas
antes da resposta do servidor. O usuário promovido as vê no próximo carregamento
de tela, sem precisar fazer logout.

---

### Nome do usuário

| Estado | Descrição |
|--------|-----------|
| **Atual** | O nome como está registrado na conta. |
| **Atualizado** | O nome após edição pelo próprio usuário. |

**Transição:** qualquer usuário ativo edita o próprio nome e salva.
Não há bloqueio para esta ação (além de autenticação válida).
Não há confirmação adicional — a ação é reversível pelo próprio usuário.

---

## 3. Regras de negócio

- Todo usuário pode editar apenas o próprio nome — nenhum perfil edita dado
  pessoal de outro usuário.
- E-mail não é editável no MVP.
- AccountancyAdmin age apenas dentro da própria contabilidade — fronteira
  definida pelo `accountancyId`.
- AccountancyAdmin pode promover apenas AccountancyEmployee (não pode promover
  outro AccountancyAdmin ou fazer demotion).
- SystemAdmin não promove membros dentro de nenhuma contabilidade — promoção
  é ato interno da contabilidade.
- SystemAdmin pode remover qualquer perfil, inclusive em contabilidades alheias,
  como ação operacional de última instância.
- Remoção do último AccountancyAdmin de uma contabilidade → bloqueada para
  qualquer solicitante, inclusive SystemAdmin.
- Remoção do último SystemAdmin da plataforma → bloqueada.
- Nenhum admin pode se auto-remover — autoexclusão é proibida para todos os
  perfis.
- Remoção é sempre soft-delete: o usuário é marcado como inativo, não excluído
  fisicamente.
- Sessão do usuário removido é invalidada imediatamente — não no próximo login.
- Histórico de autoria (processos, etapas, tarefas, comentários, convites) criado
  pelo usuário removido é preservado integralmente — a referência ao criador
  permanece e o nome é exibido mesmo após a remoção.
- Convites ativos criados por um usuário removido permanecem ativos — não são
  auto-revogados. Outros admins da contabilidade podem revogá-los manualmente.
- Promoção tem efeito imediato — permissões de AccountancyAdmin aplicadas antes
  da resposta do servidor.
- Promoção e remoção exigem confirmação explícita do solicitante antes de
  executar.
- Edição de nome não exige confirmação — ação é reversível pelo próprio usuário.
- Listagem de membros ativos é a visão padrão — inativos ficam em visão
  separada, acessível apenas a admins.
- AccountancyEmployee vê apenas nome e perfil dos colegas — sem e-mail, sem
  data de entrada, sem referência de convite.
- AccountancyAdmin e SystemAdmin veem dados completos: nome, e-mail, perfil,
  data de entrada, referência ao convite de origem (quando houver).
- Toda validação de autorização acontece no backend — a interface apenas reflete
  o estado permitido.

---

## 4. Pontos de integração

```
Frontend precisa saber:
  - Perfil do usuário autenticado (SystemAdmin, AccountancyAdmin,
    AccountancyEmployee) para exibir ou ocultar os controles de gestão.
  - Se o membro listado é o último AccountancyAdmin da contabilidade, para
    desabilitar o botão "Remover acesso" ou exibir a mensagem correta antes
    de abrir o modal de confirmação.
  - Se o membro listado é o último SystemAdmin da plataforma (para o painel
    administrativo), pela mesma razão.
  - Quais membros estão ativos e quais estão inativos, para separar as duas
    visões corretamente.
  - Dados completos do membro (nome, e-mail, perfil, data de entrada,
    referência de convite) para admins — dados reduzidos (nome, perfil)
    para AccountancyEmployee.
  - O perfil do membro após promoção sem aguardar logout — resposta de
    sucesso deve incluir o novo perfil.
  - O nome atualizado do usuário autenticado após edição, para refletir na
    interface imediatamente (menu de perfil, cabeçalho, listagem).

Backend precisa garantir:
  - Toda operação de listagem, promoção e remoção filtrada pelo accountancyId
    do solicitante — sem exceção.
  - SystemAdmin tem acesso à listagem de qualquer contabilidade ao entrar no
    contexto dela, mas o escopo da ação é sempre a contabilidade em questão.
  - Verificação atômica de "último admin" antes de executar a remoção — sem
    race condition entre dois admins tentando se remover mutuamente.
  - Invalidação de sessão do usuário removido ocorre antes da resposta de
    sucesso ser enviada ao solicitante.
  - Soft-delete: o registro do usuário permanece com status inativo, não é
    excluído do banco.
  - Referências de autoria em processos, etapas, tarefas, comentários e
    convites não são alteradas pela remoção.
  - Convites criados pelo usuário removido não são afetados pela remoção.
  - Permissões do usuário promovido são atualizadas antes da resposta de
    sucesso — o token ou o estado de sessão do usuário promovido deve refletir
    o novo perfil no próximo carregamento.
  - Edição de nome é restrita ao próprio usuário autenticado — o backend
    valida que o `userId` do payload corresponde ao usuário da sessão.
  - Rejeitar qualquer tentativa de promoção feita por SystemAdmin.
  - Rejeitar qualquer tentativa de edição de dado pessoal de terceiro.
```

---

## 5. Casos de borda

### Dois AccountancyAdmins tentando remover um ao outro simultaneamente

Situação: Admin A e Admin B são os únicos AccountancyAdmins de uma
contabilidade. Em instâncias simultâneas, cada um inicia a remoção do outro.

Decisão fechada: a verificação de "último admin" deve ser atômica no backend.
Apenas uma das requisições pode passar. A segunda recebe a resposta de bloqueio
("não é possível remover o último administrador"). O frontend exibe a mensagem
de bloqueio ao solicitante que chegou segundo.

---

### SystemAdmin tenta remover o último AccountancyAdmin de uma contabilidade

Situação: uma contabilidade tem um único AccountancyAdmin ativo. O SystemAdmin
entra no contexto dessa contabilidade e tenta remover esse membro.

Decisão fechada: o bloqueio de "último AccountancyAdmin" se aplica a qualquer
solicitante, inclusive SystemAdmin. A proteção é estrutural — uma contabilidade
sem admin fica órfã. O SystemAdmin recebe a mesma mensagem de bloqueio: é
necessário promover outro membro antes de remover o único administrador.

---

### Usuário removido tenta usar sessão existente

Situação: o usuário está autenticado com uma sessão ativa. Enquanto navega na
plataforma, um admin executa a remoção do acesso dele.

Decisão fechada: a invalidação de sessão ocorre no ato da remoção, no backend.
Qualquer requisição subsequente com aquele token retorna não autorizado. O
frontend deve tratar esse retorno como sessão expirada e redirecionar para a
tela de login.

---

### Usuário promovido tenta usar permissões de AccountancyAdmin antes de recarregar

Situação: o usuário é promovido por um admin. Antes de recarregar a página, ele
tenta executar uma ação que exige AccountancyAdmin.

Decisão fechada: as permissões são aplicadas imediatamente no backend. O
frontend, ao receber a resposta da ação (mesmo que negada por estado de tela
desatualizado), deve atualizar o estado de permissões e permitir que o usuário
tente novamente. A ação não falha por causa de permissão — ela falha apenas se
o frontend ainda não reflectiu o novo estado. O backend aceita a requisição
desde que o perfil do usuário já tenha sido atualizado.

---

### Usuário removido aparece como autor em histórico futuro

Situação: a feature de processos (workflow-process) ainda não existe, mas quando
existir, processos criados antes da remoção terão o usuário removido como autor.

Decisão fechada: a referência de autoria usa o identificador do usuário (não
o estado ativo/inativo). O nome do usuário removido deve ser exibido em contextos
de histórico — não substituído por "usuário removido" ou similar. A lógica de
exibição do nome deve buscar o registro do usuário independentemente do status
de atividade. Esta é uma invariante que os sistemas de workflow e dashboard
devem respeitar ao consumir dados de autoria.

---

### Admin remove um usuário que gerou convites ativos

Situação: um AccountancyAdmin ou AccountancyEmployee gerou convites que ainda
estão pendentes (não expirados, não revogados). O usuário é removido.

Decisão fechada: os convites permanecem ativos após a remoção do criador. Não
há revogação automática em cascata. Os demais admins da contabilidade já
enxergam esses convites no fluxo de gestão de convites e podem revogá-los
manualmente se for o caso. Nenhuma ação automática é necessária no MVP.

---

### AccountancyEmployee tenta acessar dados completos de um colega

Situação: o frontend exibe a lista de membros para AccountancyEmployee. Por
alguma razão (bug, manipulação de requisição), a requisição inclui campos
completos (e-mail, data de entrada, referência de convite).

Decisão fechada: o backend filtra os dados retornados com base no perfil do
solicitante. AccountancyEmployee recebe apenas nome e perfil,
independentemente do que o frontend solicitou. A restrição de visibilidade é
aplicada no servidor.