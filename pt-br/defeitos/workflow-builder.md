# Workflow Builder

> **Permissão necessária:** `bug.configure_workflow`

O Workflow Builder é a tela onde sua empresa define como os defeitos vão fluir — quais status existem, quais transições são possíveis entre eles, quais campos aparecem em cada etapa e quais campos customizados o módulo vai ter.

Acesse em **Configurações → Workflow de Defeitos**.

---

## Visão geral da tela

A tela é dividida em duas áreas principais:

- **Status** — lista de todos os status do workflow com seus campos e configurações
- **Transições** — lista de todas as transições possíveis entre os status

Toda alteração precisa ser salva com o botão **Salvar Workflow** para entrar em vigor.

---

## Gerenciando Status

### Criar um status

1. Clique em **Adicionar Status**
2. Defina:
   - **Nome** — nome exibido na interface (ex: "Em Triagem", "Em Investigação")
   - **Status Inicial** — marque se este é o status onde todo defeito nasce ao ser aberto (apenas um pode ser inicial)
   - **Status Final** — marque se este status representa encerramento do defeito (pode haver mais de um)
   - **Fila Padrão** — se informada, ela é usada como fila de destino na ausência de uma escolha manual de roteamento
   - **Permite atribuição manual** — se ativado, o usuário pode escolher livremente o responsável direto ao tramitar para este status

### Editar um status

Clique no nome do status ou no ícone de edição ao lado dele. As mesmas configurações de criação ficam disponíveis para edição.

### Remover um status

Clique no ícone de exclusão ao lado do status.

> **Atenção:** Não é possível remover um status que ainda possui defeitos abertos associados a ele. Mova ou encerre esses defeitos antes de remover o status.

### Reordenar status

Arraste os status pela alça lateral para reorganizar a ordem em que aparecem na interface.

---

## Campos do status

O status inicial define os campos que aparecem na abertura do defeito. Já os formulários de tramitação são definidos nas transições.

Os campos nativos disponíveis são:

| Campo | Descrição |
|-------|-----------|
| Título | Título do defeito |
| Descrição | Descrição detalhada |
| Severidade | Nível de impacto (crítico, alto, médio, baixo) |
| Prioridade | Urgência de resolução |
| Resolução | Resultado do encerramento (ex: "Corrigido", "Não reproduzível") |
| Responsável | Usuário atribuído ao defeito |
| Fila | Equipe responsável pelo defeito |

Além dos campos nativos, todos os **campos customizados ativos** também ficam disponíveis.

### Visível vs. Obrigatório

Para cada campo você pode definir:

- **Visível** — o campo aparece na interface mas não precisa ser preenchido
- **Obrigatório** — o campo precisa ser preenchido para a operação ser concluída

> **Nota:** Um campo obrigatório sempre aparece na interface, mesmo que não esteja marcado como visível.

---

## Gerenciando Transições

Uma transição define o caminho de um status para outro. Sem uma transição criada, não é possível mover um defeito entre dois status no fluxo normal.

### Criar uma transição

1. Clique em **Adicionar Transição**
2. Defina:
   - **Nome** — label exibida no botão de tramitação (ex: "Iniciar Investigação", "Corrigir", "Rejeitar")
   - **Status de Origem** — status atual do defeito
   - **Status de Destino** — status para onde o defeito vai ao tramitar

### Campos da transição

Assim como nos status, cada transição pode ter seus próprios campos visíveis e obrigatórios. Esses campos aparecem no modal de tramitação quando o usuário clica no botão da transição.

Use campos de transição para capturar informações relevantes no momento da mudança — por exemplo, exigir **Resolução** ao fechar, ou **Fila** ao passar o defeito para outra equipe.

### Remover uma transição

Clique no ícone de exclusão ao lado da transição. Remover uma transição não afeta os defeitos já existentes — apenas impede que novos defeitos usem esse caminho.

---

## Workflow padrão

Quando não existe nenhum workflow ativo na instância, o QANode cria automaticamente um workflow padrão com os seguintes status:

| Status | Tipo |
|--------|------|
| Aberto | Inicial |
| Em Andamento | — |
| Corrigido | — |
| Rejeitado | — |
| Reaberto | — |
| Fechado | Final |

Você pode usar este workflow como ponto de partida e ajustá-lo conforme o processo da sua empresa.

> **Importante:** O workflow padrão só é criado quando não existe nenhum workflow ativo. Se você já tem um workflow configurado, o padrão nunca vai sobrescrever o seu.

---

## Campos Customizados

Os campos customizados permitem que sua empresa adicione informações específicas ao processo de defeitos — como número de ticket externo, sistema afetado, versão do build, entre outros.

Acesse na própria tela do **Workflow Builder**, na seção **Campos Customizados**.

### Criar um campo customizado

1. Clique em **Adicionar Campo**
2. Defina:
   - **Nome** — identificador interno do campo (sem espaços, ex: `ticket_externo`)
   - **Rótulo** — nome exibido na interface (ex: "Ticket Jira")
   - **Tipo** — tipo do valor (texto, número, data, seleção)
   - **Ativo** — se o campo está disponível para uso

### Usando campos customizados no workflow

Após criar um campo customizado e deixá-lo ativo, ele fica disponível para ser adicionado como visível ou obrigatório em qualquer status ou transição do workflow.

### Desativar vs. excluir

- **Desativar** — o campo deixa de aparecer na interface mas os dados já salvos são preservados
- **Excluir** — o campo é removido permanentemente

> **Atenção:** Não é possível desativar ou excluir um campo que esteja em uso por um workflow ativo. Remova o campo do workflow antes de desativá-lo.

---

## Boas práticas

- **Comece simples** — um workflow com 4 ou 5 status e transições diretas é mais fácil de operar do que um com muitos desvios
- **Use fila padrão quando quiser direcionar a etapa para uma equipe específica** — isso ajuda a sugerir a fila correta quando a transição não enviar outra fila manualmente
- **Exija resolução apenas no fechamento** — campos como "Resolução" fazem mais sentido como obrigatórios na transição de encerramento
- **Evite status órfãos** — todo status (exceto os finais) deve ter ao menos uma transição de saída, caso contrário defeitos podem ficar presos
