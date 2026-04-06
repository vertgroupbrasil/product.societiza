# Documentação

## Objetivo

A documentação da feature precisa permitir que outra pessoa continue o trabalho
sem adivinhar:

- o domínio;
- o fluxo de dados;
- a regra de negócio;
- as integrações;
- e as decisões arquiteturais relevantes.

O projeto usa dois níveis principais de documentação, com propósitos diferentes.

---

## Nível 1: `src/features/{feature}/README.md`

Este é o documento vivo da feature.

Ele deve explicar:

- o que a feature representa no produto;
- qual é o fluxo de dados;
- quais são as entidades principais;
- quais regras de negócio essa feature implementa;
- como as pastas da feature se distribuem.

### Quando atualizar

- ao criar uma feature nova;
- ao fazer mudança importante de arquitetura interna;
- ao alterar regra de negócio central;
- ao adicionar integração relevante com outra feature.

### Estrutura recomendada

```md
# {Feature}

## Domínio

## Entidades principais

## Fluxo de dados

## Regras de negócio

## Responsabilidades de cada camada

## Integrações externas

## Decisões de arquitetura
```

### Regra

README da feature precisa refletir o domínio real daquela pasta. Não copiar README
de outra feature e fazer substituição superficial.

---

## Nível 2: `docs/features/tech/...`

Esse é o histórico técnico de mudanças relevantes.

Ele não substitui o README. Ele registra:

- o que mudou;
- por que mudou;
- impacto estrutural;
- riscos;
- validação executada.

### Quando criar ou atualizar

- rework estrutural;
- feature nova;
- mudança de integração;
- decisão arquitetural não óbvia;
- alteração importante de fluxo.

### Estrutura recomendada

```md
# {Feature} — {Mudança}

## O que mudou

## Por que mudou

## Impacto arquitetural

## Fluxo afetado

## Pontos de atenção

## Validação executada
```

---

## `docs/integration/...`

Quando houver integração backend/frontend relevante, a documentação técnica de
contrato deve viver em `docs/integration/`.

Esse tipo de documento é especialmente importante quando:

- backend entrega uma feature nova;
- o frontend precisa integrar contrato já pronto;
- há semântica de erro, auth, estados ou edge cases que o frontend não deve
  descobrir por tentativa e erro.

### Regra

Frontend consome essa documentação e também pode atualizá-la quando descobrir
ajustes necessários para a integração.

---

## O que uma documentação boa precisa responder

Qualquer doc relevante de feature deve ajudar a responder:

- o que essa feature resolve no produto;
- quais entidades ela manipula;
- como o dado entra e chega à UI;
- que regras de negócio importam;
- com que outras features ela se conecta;
- que limitações e riscos já são conhecidos.

Se o documento não ajuda nessas perguntas, ele está incompleto.

---

## Relação entre README e docs técnicas

### README

É o mapa vivo da feature.

### `docs/features/tech`

É o histórico técnico da mudança.

### `docs/integration`

É o contrato de integração entre camadas/equipes.

### Regra

Os três documentos não devem repetir a mesma coisa do mesmo jeito. Cada um tem
um papel diferente.

---

## O que não documentar de forma falsa

O ruleset de documentação não pode inventar regra de negócio nem contrato de
backend.

Exemplos de erro documental:

- afirmar que uma operação é hard delete quando o backend atual faz soft delete;
- afirmar relação de dependência entre features sem verificar o fluxo real;
- descrever uma feature futura como se já estivesse entregue;
- escrever README genérico que não conversa com o código da pasta.

---

## Padrão mínimo para feature relevante

Uma feature relevante deve sair com:

- `src/features/{feature}/README.md`
- `docs/features/tech/...`

E, quando houver integração com backend:

- `docs/integration/...`

---

## Como documentar mudanças arquiteturais

Se a mudança mexeu em:

- endpoints;
- schemas;
- services;
- hooks;
- barrel strategy;
- fluxo entre UI e orchestration;
- contrato de integração;

isso precisa aparecer de forma explícita na documentação técnica.

---

## Validação na documentação

Documentação técnica relevante deve registrar:

- o que foi testado;
- como foi testado;
- o que passou;
- o que ficou pendente;
- o que é risco conhecido.

Isso ajuda produto, frontend, backend e revisão a compartilharem a mesma
leitura do estado da feature.

---

## Resumo operacional

- README da feature explica o domínio vivo;
- `docs/features/tech` registra mudança estrutural;
- `docs/integration` documenta contrato backend/frontend;
- documentação não pode inventar comportamento;
- toda feature relevante precisa ser explicável sem depender de memória tácita.
