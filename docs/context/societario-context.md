# Societário — Feature Central

## O societário como núcleo do produto

O societário é o núcleo do produto. Tudo na Societiza orbita em torno do fluxo
de abertura de empresa. O societário é um quadro kanban sem drag-and-drop no
uso operacional do workflow, com colunas ordenadas que representam as etapas
pelas quais um processo passa desde o início até a conclusão.

Cada processo representa uma abertura de empresa ou um caso semelhante dentro
da contabilidade. O processo contém dados básicos da empresa sendo aberta,
vinculação com um escritório de contabilidade (accountancy), e uma série de
tarefas configuradas pelo template do workflow. Dentro do processo, existem
tarefas e campos configuráveis que precisam ser suficientemente claros para que
qualquer membro da equipe entenda em que ponto o processo está, o que falta, o 
que está travado e o que precisa ser feito em seguida.

Do ponto de vista de experiência do usuário, o societário deve transmitir
controle, previsibilidade e continuidade. O contador não deve sentir que está
apenas preenchendo software. Ele deve sentir que está tocando a operação real
da contabilidade em um único lugar. O board precisa mostrar rapidamente o
estado de cada processo, os responsáveis, as pendências e os gargalos.

O societário se conecta diretamente com o workflow template (que define as
etapas e tarefas), com a accountancy (que identifica a qual escritório o
processo pertence), e será a fonte primária de dados para o dashboard e para
futuras features de notificação.

---
 
## WorkflowTemplate e WorkflowProcess
 
O societário é composto por duas camadas complementares: o **WorkflowTemplate**,
que define a estrutura do fluxo, e o **WorkflowProcess**, que representa a
execução real de um processo a partir dessa estrutura.
 
### WorkflowProcess — o board operacional
 
O WorkflowProcess é a camada onde o usuário acompanha e opera processos. O
board é um kanban com colunas que representam as etapas do workflow. Cada card
é um processo de abertura de empresa — uma instância criada a partir de uma
versão publicada de um WorkflowTemplate, contendo as instâncias das etapas e
tarefas daquele template.
 
A experiência esperada é de visão operacional imediata: ao abrir o board, o
contador vê exatamente onde cada processo está, o que está parado, o que
precisa de atenção e o que está avançando. A ausência de drag-and-drop é uma
decisão atual; a movimentação de etapa acontece de forma explícita.
 
### WorkflowTemplate — a estrutura do fluxo
 
O WorkflowTemplate define as etapas, tarefas e campos que compõem um tipo de
processo societário. É a partir de um template ativo que os processos são
criados.
 
No MVP, os templates disponíveis para as contabilidades são fornecidos pela
própria Societiza — não existe ainda uma interface para que a contabilidade
crie ou edite seus próprios templates. Essa decisão foi tomada para garantir
qualidade e consistência nos primeiros fluxos entregues. Permitir que a
contabilidade personalize seus próprios templates é algo que está sendo
mapeado para implementação no MVP e será documentado quando consolidado.
 
O template é composto por steps, tasks e fields:
 
#### Steps (etapas)
 
Cada etapa tem um título, uma descrição e uma ordem. A ordem das etapas carrega
semântica operacional — ela representa a sequência real do fluxo de trabalho.
Cada etapa pode ter campos próprios (StepFields) que se aplicam à etapa como
um todo, não a uma tarefa específica.
 
#### StepFields (campos de etapa)
 
Campos configuráveis vinculados a uma etapa. Têm label, tipo e, quando do tipo
Select, uma lista de opções configurável. Os tipos de campo suportados são:
Text, Boolean, Integer, Select e Date.
 
#### Tasks (tarefas)
 
Cada etapa pode ter múltiplas tarefas. As tarefas têm título, descrição, tipo,
indicação de opcionalidade e uma ordem dentro da etapa. Os tipos de tarefa
existentes são: Manual, Automatic e Approval.
 
#### TaskFields (campos de tarefa)
 
Campos configuráveis vinculados a uma tarefa específica. Seguem a mesma
estrutura dos StepFields: label, tipo e lista de opções quando Select. A lista
de opções é armazenada como JSON string e parseada na camada de serviço do
frontend.
 
A ordem de etapas, tarefas e campos não é decorativa — carrega semântica
operacional e deve ser respeitada em qualquer implementação.
 
### Status do template
 
Um template pode estar em um de três estados: **Draft**, **Active** ou
**Archived**.
 
O estado Draft representa um template em construção, ainda não disponível para
uso. O estado Active representa o fluxo publicado e em uso real — apenas
templates ativos podem ser usados para criar processos. O estado Archived
representa um template descontinuado, mantido apenas para referência histórica.
 
A versão do template (ex: `v1.0`) representa a versão publicada e aprovada.
Ela não muda automaticamente ao editar — uma mudança de versão é sempre uma
decisão explícita.

### Campos dentro de tarefas e etapas — atenção ao contexto

Hoje existe no produto a ideia de inputs de texto, caixas de seleção e outros
campos configuráveis dentro de etapas e tarefas do template do workflow. Esses
campos são o mecanismo atual para capturar e estruturar parte das informações
operacionais necessárias durante a execução do processo.

**O produto ainda não possui, como feature consolidada de MVP, um formulário
nativo completo de abertura de empresa como módulo principal e oficial do
produto.** O que existe hoje, do ponto de vista de regra de negócio
consolidada, são campos configuráveis dentro do workflow-template. A direção
futura é ter um formulário nativo com branding da contabilidade onde o cliente
preenche os dados que alimentam o processo, mas isso ainda não existe como
capacidade entregue. Não trate a feature futura de formulário como se já fosse
uma capacidade principal do MVP.