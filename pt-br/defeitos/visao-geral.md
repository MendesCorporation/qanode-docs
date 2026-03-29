# Módulo de Defeitos — Visão Geral

> **Disponível em:** QANode Enterprise

O módulo de Defeitos transforma uma falha de execução em um processo controlado de rastreamento. Em vez de uma execução com falha simplesmente ficar registrada no histórico, você pode abrir um defeito formal, atribuí-lo a uma pessoa ou equipe, acompanhar sua investigação e encerrá-lo com trilha completa de histórico.

---

## O que o módulo oferece

- **Abertura de defeito** a partir de uma execução com falha ou manualmente
- **Workflow configurável** — sua empresa define os status e as transições do processo
- **Filas e atribuição** — defeitos podem ser direcionados a equipes ou pessoas específicas
- **Claim** — um usuário assume o defeito para si antes de agir
- **Investigação em sandbox** — cópia descartável do fluxo original para investigar sem impactar execuções oficiais
- **Campos customizados** — campos adicionais definidos pela sua empresa
- **Comentários, anexos e histórico completo** — toda interação com o defeito fica registrada

---

## Como acessar

O módulo de Defeitos aparece no menu lateral com o ícone de **bug** (🐛). Ao clicar, você vê a lista de todos os defeitos da instância com filtros por status, severidade, prioridade, fila e responsável.

Para abrir um defeito diretamente de uma execução com falha, acesse o detalhe da execução e clique em **Abrir Defeito**.

---

## Permissões

O acesso a cada funcionalidade do módulo é controlado por permissões atribuídas ao papel do usuário. As permissões disponíveis são:

| Permissão | O que permite |
|-----------|--------------|
| `bug.view` | Ver defeitos, comentar e gerenciar os próprios anexos |
| `bug.create` | Abrir novos defeitos |
| `bug.edit` | Editar campos customizados do defeito |
| `bug.assign` | Atribuir defeito a pessoas/filas e fazer claim |
| `bug.transition` | Tramitar nas transições normais do workflow |
| `bug.transition_any` | Bypass administrativo — tramitar para qualquer status sem restrição de posse |
| `bug.run_sandbox` | Criar e descartar sandbox de investigação |
| `bug.delete_attachment_any` | Excluir anexos enviados por qualquer pessoa |
| `bug.configure_workflow` | Configurar o workflow, status, transições e campos customizados |

As permissões são configuradas em **Configurações → Papéis**.

---

## Conceitos fundamentais

### Workflow

O workflow define quais status existem, quais transições são possíveis entre eles e quais campos aparecem em cada etapa. Toda instância tem exatamente um workflow ativo. Veja [Workflow Builder](./workflow-builder.md).

### Status

Cada defeito está sempre em um status. O status inicial é definido no workflow e é onde o defeito nasce. Status finais indicam encerramento — quando o defeito chega a um status final, a data de fechamento é registrada automaticamente.

### Transição

Uma transição é o caminho entre dois status. Cada transição pode definir campos obrigatórios que precisam ser preenchidos na hora de tramitar.

### Fila

Uma fila representa uma equipe ou grupo responsável por um conjunto de defeitos. Quando um defeito está **apenas em fila** (sem responsável direto), qualquer membro daquela fila com a permissão adequada pode agir sobre ele. Se houver responsável direto, a posse desse usuário prevalece no fluxo normal.

### Claim

Claim é o ato de assumir o defeito. Ao dar claim, o defeito passa a ser seu. A tramitação normal exige que o defeito esteja claimado no usuário que vai tramitar.

### Sandbox

O sandbox é uma cópia descartável do fluxo original usada para investigar a falha. Alterações no sandbox não afetam o fluxo oficial nem contam nos KPIs e relatórios. Veja [Sandbox de Investigação](./sandbox.md).

---

## Navegação desta seção

| Página | O que você vai aprender |
|--------|------------------------|
| [Workflow Builder](./workflow-builder.md) | Como configurar status, transições, campos e custom fields |
| [Ciclo de Vida do Defeito](./ciclo-de-vida.md) | Como abrir, atribuir, claimar e tramitar defeitos |
| [Sandbox de Investigação](./sandbox.md) | Como investigar falhas sem impactar execuções oficiais |
| [Comentários, Anexos e Histórico](./comentarios-anexos.md) | Como colaborar e acompanhar o histórico de um defeito |
