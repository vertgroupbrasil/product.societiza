# Features — Specs de Produto

## O que é uma spec de feature

Uma spec é o documento de produto que define o que uma feature é, que problema
resolve, para quem, como se conecta com o produto existente e o que está fora
de escopo. Ela é gerada pela skill `feature-design`.

---

## Formato

Specs são salvas em markdown:

```
features/specs/{feature-name}.md
```

O nome do arquivo é o nome da feature em kebab-case, sem datas.

---

## Workflow

```
ideia -> spec -> aprovação humana -> contrato -> implementação
```

1. **Ideia**: alguém identifica um problema ou oportunidade
2. **Spec**: a skill `feature-design` gera a especificação de produto
3. **Aprovação**: humano revisa, ajusta e aprova a spec
4. **Contrato**: orquestrador gera o contrato OpenAPI em `contracts/api/`
5. **Implementação**: repos de execução recebem a spec + contrato

---

## Relação com contracts/

Uma spec aprovada gera um contrato. O contrato é a interface técnica; a spec
é a definição de produto. Ambos devem existir antes de iniciar a implementação.

---

## Specs não são

- Documentos técnicos (sem código, schemas, endpoints)
- Planos de implementação (esses ficam nos repos de execução)
- Documentos de arquitetura
