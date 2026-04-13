---
status: draft
domain: identity
created_at: 2026-04-13
---

# Convite por Link Copiável

## 1. Visão da feature

Hoje, quando o administrador de uma contabilidade quer adicionar um novo membro à equipe, ele precisa informar o email da pessoa e o sistema dispara um convite por email automaticamente. Esse fluxo depende de integração externa de email e tira do administrador o controle sobre como e quando o convite chega.

Com esta mudança, o fluxo se torna mais simples e direto: a área de membros exibe um link de convite único para a contabilidade. O administrador copia o link e envia para quem quiser adicionar à equipe — por WhatsApp, mensagem, ou qualquer canal que já use no dia a dia. Quem acessar o link preenche seus dados e já entra na equipe automaticamente, sem espera e sem aprovação.

O administrador recebe uma notificação imediata quando alguém entra pelo link, mantendo visibilidade total sem nenhuma burocracia no caminho. O link fica ativo indefinidamente — o administrador só precisa gerar um novo quando quiser invalidar o anterior, por exemplo após uma saída de equipe.

---

## 2. Problema que resolve

O administrador de uma contabilidade se comunica com sua equipe principalmente por canais diretos — WhatsApp na maioria dos casos. O fluxo atual de convite exige que ele informe o email da pessoa, espere o sistema enviar o email, torça para o email não cair em spam, e aguarde o funcionário acessar a caixa de entrada e agir. Se algo der errado nesse caminho, o administrador precisa voltar à plataforma, revogar o convite antigo e criar outro.

Esse atrito acontece em um momento crítico: quando a contabilidade está montando a equipe. Um onboarding travado por um email não recebido é uma fricção desnecessária em um produto que precisa causar boa primeira impressão.

O problema real é que o produto está controlando o canal de comunicação e adicionando dependência externa em vez de deixar isso com quem conhece a equipe — o próprio administrador.

---

## 3. Para quem é

**Beneficiário direto:** O AccountingAdmin — o administrador da contabilidade que gerencia a equipe. É ele quem acessa o link, copia e decide para quem e como enviar.

**Beneficiário indireto:** O novo membro (AccountingEmployee) — recebe o convite pelo canal que já usa, clica no link e já está dentro em segundos, sem precisar procurar um email em uma caixa que talvez não monitore com frequência.

**Momento da jornada:** No onboarding inicial da contabilidade (quando o administrador está montando a equipe pela primeira vez) e sempre que um novo funcionário precisa ser adicionado.

---

## 4. Como melhora a experiência atual

**Antes:**
O administrador entra na área de membros, informa o email e o perfil do novo membro, e clica em convidar. O sistema dispara um email automaticamente. O administrador não tem o link em mãos e não sabe se o email foi recebido. Se o funcionário não aparecer, o administrador precisa voltar à plataforma para reenviar — ainda sem garantia de que o email será visto. Para avisar por WhatsApp, não há nada concreto para enviar além de "olha o email".

**Depois:**
A área de membros exibe permanentemente o link de convite da contabilidade com um botão "Copiar link". O administrador copia o link e manda no WhatsApp: "Oi João, entra aqui pra acessar nossa plataforma." O funcionário clica, preenche nome, email e senha, e já está dentro — sem esperar aprovação. O administrador recebe uma notificação: *"João Silva acabou de entrar na sua equipe."* Sem dependência de email, sem reenvio, sem espera.

---

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**
- Módulo `identity` — esta feature modifica o comportamento do fluxo de convite que já existe. A lógica de criação de conta do membro é reaproveitada; o que muda é como o link é gerado, distribuído e monitorado.
- `AccountancyId` — o link de convite continua scoped à contabilidade do administrador. Quem usa o link entra especificamente naquela contabilidade.

**Dependência implícita — remoção de membros:**
Para que o ciclo de controle seja completo, o administrador precisa conseguir remover um membro da equipe caso não reconheça quem entrou pelo link. Hoje essa funcionalidade não existe na plataforma. Esta feature e a feature de remoção de membros andam juntas para fechar o ciclo de segurança.

