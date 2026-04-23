---
status: approved
spec: identity-invitation-links
created_at: 2026-04-17
approved_at: 2026-04-23
---

# Lógica de Negócio — Links de Convite

## 1. Fluxo por perfil

### SystemAdmin

O SystemAdmin opera sem vínculo a nenhuma contabilidade. Ele pode criar dois
tipos de link: um que convida outro SystemAdmin (sem contabilidade associada)
e um que convida um AccountancyAdmin para uma contabilidade específica
(indicando qual contabilidade no momento da criação).

**Criando um link SA → SA:**

1. O SystemAdmin acessa sua área geral de convites (que contém apenas links SA→SA) e escolhe gerar um link para outro SystemAdmin.
2. O sistema cria o link com limite fixo de 1 uso e validade de 7 dias. O SystemAdmin não define o limite — ele é fixo.
3. O link fica disponível para cópia imediata. O SystemAdmin compartilha pelo canal que preferir.
4. O link fica visível na área de convites ativos do SystemAdmin.

**Criando um link SA → AA:**

1. O SystemAdmin navega até a contabilidade de destino (módulo Accountancy) e entra no contexto dela.
2. Dentro da área de convites da contabilidade, escolhe gerar um link para AccountancyAdmin.
3. O sistema cria o link com limite fixo de 1 uso e validade de 7 dias. A contabilidade vem do contexto de navegação — não é campo preenchido pelo SystemAdmin.
4. O link fica disponível para cópia e visível na área de convites ativos daquela contabilidade.

**Tentativas de ação proibida:**

O SystemAdmin não pode reativar um link em estado terminal. O SystemAdmin não
pode excluir links — o mecanismo de neutralização é exclusivamente a revogação.

---

### AccountancyAdmin

O AccountancyAdmin opera sempre dentro da sua própria contabilidade. Ele pode
criar dois tipos de link: um que convida outro AccountancyAdmin (para a mesma
contabilidade) e um que convida AccountancyEmployees (para a mesma
contabilidade).

**Criando um link AA → AA:**

1. O AccountancyAdmin acessa a área de convites e escolhe gerar um link para
   outro AccountancyAdmin da sua contabilidade.
2. O sistema cria o link com limite fixo de 1 uso e validade de 7 dias. O
   AccountancyAdmin não define o limite — ele é fixo.
3. O link fica disponível para cópia imediata.
4. Todos os AccountancyAdmins da mesma contabilidade enxergam este link.

**Criando um link AA → AE:**

1. O AccountancyAdmin acessa a área de convites e escolhe gerar um link para
   AccountancyEmployee.
2. O sistema pede que o AccountancyAdmin informe quantas pessoas vão entrar por
   aquele link (o `maxUses`). Esse é o único parâmetro que o admin define.
3. O sistema cria o link com o limite informado e validade de 7 dias.
4. O link fica disponível para cópia e visível para todos os AccountancyAdmins
   da contabilidade.

**Visualizando links da contabilidade:**

O AccountancyAdmin vê, por padrão, apenas os links ativos da sua contabilidade
— incluindo os criados por outros AccountancyAdmins do mesmo tenant. A visão
histórica (links esgotados, expirados e revogados) está disponível em separado.
Para cada link, o admin vê o perfil de destino, o limite de usos, a contagem
atual de usos, a data de criação, a data de expiração e quem criou o link.

**Tentativas de ação proibida:**

- Se um AccountancyAdmin tentar criar um link apontando para outra
  contabilidade: o sistema rejeita. A contabilidade do link é sempre a do
  próprio admin, sem exceção.
- Se tentar criar um link para um perfil superior ao seu (ex: SystemAdmin):
  o sistema rejeita.
- Se tentar reativar um link em estado terminal: o sistema rejeita.

---

### AccountancyEmployee

