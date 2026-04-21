---
status: approved
spec: accountancy-org
created_at: 2026-04-21
approved_at: 2026-04-21
---

# Accountancy Org — Lógica de Negócio

## 1. Fluxo por perfil

### SystemAdmin

O SystemAdmin é o único perfil com poder de escrita sobre contabilidades. Ele
opera a plataforma como back-office e é responsável por provisionar cada novo
cliente da Societiza.

**Listar contabilidades:**

1. Acessa a listagem de contabilidades cadastradas.
2. Vê a lista paginada, ordenada por nome legal. Cada item exibe nome legal e CNPJ formatado.
3. Pode navegar entre páginas. Não há busca nem filtros.

**Criar contabilidade:**

1. Inicia o formulário de criação.
2. Preenche os campos: CNPJ, nome legal, endereço, cidade, estado, CEP e telefone são obrigatórios. Nome fantasia e e-mail são opcionais.
3. Submete o formulário.
4. O sistema valida a estrutura do CNPJ via algoritmo local. Se inválido, a criação é bloqueada e o campo é sinalizado com a mensagem de erro correspondente.
5. O sistema verifica se o CNPJ já está cadastrado. Se já existir, a criação é bloqueada e o campo é sinalizado informando que o CNPJ já está em uso.
6. Se todas as validações passarem, a contabilidade é cadastrada e fica imediatamente habilitada a receber o link de convite para o primeiro AccountancyAdmin.
7. O sistema exibe confirmação e retorna à listagem.

**Consultar detalhes de uma contabilidade:**

1. Seleciona uma contabilidade na listagem.
2. Vê todos os dados cadastrais da firma: CNPJ, nome legal, nome fantasia, endereço, cidade, estado, CEP, telefone e e-mail. Campos opcionais não preenchidos não são exibidos.

**Editar contabilidade:**

1. A partir da tela de detalhes, inicia a edição.
2. Todos os campos do formulário ficam disponíveis para alteração.
3. Submete as alterações.
4. O sistema valida a estrutura do CNPJ. Se inválido, bloqueia e sinaliza.
5. O sistema verifica unicidade do CNPJ, excluindo a própria contabilidade da comparação. Se o CNPJ pertencer a outra contabilidade, bloqueia e sinaliza.
6. Se todas as validações passarem, os dados são atualizados.

**Ação proibida — excluir ou inativar contabilidade:**
Não existe essa operação. O sistema não expõe nenhum controle de exclusão ou
mudança de status. Se o SystemAdmin precisar encerrar um cliente, a operação é
tratada fora da plataforma.

---

### AccountancyAdmin

O AccountancyAdmin não cria nem edita a própria contabilidade. Seu ponto de
entrada na feature é a tela "Minha Contabilidade", que aparece logo no
primeiro login após aceitar o convite.

**Consultar "Minha Contabilidade":**

1. Faz login pela primeira vez via link de convite. O convite já o vincula à contabilidade correspondente.
2. Acessa "Minha Contabilidade".
3. Vê os dados cadastrais da sua firma: CNPJ formatado, nome legal, nome fantasia (se preenchido), endereço, cidade, estado, CEP formatado, telefone formatado e e-mail (se preenchido).
4. A tela é somente leitura. Não há nenhum controle de edição disponível.
5. Em acessos subsequentes, a tela continua disponível com os dados sempre atualizados conforme o SystemAdmin mantiver.

**Ação proibida — tentar editar dados cadastrais:**
O sistema não expõe nenhum controle de edição para este perfil. Se a tentativa
ocorrer via manipulação direta de requisição, o servidor rejeita a operação e
retorna erro de autorização.

---

### AccountancyEmployee

O AccountancyEmployee tem exatamente a mesma experiência de leitura que o
AccountancyAdmin em "Minha Contabilidade". Não há diferença de exibição entre
os dois perfis nesta feature.

**Consultar "Minha Contabilidade":**

1. Acessa "Minha Contabilidade".
2. Vê os mesmos dados cadastrais que o AccountancyAdmin vê: CNPJ formatado, nome legal, nome fantasia (se preenchido), endereço, cidade, estado, CEP formatado, telefone formatado e e-mail (se preenchido).
3. A tela é somente leitura. Não há nenhum controle de edição disponível.

**Ação proibida — tentar criar ou editar contabilidade:**
O servidor rejeita qualquer tentativa de escrita por este perfil e retorna
erro de autorização.

---

## 2. Estados e transições

### Contabilidade

A contabilidade tem um único estado operacional: **cadastrada**. Não existe
ciclo de vida. Não existem estados de ativa, inativa, suspensa ou arquivada.

| Estado | O que significa | O que causa a entrada |
|---|---|---|
| Cadastrada | A contabilidade existe na plataforma, está habilitada a receber convites e a operar | SystemAdmin conclui o formulário de criação com sucesso |

Não há saída deste estado. A contabilidade cadastrada permanece indefinidamente.

### Dados cadastrais