**O que esta feature habilita no futuro:**
- Remove a dependência de email do fluxo de onboarding de membros, deixando a integração de email disponível exclusivamente para fluxos que realmente precisam dela (como recuperação de senha).
- Simplifica o gerenciamento de convites: em vez de rastrear convites individuais por pessoa, o administrador gerencia um único link ativo.
- Estabelece o padrão de notificação de atividade de equipe, que pode evoluir para um feed de atividades no futuro.

**Posição no produto:**
Feature core do módulo `identity`. Simplifica e fortalece um fluxo existente que é pré-requisito para qualquer uso colaborativo da plataforma.

---

## 6. O que não é (escopo negativo)

**Não é** um convite individual pré-configurado por email. O administrador não precisa informar o email de destino antes de gerar o link. Qualquer pessoa com o link pode criar uma conta e entrar.

**Não inclui** aprovação manual pelo administrador. Quem usa o link entra automaticamente — a visibilidade vem pela notificação, não por uma fila de aprovação.

**Não inclui** expiração automática por tempo ou por número de usos. O link é válido indefinidamente até o administrador gerar um novo.

**Não inclui** múltiplos links ativos simultaneamente. Nesta versão, há um único link de convite ativo por contabilidade.

**Não inclui** envio automático de email no fluxo de convite. A integração de email é removida deste fluxo inteiramente.

**Não inclui** rastreamento individual de quem visualizou o link sem entrar.

**Não é** uma mudança no fluxo de criação de conta após clicar no link. A pessoa preenche seus dados e cria a senha — esse fluxo não muda.

---

## 7. Custos adicionais

Nenhum custo externo identificado.

Esta feature elimina a dependência de um provedor de email para o fluxo de convite, reduzindo custos e dependências operacionais. Não introduz nenhum serviço externo, API de terceiros ou infraestrutura adicional.

---

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Tipo de link | Link genérico de workspace — qualquer pessoa com o link pode entrar. Sem predefinição de email. |
| Canal de entrega | O produto não envia o convite. O administrador escolhe o canal e envia o link por conta própria. |
| Expiração do link | Sem prazo automático e sem limite de usos. O link só expira quando o administrador gerar um novo. |
| Entrada do novo membro | Automática — quem usa o link entra direto, sem aprovação do administrador. |
| Visibilidade de entrada | O administrador recebe notificação imediata quando alguém entra pelo link: *"[Nome] acabou de entrar na sua equipe."* |
| Perfil do novo membro | AccountingEmployee sempre. O administrador ajusta permissões depois, se necessário. |
| Integração de email no convite | Removida do fluxo de convite. Nenhum email é disparado ao criar ou gerenciar o link. |
| Quantidade de links ativos | Um único link ativo por contabilidade. |
| Visibilidade do link | Sempre visível na área de membros, com botão "Copiar" e metadado: *"gerado em [data] · [N] pessoas entraram por este link."* |
| Quem gerencia o link | Exclusivamente o AccountingAdmin. |
| Email já cadastrado na plataforma em outra contabilidade | Permitido. Um usuário pode pertencer a múltiplas contabilidades. Se o email já está cadastrado, a página de registro detecta isso e apresenta um fluxo de login — após autenticar, o usuário é adicionado à nova contabilidade automaticamente, sem precisar criar uma nova conta. |
| Página de registro | Exibe o nome da contabilidade para dar contexto ao novo membro sobre onde está entrando. |
| Fluxo de criação de conta | Sem alteração. Quem acessa o link preenche nome, email e senha normalmente. |
| Dependência de remoção de membros | Reconhecida como dependência implícita para fechar o ciclo de controle. As duas features andam juntas. |

---

## 9. Perguntas abertas

Nenhuma. Todas as decisões de produto estão fechadas.