O AccountancyEmployee não interage com a criação, visualização nem gestão de
links de convite. Ele só aparece neste fluxo como beneficiário: entra no
sistema por um link criado para ele. Após criar sua conta e entrar, não tem
acesso à área de convites.

---

### Convidado (sem conta na Societiza)

1. Recebe o link pelo canal que o admin escolheu (WhatsApp, e-mail, etc.).
2. Acessa o link no navegador.
3. O sistema valida o link: verifica se está ativo (não expirado, não revogado,
   não esgotado).
4. Se válido: o sistema apresenta a tela de criação de conta. O convidado
   informa o e-mail que preferir e cria uma senha.
5. Ao concluir o cadastro, a conta é criada com o perfil e a contabilidade
   definidos pelo link. O contador de usos do link é incrementado.
6. A conta do novo usuário carrega a referência ao link pelo qual entrou.
7. Se o uso consumido for o último (usos = maxUses), o link transita
   automaticamente para esgotado.

---

### Convidado (com conta existente na Societiza)

1. Acessa o link no navegador.
2. O sistema detecta que o e-mail já está associado a uma conta.
3. O sistema bloqueia o acesso e apresenta mensagem explicativa.
4. Nenhuma absorção automática de conta acontece. Nenhuma associação a nova
   contabilidade. Nenhuma ação adicional do sistema.

---

## 2. Estados e transições

### Link de convite

Um link de convite existe em um de três estados:

**Ativo**
O link está dentro do prazo de validade (7 dias a partir da criação) e tem usos
disponíveis (usos atuais < maxUses). Só links ativos podem
ser usados por convidados e aparecem na visão padrão da área de convites.

**Esgotado**
O limite de usos foi atingido (usos atuais = maxUses). Transição causada pelo
sistema no momento em que o último uso é consumido. Estado terminal.

**Expirado**
O prazo de 7 dias venceu sem que o limite de usos tenha sido atingido.
Transição causada pelo sistema pela passagem do tempo. Estado terminal.

**Diagrama de transições:**

```
[Criado] → Ativo
Ativo → Esgotado   (último uso consumido)
Ativo → Expirado   (7 dias passados)
```

Nenhum estado terminal tem transição de saída. Não existe reativação.

---

### Conta criada via link

Quando o convidado usa um link e cria sua conta, a conta nasce com:
- O perfil de acesso definido pelo link (SystemAdmin, AccountancyAdmin ou
  AccountancyEmployee).
- A contabilidade definida pelo link (quando aplicável).
- A referência ao link de origem.

Este vínculo é imutável — a referência de origem não muda após a criação da
conta. Mudar o perfil de um usuário depois (ex: promoção de AE para AA) é um
fluxo separado de gestão de usuário, fora do escopo desta feature.

---

## 3. Regras de negócio

- SystemAdmin pode criar links SA→SA e SA→AA apenas.
- AccountancyAdmin pode criar links AA→AA e AA→AE apenas.
- AccountancyAdmin só cria links dentro da sua própria contabilidade. A contabilidade do link é sempre a do admin que o criou.
- Links SA→SA têm limite fixo de 1 uso. Não configurável.
- Links SA→AA têm limite fixo de 1 uso. Não configurável.
- Links AA→AA têm limite fixo de 1 uso. Não configurável.
- Links AA→AE têm limite definido pelo AccountancyAdmin no momento da criação. O admin informa o valor; o sistema não impõe um teto máximo nesta versão.
- **Unicidade de link ativo por tipo e escopo:** em qualquer momento, só pode existir um único link ativo para cada combinação de tipo e escopo. O escopo é definido pelo par (tipo de link, contabilidade de destino quando aplicável):
  - Um único link SA→SA ativo globalmente.
  - Um único link SA→AA ativo por contabilidade de destino.
  - Um único link AA→AA ativo por contabilidade.
  - Um único link AA→AE ativo por contabilidade.