Os dados cadastrais não têm estados — eles simplesmente refletem o último
valor salvo pelo SystemAdmin. A única transição é de "dados anteriores" para
"dados atualizados", provocada por uma edição bem-sucedida.

---

## 3. Regras de negócio

- CNPJ, nome legal, endereço, cidade, estado, CEP e telefone são obrigatórios para criar ou editar uma contabilidade.
- Nome fantasia e e-mail são opcionais.
- CNPJ é único no sistema → tentativa de criar ou editar com CNPJ já pertencente a outra contabilidade é bloqueada.
- CNPJ é validado estruturalmente por algoritmo local antes de qualquer persistência → CNPJ inválido bloqueia a operação, sem acionar serviço externo.
- CNPJ, CEP e telefone são armazenados sem formatação (apenas dígitos) → a formatação para exibição é responsabilidade exclusiva da camada de apresentação.
- Apenas SystemAdmin pode criar e editar contabilidades → qualquer operação de escrita por AccountancyAdmin ou AccountancyEmployee é rejeitada no servidor.
- AccountancyAdmin e AccountancyEmployee têm acesso somente leitura aos dados cadastrais da sua própria contabilidade.
- Um usuário pertence a exatamente uma contabilidade → não há seleção de contabilidade na sessão.
- A contabilidade da sessão é determinada pelo `accountancyId` carregado pelo Identity no momento da autenticação → o frontend não escolhe nem troca a contabilidade ativa.
- Não existe exclusão de contabilidade → o sistema não expõe nenhum mecanismo de delete.
- Toda feature da plataforma opera dentro do escopo do `accountancyId` → a contabilidade é o discriminante multi-tenant de todos os dados.

---

## 4. Pontos de integração

```
Frontend precisa saber:
  - Os dados cadastrais completos da contabilidade da sessão atual, para
    renderizar "Minha Contabilidade" (AccountancyAdmin e AccountancyEmployee)
  - Quais campos são obrigatórios vs opcionais, para validação client-side
    no formulário de criação e edição (SystemAdmin)
  - O total de registros da listagem, para calcular paginação (SystemAdmin)
  - As regras de formatação de exibição: CNPJ → XX.XXX.XXX/XXXX-XX,
    CEP → XXXXX-XXX, telefone → (XX) XXXXX-XXXX ou (XX) XXXX-XXXX
  - Quais campos opcionais estão ausentes, para omiti-los da exibição sem
    renderizar valores nulos ou espaços em branco

Backend precisa garantir:
  - Que operações de criação e edição são rejeitadas para qualquer perfil
    que não seja SystemAdmin, independente do que o frontend envie
  - Que o CNPJ é validado estruturalmente antes de persistir
  - Que a unicidade do CNPJ é verificada antes de criar ou editar,
    excluindo a própria contabilidade na verificação de edição
  - Que CNPJ, CEP e telefone são persistidos sem formatação (apenas dígitos)
  - Que a listagem de SystemAdmin é paginada e ordenada por nome legal
  - Que os dados de "Minha Contabilidade" retornados correspondem
    exclusivamente à contabilidade do usuário autenticado na sessão
```

---

## 5. Casos de borda

**CNPJ duplicado na edição**
SystemAdmin edita uma contabilidade e tenta salvar o CNPJ de outra contabilidade
já cadastrada. A verificação de unicidade deve excluir a própria contabilidade
sendo editada — o CNPJ dela não pode ser interpretado como duplicata de si mesma.
Decisão: o backend executa a verificação excluindo o `accountancyId` atual da
comparação.

**Campos opcionais ausentes em "Minha Contabilidade"**
Nome fantasia ou e-mail não foram preenchidos pelo SystemAdmin. A tela de
"Minha Contabilidade" não deve exibir o rótulo do campo com valor vazio, nem
traços ou placeholders. Campos ausentes são simplesmente omitidos da exibição.
Decisão: o frontend omite o campo inteiro quando o valor não está presente na
resposta.

**Listagem vazia no SystemAdmin**
Nenhuma contabilidade cadastrada ainda. A tela de listagem exibe um estado
vazio sem erro e com indicação clara de que não há registros.
Decisão: o backend retorna lista vazia com total zero; o frontend renderiza
estado vazio.

**AccountancyAdmin faz login antes de a contabilidade existir**
Situação impossível pelo fluxo: o link de convite só pode ser gerado para
uma contabilidade já cadastrada, e o Identity só vincula o usuário ao
`accountancyId` presente no convite. Se ocorrer por inconsistência de dados,
o Identity não consegue estabelecer o contexto de sessão.
Decisão: este caso é falha de provisionamento, não caso de uso desta feature.
O tratamento pertence ao módulo Identity.

**Edição simultânea da mesma contabilidade por dois SystemAdmins**
Dois SystemAdmins editam a mesma contabilidade ao mesmo tempo. O último a
salvar prevalece.
Decisão: sem controle de concorrência otimista neste estágio. O último save
vence. Registrar nos casos de borda do integration.md se necessário.