# Accountancy

A feature de accountancy representa a contabilidade dentro da plataforma. Ela
é o contorno organizacional do produto. O escritório de contabilidade que
compra e usa a Societiza. A accountancy guarda dados básicos do escritório
(nome legal, nome fantasia, CNPJ, telefone, e-mail) e é o ponto de partida
para a criação de times e para a separação de dados entre clientes diferentes.

A importância da accountancy vai além de ser apenas um cadastro. Ela é o
discriminante multi-tenant do sistema: convites, processos, templates e futuras
funcionalidades de branding partem do contexto da contabilidade. Mesmo quando
uma feature parecer focada em workflow, deve-se lembrar que a entidade da
contabilidade é o contorno organizacional do produto.

## Estado atual

O que está implementado hoje: criação, edição, listagem de contabilidades,
vinculação com processos no board via filtro.

## Gaps conhecidos

- Branding personalizado por contabilidade
- Configurações avançadas por escritório
- Multi-contabilidade gerenciada por um único admin