- A tentativa de criar um segundo link ativo no mesmo escopo é rejeitada pelo sistema. O admin precisa aguardar transição natural para estado terminal (esgotado ou expirado) antes de criar um novo.
- A checagem de unicidade é feita via consulta à projection imediatamente antes da criação (soft check). Não é atômica — race conditions de criação simultânea são um risco aceito no MVP de baixo volume.
- A unicidade se aplica apenas a links no estado ativo. Links em estados terminais (esgotado, expirado) não bloqueiam a criação de um novo link no mesmo escopo.
- Todo link expira exatamente 7 dias após a criação, independente do perfil ou tipo.
- Um link só pode ser usado se estiver ativo: dentro do prazo e com usos disponíveis. As condições são verificadas no momento do uso.
- Ao ser usado, o contador de usos do link é incrementado. Se o incremento leva o contador ao limite, o link transita para esgotado no mesmo instante.
- Convidado com conta existente na Societiza não pode usar o link. O sistema bloqueia com mensagem. Nenhuma conta é criada ou modificada.
- Todo usuário criado via link carrega a referência ao link de origem. Esta referência não é modificável após a criação da conta.
- O link SA→AA é sempre criado no contexto de uma contabilidade específica. A contabilidade vem do contexto de navegação, não de seleção em formulário.
- O link SA→AA aparece na área de convites da contabilidade de destino — visível para AccountancyAdmins do tenant e para o SystemAdmin quando ele acessa o contexto daquela contabilidade. Não aparece na área geral de convites do SystemAdmin.
- Nenhum link é excluído. Revogação é o único mecanismo de neutralização.
- Links esgotados, expirados e revogados permanecem visíveis no histórico com status explícito.
- A visão padrão da área de convites exibe apenas links ativos. Links inativos ficam na visão histórica separada.
- Todos os AccountancyAdmins de uma mesma contabilidade enxergam todos os links daquela contabilidade (criados por qualquer um dos admins do tenant).
- Usuários que já entraram por um link não são afetados pela revogação posterior desse link. A revogação impede novos usos — não desfaz usos anteriores.

---

## 4. Pontos de integração

```
Frontend precisa saber:
  - Status atual do link (ativo, esgotado, expirado, revogado)
  - Contagem atual de usos e o limite máximo (para exibir "2/5", "1/1", etc.)
  - Data de criação e data de expiração do link (para calcular e exibir aviso
    de expiração iminente — o cálculo é responsabilidade do frontend)
  - Perfil de destino do link (SystemAdmin, AccountancyAdmin, AccountancyEmployee)
  - Contabilidade de destino do link (quando SA → AA)
  - Quem criou o link (nome ou identificador do admin criador)
  - Lista de usuários que entraram por aquele link (nome e data de entrada),
    acessível a partir do link na visão histórica ou no detalhe do link ativo
  - Qual é a contabilidade do admin autenticado (para saber quais links ele
    pode criar e revogar)
  - Resultado do cadastro via link: se bem-sucedido ou, em caso de erro, o
    motivo (link expirado, link esgotado, e-mail já existente).
  - Se já existe um link ativo no escopo em questão (tipo + contabilidade de
    destino quando aplicável) — para desabilitar o botão de criar novo link e
    sinalizar ao admin que ele precisa revogar o link ativo atual antes de
    gerar outro, ou tratar a resposta de conflito retornada pelo backend na
    tentativa de criação

Backend precisa garantir:
  - Verificação das três condições de usabilidade no momento do uso: status
    ativo, usos disponíveis (Uses < MaxUses) e prazo não vencido
    (ExpiresAt > now).
  - Incremento atômico do contador de usos — impede que dois convidados
    simultâneos ambos passem com o último uso disponível
  - Transição automática para "esgotado" no momento em que o último uso é
    consumido — não em job assíncrono, no ato do uso
  - Expiração calculada no momento da validação do link, não apenas via job
    de varredura — o link expirado é rejeitado mesmo sem ter sido marcado
    previamente por um job
  - Isolamento multi-tenant na criação de links: a contabilidade do link é
    sempre a do contexto de navegação ativo — o backend não aceita
    `accountancyId` enviado pelo frontend, nem para links de AccountancyAdmin
    nem para links SA→AA criados pelo SystemAdmin dentro de uma contabilidade
  - Isolamento multi-tenant na listagem: AccountancyAdmin vê apenas links da
    sua contabilidade; SystemAdmin vê todos
  - Checagem soft de unicidade de link ativo por tipo e escopo na criação —
    a checagem de "existe link ativo no mesmo escopo?" é feita via consulta
    à projection imediatamente antes da criação. Não é uma operação atômica;
    race conditions de criação simultânea são um risco aceito no MVP de
    baixo volume (ver Seção 3). Escopo é o par (tipo, contabilidade de destino
    quando aplicável). Resposta de conflito distinguível pelo frontend, para
    permitir mensagem específica ao admin
  - Referência ao link de origem gravada na conta criada, de forma não
    modificável após a criação
  - Bloqueio de uso do link por usuário com conta existente, com motivo
    distinguível pelo frontend (para apresentar mensagem adequada)
```

