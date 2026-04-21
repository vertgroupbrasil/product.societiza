---
status: approved
domain: tooling
created_at: 2026-04-20
approved_at: 2026-04-20
---

# Agente Tech-Lead de Integração

## 1. Visão da feature

Quando um desenvolvedor backend conclui a implementação de uma feature em `api.societiza`, ele não precisa documentar manualmente como o que foi construído se conecta com a spec de produto aprovada. Um agente especializado — o tech-lead agent — vive dentro do repositório `api.societiza`, tem acesso direto ao código implementado e às specs aprovadas em `product.societiza`, e produz o documento `integration.md` de forma autônoma.

Após produzir o documento, o agente abre um PR cross-repo diretamente em `product.societiza` para revisão humana — sem que o repositório de produto precise tocar no código-fonte do backend.

O agente age como um tech-lead que conhece profundamente o que foi construído e sabe traduzir isso para o contrato que o frontend precisa consumir.

## 2. Problema que resolve

Hoje, produzir o `integration.md` exige que alguém leia o código do backend e o cruze manualmente com a spec de produto aprovada. Isso significa uma de duas coisas: dar ao repositório `product.societiza` acesso ao código-fonte de `api.societiza` (indesejável pela separação de responsabilidades), ou depender de quem implementou para escrever a documentação manualmente (raro, demorado, sujeito a lacunas).

O resultado prático: o `integration.md` chega tarde, é incompleto ou não existe. O frontend começa a integração sem saber exatamente o que o backend entregou, gerando ruído entre times, retrabalho e atrasos.

A frequência é alta — cada feature implementada no backend dispara esse problema. E hoje não existe nenhum mecanismo estruturado para resolvê-lo.

## 3. Para quem é

**Beneficiário direto:** o desenvolvedor backend que concluiu a implementação. Ele aciona o agente e recebe um PR pronto para revisão, sem precisar escrever documentação manualmente.

**Beneficiário indireto:** o desenvolvedor frontend, que passa a ter um contrato preciso e atualizado para integrar a feature — produzido por quem conhece o código, não por quem leu o código de fora.

**Perfil:** time de desenvolvimento interno da Societiza. Esta feature não serve usuários finais da plataforma — serve o fluxo de trabalho entre os repositórios de execução.

## 4. Como melhora a experiência atual

**Antes:**
O backend conclui a implementação. Para que o `integration.md` seja produzido com precisão, `product.societiza` precisaria ter acesso ao código de `api.societiza` — ou o dev precisaria escrever manualmente. Na prática, o documento não é produzido no momento certo, o frontend começa a integrar sem ele, e a documentação se torna retroativa ou não acontece.

**Depois:**
O dev aciona o agente ao concluir a implementação. O agente lê a spec aprovada (`spec.md` + `logic.md`) em `product.societiza`, analisa o código implementado em `api.societiza`, produz o `integration.md` seguindo o template canônico e abre um PR cross-repo em `product.societiza` para revisão. O frontend recebe o contrato antes de iniciar a integração.

## 5. Como se conecta com o produto existente

**Dependências obrigatórias:**
- Specs aprovadas (`spec.md` + `logic.md`) em `product.societiza` — o agente as lê via MCP do GitHub
- Template `integration.md` em `product.societiza/templates/integration.md` — base do documento produzido
- Acesso via API ao GitHub — necessário para criar branches e abrir PRs cross-repo em `product.societiza`

**O que esta feature habilita no futuro:**
- Fecha o ciclo de documentação do workflow `spec → logic → integration` de forma automatizada
- Permite que o frontend comece integrações com contratos confiáveis e produzidos no momento certo
- Abre caminho para validação automática de consistência entre spec aprovada e código implementado

**Posição no produto:**
Feature de tooling interno. Não é uma feature de produto para o usuário final da Societiza — é infraestrutura do processo de desenvolvimento. Fortalece a qualidade e velocidade do ciclo de entrega.

## 6. O que não é (escopo negativo)

**Não é** um validador de implementação. O agente documenta o que foi construído — não julga se está correto em relação à spec.

**Não é** um agente de aprovação. O PR que ele cria ainda requer revisão e aprovação humana antes de ser mergeado.

**Não inclui** geração de `spec.md` ou `logic.md`. O agente só produz `integration.md` — os artefatos anteriores continuam sendo responsabilidade de `product.societiza`.

**Não inclui** deploy ou execução de código. O agente lê, analisa e escreve documentação — nada mais.

**Não inclui** suporte a múltiplas features simultâneas na mesma execução. Cada acionamento documenta uma feature específica.

## 7. Custos adicionais

**GitHub API:** necessário para criar branches e PRs cross-repo em `product.societiza`. Dentro do plano atual do GitHub se o repositório já está hospedado lá — sem custo adicional de assinatura previsto.

**Claude API (tokens de LLM):** cada execução do agente consome tokens para leitura da spec, análise do código e geração do `integration.md`. Custo variável por execução — esperado ser baixo por feature (leitura + geração de um documento estruturado).

Nenhum outro custo externo identificado.

## 8. Decisões tomadas

| Ponto | Decisão |
|---|---|
| Onde o agente vive | Em `api.societiza` — o agente precisa de acesso nativo ao código implementado |
| Quem produz o `integration.md` | O agente, não `product.societiza` — inversão intencional de responsabilidade |
| Como lê as specs | Via MCP do GitHub apontado para `product.societiza` |
| Como entrega o artefato | Abre um PR cross-repo em `product.societiza` — não commita direto na main |
| Aprovação do PR | Humana — o agente cria, o tech lead ou dev responsável aprova |
| Template base | `product.societiza/templates/integration.md` — o agente segue o template canônico |
| Acionamento | Manual — o dev aciona explicitamente ao concluir a implementação; sem disparo automático via CI/CD |
| Credenciais cross-repo | `api.societiza` já possui a configuração necessária para acesso ao GitHub com permissão cross-repo |
| Domínio da spec | Features de tooling interno não seguem a taxonomia de domínio de produto. O marcador `tooling` é usado para features de infraestrutura de desenvolvimento sem domínio de produto correspondente |
| Escopo de leitura do código | O agente lê apenas os arquivos diretamente relacionados à feature sendo documentada — não faz varredura ampla do repositório |

## 9. Perguntas abertas

Nenhuma. Todas as decisões de produto estão fechadas.