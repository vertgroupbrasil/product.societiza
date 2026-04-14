---
status: draft
domain: identity
created_at: 2026-04-13
---

# Fluxo Completo de Convites

## 1. Visão da feature

O sistema de convites da Societiza é a porta de entrada de todos os usuários na plataforma. Ninguém se cadastra de forma autônoma — todo acesso começa por um convite. Quem entra, por qual porta e com qual papel são decisões controladas por quem já está dentro com autoridade para isso.

Esta spec define esse fluxo de ponta a ponta: quem pode convidar quem, como o convite chega, o que acontece quando é aceito e como o produto garante que cada pessoa entre no lugar certo com o papel certo.

A mudança central é eliminar a dependência de email para o caso mais frequente — administradores de contabilidade montando suas equipes. Em vez de o produto enviar um email automaticamente, o administrador recebe um link permanente que ele copia e envia pelo canal que já usa no dia a dia. Para operações de plataforma feitas pelo time da Societiza, o email se mantém como canal principal, complementado pelo link exibido na tela.

---

## 2. Problema que resolve

Contabilidades pequenas operam por WhatsApp. Quando o administrador precisa adicionar um novo funcionário à plataforma, o sistema atual exige que ele informe o email da pessoa e aguarde o email de convite ser entregue, visualizado e clicado. Se o email cair em spam, chegar em uma caixa pouco monitorada ou simplesmente ser ignorado, o processo trava. O administrador precisa voltar à plataforma para reenviar, sem nenhuma garantia de que desta vez será visto.

Esse atrito acontece exatamente no momento de maior impacto: o onboarding da equipe. Um produto que promete clareza e controle operacional não pode depender de um canal externo que o próprio usuário não controla para montar sua equipe.

O problema secundário é que o fluxo atual não distingue os diferentes tipos de convite nem deixa claro ao administrador o que acontece após o convite ser criado — o link vai para o email e some. O administrador fica sem nenhum recurso nas mãos.

---

## 3. Para quem é

**AccountingAdmin** — o administrador de uma contabilidade. É quem monta a equipe, adiciona novos funcionários e eventualmente adiciona outro administrador. Esse é o fluxo de maior frequência na plataforma, especialmente durante o onboarding inicial da contabilidade. Esse usuário se comunica com sua equipe principalmente por canais diretos como WhatsApp.

**SystemAdmin** — o operador da própria Societiza. Realiza operações de plataforma: cria novos SystemAdmins e faz o onboarding de novas contabilidades (criando a contabilidade e convidando o primeiro administrador). São operações raras, formais e de alta sensibilidade.

**Novo membro** — a pessoa sendo convidada. Recebe o convite pelo canal que o convidante escolher, acessa o link, cria sua conta e entra na plataforma com o papel correto desde o início, sem precisar de aprovação adicional.

---

## 4. Como melhora a experiência atual

**Antes — fluxo de equipe (AccountingAdmin):**
O administrador entra na área de membros, informa o email e o papel do novo funcionário, clica em convidar. O sistema dispara um email. O administrador não tem o link em mãos, não sabe se o email chegou, e se o funcionário não aparecer, precisa voltar para reenviar. Para avisar pelo WhatsApp, não tem nada concreto para enviar.

**Depois — fluxo de equipe (AccountingAdmin):**
A área de membros exibe dois links permanentes: um Link de Administrador e um Link de Membro. O administrador copia o link certo e manda no WhatsApp: *"Oi João, entra aqui pra acessar nossa plataforma."* O funcionário clica, preenche nome, email e senha, e já está dentro. O administrador recebe uma notificação imediata: *"João Silva entrou como Membro da sua equipe."* Sem dependência de email, sem reenvio, sem espera.

**Antes — fluxo de plataforma (SystemAdmin):**
O SystemAdmin informa o email da pessoa, o sistema dispara um email e retorna apenas um identificador interno. O link nunca aparece na tela.

**Depois — fluxo de plataforma (SystemAdmin):**
O email continua sendo enviado — é o canal esperado nesse contexto formal. Mas agora o link também aparece na tela imediatamente após a criação, permitindo que o SystemAdmin o copie e compartilhe por outro canal se necessário. Para onboarding de novas contabilidades, a criação da contabilidade e o convite ao primeiro administrador acontecem em um único fluxo unificado.

---

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**
- Módulo `identity` — esta spec modifica e expande o fluxo de convite existente. O mecanismo de aceitar o convite e criar a conta é reaproveitado; o que muda é como o link é gerado, distribuído e monitorado.
- `AccountancyId` — o isolamento multi-tenant é preservado. Cada convite — seja link genérico ou link por email — carrega a contabilidade de destino. Quem usa o link entra especificamente naquela contabilidade.
- Módulo `accountancy` — o onboarding de AccountingAdmin pressupõe que a contabilidade já existe. No fluxo unificado do SystemAdmin, a criação da contabilidade e o convite acontecem juntos.