---

## 5. Casos de borda

### Race condition no último uso
Dois convidados acessam e tentam usar o link simultaneamente quando apenas 1
uso resta. **Decisão:** o backend garante atomicidade — apenas um dos dois
sucede. O segundo recebe a resposta de link esgotado, como se o link já
estivesse no estado esgotado. O frontend apresenta a mensagem correspondente.

### Link expirado sem varredura prévia
Um link pode chegar ao momento do uso com o prazo vencido mas sem ter sido
marcado por um job de expiração. **Decisão:** a validação de expiração é feita
em tempo real, no momento do uso, comparando `expiresAt` com o instante atual.
O job de varredura (se existir) é auxiliar, não autoritativo.

### AccountancyAdmin tenta ver links de outra contabilidade
**Decisão:** o sistema não retorna dados de outra contabilidade. A listagem é
sempre filtrada pela contabilidade do admin autenticado. O sistema não expõe
nem a existência de links de outros tenants.

### Convidado acessa link com conta existente em contabilidade diferente
Um usuário tem conta na Societiza (em outra contabilidade) e tenta entrar por
um link de uma nova contabilidade. **Decisão:** bloqueado. Mensagem explicativa.
Nenhuma conta nova é criada, nenhuma associação acontece. O sistema atual
pressupõe que um usuário pertence a uma única contabilidade — multi-contabilidade
é fora do escopo desta versão.

### Link AA→AE com maxUses informado como zero ou negativo
O admin tenta criar um link com limite inválido. **Decisão:** o sistema rejeita
na criação. O limite mínimo é 1. O backend valida antes de criar o link.

### Tentativa de criar segundo link ativo no mesmo escopo
Um admin tenta gerar um novo link quando já existe outro ativo no mesmo escopo
— por exemplo, um AccountancyAdmin pede um novo link AA→AE quando já há um
link AA→AE ativo na contabilidade, ou um SystemAdmin pede um novo link SA→AA
para uma contabilidade que já tem um link SA→AA ativo. **Decisão:** o sistema
rejeita a criação com resposta de conflito. A mensagem indica que já existe
link ativo no escopo e orienta o admin a revogar o link existente antes de
criar um novo, ou aguardar transição natural. Nenhum link novo é gerado. O
link ativo preexistente permanece intacto.

### Criação simultânea por dois admins do mesmo tenant
Dois AccountancyAdmins do mesmo tenant tentam criar, no mesmo instante, um
link do mesmo tipo para a mesma contabilidade. **Decisão:** o backend faz
checagem soft via projection antes de criar. Em volume baixo, a probabilidade
de race condition é aceita como risco de MVP. A segunda criação pode resultar
em dois links ativos no mesmo escopo em cenário de concorrência extrema.