# Contracts — Interface entre Repos

## O que é um contrato neste ecossistema

Um contrato é a definição formal da interface entre o backend (api.societiza)
e o frontend (web.societiza). Ele descreve endpoints, request/response shapes,
error envelopes, auth requirements e estados de negócio.

Os contratos vivem neste repo (orquestrador) porque são a única fonte de
verdade que ambos os repos de execução consomem.

---

## Formato

Contratos são definidos em OpenAPI YAML:

```
contracts/api/{feature-name}.yaml
```

Cada feature tem seu próprio arquivo de contrato.

---

## Ciclo de vida

```
draft -> review -> approved -> published
```

1. **Draft**: gerado pelo orquestrador após spec aprovada
2. **Review**: validado contra o backend real
3. **Approved**: humano confirma que o contrato está correto
4. **Published**: frontend e backend podem consumir

---

## Quem produz

O agente orquestrador (este repo) gera o contrato com input do backend.
O backend valida que o contrato reflete a implementação real.

## Quem consome

- **api.societiza**: implementa os endpoints conforme o contrato
- **web.societiza**: integra os endpoints conforme o contrato

---

## Versionamento

Contratos são versionados por URL path:

```
contracts/api/v1/{feature-name}.yaml
contracts/api/v2/{feature-name}.yaml
```

Quebras de contrato exigem nova versão. Adições retrocompatíveis podem ser
feitas na versão atual.
