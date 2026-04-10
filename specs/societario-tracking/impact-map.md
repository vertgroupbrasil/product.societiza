# Mapa de Impacto — Rastreamento do Processo Societário

> Feature: societario-tracking
> Spec: specs/societario-tracking/spec.md
> Contrato: specs/societario-tracking/integration.md (gerado após implementação do backend)
> Data: 2026-04-07

---

## Visão geral do impacto

Esta feature toca dois repos de execução com responsabilidades distintas e
sem dependência entre si — backend pode ser implementado e validado antes do
frontend iniciar.

| Repo | Impacto | Pode iniciar sem o outro? |
|---|---|---|
| api.societiza | Alto — novo aggregate, dois endpoints, geração de token | Sim |
| web.societiza | Médio — nova rota pública, novo componente, novo elemento na sheet | Sim, após contrato publicado |

---

## api.societiza

### O que precisa ser construído

#### 1. Campo `trackingToken` no aggregate `WorkflowProcess`

Na criação de qualquer novo processo, o backend deve gerar automaticamente um
`trackingToken` único e opaco (sugestão: prefixo `tk_` + UUID v4 ou string
aleatória de 12+ caracteres). Esse token é imutável após a criação.

- Adicionar campo `trackingToken` ao aggregate `WorkflowProcess`
- Gerar o token no momento da criação do processo
- Persistir o token junto ao processo
- O token não deve ser derivado do `processId` — deve ser independente e opaco

#### 2. Lógica de expiração

O backend deve calcular se o link está expirado no momento da requisição:
- Processo em andamento (`status != completed`): link ativo
- Processo concluído há 30 dias ou menos: link ativo
- Processo concluído há mais de 30 dias: retornar `410 Gone`

O campo de data de conclusão (`completedAt` ou equivalente) precisa existir ou
ser introduzido no aggregate.

#### 3. Endpoint público — `GET /track/{trackingToken}`

Sem autenticação. Deve retornar:
- `processName`: nome da empresa em abertura
- `status`: `in_progress` ou `completed`
- `steps[]`: sequência ordenada de etapas com `order`, `name` e `state`
  (`completed`, `active`, `pending`)

O estado de cada etapa é derivado da posição atual do processo:
- Etapas anteriores à etapa atual → `completed`
- Etapa atual → `active`
- Etapas posteriores → `pending`

**Isolamento obrigatório:** este endpoint não pode expor tarefas, campos,
dados de outros processos, nem informações da contabilidade além do nome
do processo.

#### 4. Endpoint autenticado — `GET /processes/{processId}/tracking-link`

Requer autenticação. Deve validar que o processo pertence à contabilidade
(`accountancyId`) do usuário autenticado antes de retornar a URL.

Retorna:
- `trackingUrl`: URL pública completa no formato `https://societiza.com.br/rastreio/{trackingToken}`

#### 5. Notificação por e-mail (implementação futura — não bloqueia o MVP desta feature)

Quando o processo avançar de etapa, disparar e-mail ao cliente com identidade
visual da contabilidade (nome, logo, remetente). Aguarda campo de e-mail do
cliente no processo.

---

### Checklist api.societiza

- [ ] Adicionar `trackingToken` ao aggregate `WorkflowProcess`
- [ ] Gerar token automaticamente na criação do processo
- [ ] Adicionar campo `completedAt` ao processo (se ainda não existir)
- [ ] Implementar lógica de expiração (30 dias após `completedAt`)
- [ ] Implementar `GET /track/{trackingToken}` (público, sem auth)
- [ ] Implementar `GET /processes/{processId}/tracking-link` (autenticado)
- [ ] Validar isolamento de dados no endpoint público
- [ ] Validar que `/tracking-link` respeita `accountancyId` do usuário autenticado
- [ ] Cobrir retornos 404 e 410 no endpoint público

---

## web.societiza

### O que precisa ser construído

#### 1. Rota pública `/rastreio/[trackingToken]`

Nova rota sem autenticação, fora do layout protegido da aplicação. O cliente
acessa essa URL diretamente pelo link compartilhado pelo contador.

A página consome `GET /track/{trackingToken}` e renderiza:
- Nome do processo
- Kanban horizontal (ou vertical em mobile) com as etapas em sequência
- Estado visual de cada etapa: concluída, ativa, pendente
- Estado de expiração (410) com mensagem adequada
- Estado de não encontrado (404) com mensagem adequada

A identidade visual da página deve ser neutra ou da contabilidade — a Societiza
não deve aparecer para o cliente.

#### 2. Elemento "Link de rastreamento" na sheet do processo

Na sheet lateral que o contador usa para operar e avançar o processo, adicionar
uma seção ou elemento que:
- Exibe a URL de rastreamento (truncada visualmente)
- Oferece botão "Copiar link"
- Consome `GET /processes/{processId}/tracking-link`

#### 3. Schema e hook para o contrato de rastreamento

Criar o schema TypeScript e o hook de fetch correspondente ao contrato
`GET /track/{trackingToken}`, seguindo o padrão já adotado no projeto.

---

### Checklist web.societiza

- [ ] Criar rota pública `/rastreio/[trackingToken]` fora do layout autenticado
- [ ] Criar componente de kanban de rastreamento (etapas: completed / active / pending)
- [ ] Tratar estado 404 com mensagem adequada ao cliente
- [ ] Tratar estado 410 (link expirado) com mensagem adequada ao cliente
- [ ] Adicionar elemento de link de rastreamento na sheet do processo
- [ ] Implementar botão "Copiar link" na sheet
- [ ] Criar schema e hook para `GET /track/{trackingToken}`
- [ ] Criar schema e hook para `GET /processes/{processId}/tracking-link`
- [ ] Garantir que a rota pública não herda o middleware de autenticação

---

## Breaking changes e riscos de integração

| Risco | Severidade | Mitigação |
|---|---|---|
| Token gerado retroativamente em processos existentes | Médio | Decidir se processos já criados recebem token. Recomendação: sim, via migration. |
| Exposição acidental de dados internos no endpoint público | Alto | Teste explícito de isolamento — garantir que tarefas, campos e dados de outros clientes não aparecem |
| Endpoint público sem rate limiting | Médio | Adicionar rate limiting por token para evitar abuso |
| Rota pública do frontend herdando middleware de auth | Médio | Verificar configuração do middleware no Next.js para excluir `/rastreio/*` |
| `completedAt` ausente no aggregate atual | Baixo | Verificar existência do campo antes de implementar lógica de expiração |

---

## Ordem sugerida de implementação

```
1. api.societiza
   ├── Adicionar trackingToken e completedAt ao WorkflowProcess
   ├── Implementar GET /track/{trackingToken}
   └── Implementar GET /processes/{processId}/tracking-link

2. web.societiza (após endpoints disponíveis em staging)
   ├── Rota pública /rastreio/[trackingToken] + kanban
   └── Elemento de link na sheet do processo
```

Backend e frontend podem ser iniciados em paralelo se o time trabalhar contra
o contrato documentado em `specs/societario-tracking/integration.md` (gerado após implementação do backend).
O frontend pode mockar os endpoints enquanto o backend não estiver pronto.