**Dependência implícita — remoção de membros:**
Para fechar o ciclo de controle dos links genéricos, o administrador precisa conseguir remover um membro da equipe caso não reconheça quem entrou pelo link. Hoje essa funcionalidade não existe na plataforma. Esta feature e a remoção de membros andam juntas.

**O que esta feature habilita:**
- Promoção de papel: um AccountingEmployee promovido a AccountingAdmin sem novo convite. O backend já tem essa política planejada; esta spec a habilita e deve ser especificada separadamente.
- Seleção de contabilidade ativa: com múltiplas contabilidades por usuário, o produto precisará de um mecanismo de troca de contexto. Esta spec cria a necessidade; a experiência de seleção deve ser especificada separadamente.
- Notificações de equipe: a notificação de entrada pelo link é a base para um futuro feed de atividades de equipe.

**Posição no produto:**
Feature core do módulo `identity`. Pré-requisito para qualquer uso colaborativo real da plataforma. Não adiciona uma camada periférica — fortalece o núcleo.

---

## 6. O que não é (escopo negativo)

**Não inclui** aprovação manual de entrada. Quem usa o link genérico entra automaticamente. O controle é exercido pela notificação de entrada e pela capacidade de remover membros.

**Não inclui** expiração automática dos links genéricos por tempo ou por número de usos. O link é válido indefinidamente até o administrador gerar um novo.

**Não inclui** múltiplos links ativos do mesmo tipo simultaneamente. Há exatamente um Link de Administrador e um Link de Membro ativos por contabilidade em qualquer momento.

**Não inclui** link genérico para operações do SystemAdmin. Esses fluxos permanecem com link específico por email — são operações sensíveis de plataforma.

**Não inclui** rastreamento individual de quem visualizou o link sem completar o cadastro. Apenas entradas efetivas são registradas no metadado do link.

**Não inclui** a especificação da experiência de seleção de contabilidade ativa no login. Esta spec decide que múltiplas contabilidades são permitidas e que existe uma tela de seleção; a experiência detalhada dessa tela deve ser especificada separadamente.

**Não inclui** a especificação do fluxo de promoção de papel. Esta spec habilita a promoção; o fluxo completo deve ser especificado separadamente.

---

## 7. Custos adicionais

Nenhum custo externo identificado.

Esta feature remove a dependência de provedor de email para os fluxos de convite do AccountingAdmin, reduzindo custos e dependências operacionais. Para os fluxos do SystemAdmin, o envio de email é mantido, mas não representa um custo novo — já existe hoje.

---

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Mecanismo para AccountingAdmin | Dois links genéricos permanentes por contabilidade: Link de Administrador e Link de Membro |
| Mecanismo para SystemAdmin | Link específico por email: email automático enviado + link exibido na tela como complemento |
| Perfil ao usar o Link de Administrador | AccountingAdmin da contabilidade |
| Perfil ao usar o Link de Membro | AccountingEmployee da contabilidade |
| Entrada pelo link genérico | Automática, sem aprovação manual |
| Notificação ao admin | Imediata quando alguém entra por qualquer um dos links genéricos |
| Validade dos links genéricos | Indefinida — expira apenas quando o admin gera um novo |
| Confirmação ao gerar novo link | "O link atual vai parar de funcionar imediatamente. Continuar?" |
| Metadado do link | "gerado em [data] · [N] pessoas entraram por este link" |
| Email já cadastrado em outra contabilidade | Permitido. Apresenta fluxo de login; após autenticar, usuário é adicionado à nova contabilidade automaticamente |
| Múltiplas contabilidades por usuário | Permitido. Um usuário pode pertencer a mais de uma contabilidade |
| Seleção de contabilidade ativa | Tela de seleção simples após login. Produto lembra a última usada como padrão. Troca acessível a qualquer momento |
| Onboarding de AccountingAdmin pelo SystemAdmin | Fluxo unificado: criação da contabilidade e convite ao primeiro admin na mesma tela |
| Página de registro | Exibe o nome da contabilidade para dar contexto ao novo membro |
| Quem gerencia os links genéricos | Exclusivamente o AccountingAdmin |
| Remoção de membros | Dependência reconhecida. As duas features andam juntas |
| Promoção de papel | Habilitada por esta spec. Deve ser especificada separadamente |

---

## 9. Perguntas abertas

Nenhuma. Todas as decisões de produto estão fechadas.
