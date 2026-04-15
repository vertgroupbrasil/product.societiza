---
status: draft
domain: identity
created_at: 2026-04-13
---

# Fluxo Completo de Convites

## 1. Visão da feature

O sistema de convites da Societiza é a porta de entrada de todos os usuários na plataforma. Ninguém se cadastra de forma autônoma — todo acesso começa por um convite. Quem entra, por qual porta e com qual papel são decisões controladas por quem já está dentro com autoridade para isso.

Esta spec define esse fluxo de ponta a ponta para os três cenários reais de convite na plataforma: SystemAdmin convidando outro SystemAdmin, SystemAdmin convidando o AccountingAdmin de uma nova contabilidade, e AccountingAdmin convidando funcionários para sua equipe.

A mudança central é introduzir um link genérico para o caso mais frequente — o AccountingAdmin adicionando funcionários — eliminando a dependência de email para esse cenário cotidiano. Para os cenários do SystemAdmin, que são raros e formais, o link continua sendo enviado por email para a pessoa específica.

---

## 2. Problema que resolve

Contabilidades pequenas operam por WhatsApp. Quando o AccountingAdmin precisa adicionar funcionários à plataforma, o sistema atual exige que ele informe o email de cada pessoa individualmente e aguarde a entrega do email de convite. Se o email cair em spam, chegar em uma caixa pouco monitorada ou ser ignorado, o processo trava. Para uma equipe de cinco funcionários, são cinco convites individuais, cinco esperas, e cinco potenciais problemas de entrega.

Esse atrito acontece exatamente no momento de maior impacto: o onboarding da equipe. O administrador não tem o link em mãos, não pode mandar pelo WhatsApp, e fica refém da entrega de email.

Para os cenários do SystemAdmin, o problema é menor mas existe: o link nunca aparece na tela após a criação do convite. Se algo der errado com o email, não há recurso para acompanhar por outro canal.

---

## 3. Para quem é

**SystemAdmin** — operador da própria Societiza. Realiza duas operações de convite, ambas raras e formais: criar novos SystemAdmins (expansão do time interno) e fazer o onboarding de novas contabilidades, convidando o AccountingAdmin único de cada uma.

**AccountingAdmin** — administrador único de uma contabilidade. Ele mesmo foi convidado pelo SystemAdmin no momento em que sua contabilidade foi cadastrada. Sua única operação de convite é adicionar funcionários (AccountingEmployee) à equipe. Esse é o fluxo de maior frequência na plataforma.

**Novo membro** — a pessoa sendo convidada. Pode ser um SystemAdmin, um AccountingAdmin (no momento de onboarding de uma nova contabilidade) ou um AccountingEmployee. Recebe o convite pelo canal apropriado, acessa o link, cria sua conta e entra na plataforma com o papel correto.

---

## 4. Os três cenários de convite

A plataforma tem exatamente três cenários de convite. Cada um tem características próprias.

| Cenário | Quem convida | Quem é convidado | Mecanismo |
|---|---|---|---|
| 1 | SystemAdmin | SystemAdmin | Link específico por email |
| 2 | SystemAdmin | AccountingAdmin (único da contabilidade recém-criada) | Link específico por email |
| 3 | AccountingAdmin | AccountingEmployee | Link genérico de equipe, válido por 7 dias |

**Regras estruturais:**
- Cada contabilidade tem **um único AccountingAdmin**, criado pelo SystemAdmin no momento de onboarding da contabilidade
- O AccountingAdmin **não convida outros AccountingAdmins** — esse papel é exclusivo do SystemAdmin
- O AccountingAdmin convida **apenas AccountingEmployees**, sempre por meio do link genérico de equipe

---

## 5. Como melhora a experiência atual

**Antes — fluxo de equipe (AccountingAdmin → Employee):**
O administrador entra na área de membros, informa o email e o papel de cada funcionário, um por um, e clica em convidar. Para cada um, o sistema dispara um email. O administrador não tem o link em mãos, não sabe se os emails chegaram, e se algum funcionário não aparecer, precisa voltar para reenviar individualmente.

**Depois — fluxo de equipe (AccountingAdmin → Employee):**
A área de membros exibe um único Link de Equipe, permanentemente disponível com botão de copiar. O administrador copia o link e manda no grupo da equipe ou para cada funcionário no WhatsApp. Cada funcionário clica, preenche nome, email e senha, e está dentro. Os novos funcionários aparecem na lista de membros da contabilidade conforme entram. Após 7 dias, o link expira e o administrador precisa renovar para que novos cadastros sejam aceitos — protegendo contra links que circularam além do tempo previsto.

**Antes — fluxos do SystemAdmin:**
O SystemAdmin informa o email da pessoa, o sistema dispara um email automaticamente e retorna apenas um identificador interno. O link nunca aparece na tela.

**Depois — fluxos do SystemAdmin:**
O email continua sendo enviado — é o canal esperado nesse contexto formal. Adicionalmente, o link aparece na tela imediatamente após a criação, permitindo ao SystemAdmin copiar e compartilhar por outro canal se necessário. Para o cenário de onboarding de nova contabilidade, a criação da contabilidade e o convite do AccountingAdmin são duas operações sequenciais e independentes: o SystemAdmin primeiro cadastra a contabilidade e, em seguida, gera o convite do administrador único daquela contabilidade.

---

## 6. Como se conecta com o produto existente

**Dependências obrigatórias:**
- O isolamento multi-tenant é preservado. Cada convite carrega a contabilidade de destino quando aplicável (cenários 2 e 3).
- O cenário 2 pressupõe que a contabilidade já foi criada. A criação da contabilidade e o convite do administrador são duas operações distintas, executadas em sequência pelo SystemAdmin.

**O que esta feature habilita:**
- Suporte a usuários que atuam em múltiplas contabilidades: como um AccountingEmployee pode ser um contador autônomo prestando serviço para mais de um escritório, esta spec permite que o mesmo email seja convidado por diferentes contabilidades. A experiência de seleção de contabilidade ativa no login deve ser especificada separadamente.

**Posição no produto:**
Feature core de acesso e colaboração. Pré-requisito para qualquer uso colaborativo real da plataforma. Não adiciona uma camada periférica — fortalece o núcleo.

---

## 7. O que não é (escopo negativo)

**Não inclui** convite de AccountingAdmin pelo próprio AccountingAdmin. Cada contabilidade tem um único AccountingAdmin, criado pelo SystemAdmin no onboarding. Não há mecanismo para o AccountingAdmin promover outro usuário a administrador nem convidar outro administrador.

**Não inclui** aprovação manual de entrada pelo link genérico. Quem usa o link da equipe entra automaticamente. O controle é exercido pela validade limitada de 7 dias do link e pela visibilidade dos membros na lista da equipe.

**Não inclui** notificação em tempo real ao admin quando alguém entra pelo link. O admin acompanha as entradas vendo a lista de membros da contabilidade. Notificações ficam fora do escopo desta spec e do MVP.

**Não inclui** link genérico para cenários do SystemAdmin. Esses fluxos permanecem com link específico por email — são operações sensíveis de plataforma.

**Não inclui** múltiplos links ativos de equipe simultaneamente. Há exatamente um link de equipe ativo por contabilidade em qualquer momento.

**Não inclui** rastreamento individual de quem visualizou o link sem completar o cadastro. Apenas entradas efetivas são registradas no metadado do link.

**Não inclui** a especificação da experiência de seleção de contabilidade ativa no login. Esta spec permite que um usuário pertença a múltiplas contabilidades (caso de contador autônomo); a tela de seleção é uma feature derivada que deve ser especificada separadamente.

**Não inclui** notificação ou aviso prévio de expiração do link de equipe. Após 7 dias, o link simplesmente para de aceitar novos cadastros — o admin renova quando precisar adicionar mais alguém.

---

## 8. Custos adicionais

Nenhum custo externo identificado.

Esta feature remove a dependência de provedor de email para o fluxo de convite de AccountingEmployees, reduzindo custos e dependências operacionais. Para os cenários do SystemAdmin, o envio de email é mantido, mas não representa um custo novo — já existe hoje.

---

## 9. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Cenário 1 — SystemAdmin → SystemAdmin | Link específico por email. Email automático enviado + link exibido na tela como complemento. |
| Cenário 2 — SystemAdmin → AccountingAdmin | Link específico por email. Operações sequenciais: SystemAdmin primeiro cadastra a contabilidade e depois gera o convite do admin único. |
| Cenário 3 — AccountingAdmin → AccountingEmployee | Link genérico de equipe. Vários funcionários podem se cadastrar pelo mesmo link. |
| Quantos AccountingAdmins por contabilidade | Exatamente um. Criado pelo SystemAdmin no onboarding. AccountingAdmin não convida outros admins. |
| Validade do link de equipe | 7 dias. Após expiração, admin renova manualmente para aceitar novos cadastros. |
| Renovação do link | Manual pelo AccountingAdmin. Sem aviso prévio antes da expiração. |
| Entrada pelo link de equipe | Automática, sem aprovação manual do admin. |
| Acompanhamento de entradas | Pela lista de membros da contabilidade. Sem notificação em tempo real — notificações ficam fora do MVP. |
| Visibilidade do link de equipe | Sempre visível na área de membros, com botão "Copiar" e metadado: "gerado em [data] · expira em [data] · [N] pessoas entraram." |
| Página de registro | Exibe o nome da contabilidade para dar contexto ao novo membro. |
| Email já cadastrado em outra contabilidade | Permitido. Apresenta fluxo de login; após autenticar, o usuário é adicionado à nova contabilidade automaticamente. |
| Múltiplas contabilidades por usuário | Permitido (caso de contador autônomo prestando serviço para múltiplos escritórios). |
| Quem gerencia o link de equipe | Exclusivamente o AccountingAdmin da contabilidade. |

---

## 10. Perguntas abertas

Nenhuma. Todas as decisões de produto estão fechadas.
