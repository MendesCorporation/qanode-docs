# Projetos

**Projetos** são a unidade principal de organização no QANode. Cada projeto agrupa cenários de teste (fluxos), suítes, documentação e execuções relacionados a um sistema, funcionalidade ou equipe.

---

## Criando um Projeto

1. No menu lateral, clique em **Projetos**
2. Clique em **+ Novo Projeto**
3. Preencha os campos:

| Campo | Obrigatório | Descrição |
|-------|-------------|-----------|
| **Nome** | Sim | Nome do projeto |
| **Descrição** | Não | Descrição detalhada |
| **Status** | Sim | Status atual do projeto |
| **Data de Início** | Não | Data planejada de início |
| **Data de Término** | Não | Data planejada de conclusão |

4. Clique em **Criar**

<!-- ![Criar projeto](../../assets/images/criar-projeto.png) -->
*Imagem: Modal de criação de projeto com todos os campos preenchidos*

---

## Status do Projeto

| Status | Descrição |
|--------|-----------|
| **Ativo** | Projeto em andamento |
| **Concluído** | Projeto finalizado com sucesso |
| **Cancelado** | Projeto cancelado |
| **Arquivado** | Projeto arquivado (não aparece na lista principal) |

Para alterar o status, acesse o projeto e use as opções no cabeçalho:
- **Concluir** → Marca como concluído
- **Cancelar** → Marca como cancelado
- **Arquivar** → Move para arquivados

---

## Página do Projeto

A página de detalhe do projeto possui as seguintes abas:

### Visão Geral

Exibe estatísticas e resumo do projeto:

- **Total de Cenários** — Quantidade de fluxos de teste
- **Total de Suítes** — Quantidade de suítes
- **Cenários Pendentes/Aprovados/Reprovados** — Status dos cenários
- **Execuções Recentes** — Lista das últimas execuções
- **Progresso** — Comparação entre tempo decorrido e cenários executados

<!-- ![Visão geral do projeto](../../assets/images/projeto-visao-geral.png) -->
*Imagem: Página de visão geral mostrando cards de estatísticas, execuções recentes e barra de progresso*

### Cenários

Lista todos os cenários (fluxos) do projeto:

| Informação | Descrição |
|------------|-----------|
| **Nome** | Nome do cenário |
| **Status** | Último resultado (pendente, sucesso, falha) |
| **Data de Criação** | Quando foi criado |

**Ações:**
- **+ Novo Cenário** — Cria um novo fluxo e abre o editor
- **Executar** — Executa o cenário rapidamente
- **Editar** — Abre o editor de fluxos
- **Excluir** — Remove o cenário

No QANode Enterprise, esses cenários também podem ser executados por pipeline com o `@qanode/cli` usando:

- **ID**
- **nome do cenário + nome do projeto**

> Para detalhes, veja [CLI e API do CI/CD](../ci-cd/cli-api.md).

### Suítes

Lista as suítes do projeto com informações de agendamento e último resultado.

> Para detalhes sobre suítes, veja [Suítes de Teste](../suites/suites.md).

### Execuções

Histórico de todas as execuções do projeto:

| Informação | Descrição |
|------------|-----------|
| **Cenário/Suíte** | Nome do que foi executado |
| **Status** | Resultado (sucesso, falha, executando) |
| **Duração** | Tempo de execução |
| **Data** | Data e hora da execução |

### Defeitos — Enterprise

Lista os defeitos vinculados ao projeto. Para cada cenário da lista é possível ver se ele está **bloqueado por um defeito ativo**, com link direto para o defeito responsável pelo bloqueio.

**Ações disponíveis na lista de cenários:**

| Ação | Descrição |
|------|-----------|
| **Bloquear por Defeito** | Associa um defeito existente como bloqueador do cenário |
| **Desbloquear** | Remove a associação de bloqueio |

Um cenário bloqueado por defeito aparece com indicação visual na lista e é contabilizado na métrica **Cenários Bloqueados** dos relatórios.

> Para abrir um novo defeito, use a execução com falha do cenário ou acesse diretamente a aba **Defeitos** no menu lateral.

### Documentação

Gerenciamento de arquivos do projeto:

- **Upload** — Faça upload de documentos (requisitos, especificações, etc.)
- **Download** — Baixe documentos anexados
- **Excluir** — Remova documentos

---

## Filtragem e Busca

Na lista de projetos, você pode:

- **Buscar por nome** — Campo de busca no topo
- **Filtrar por status** — Dropdown de filtro (Ativo, Concluído, etc.)

---

## Duplicar Projeto

Para criar uma cópia de um projeto existente:

1. Na lista de projetos, clique nas opções (⋮) do projeto
2. Selecione **Duplicar**
3. Um novo projeto será criado com os mesmos cenários

---

## Dicas

- Use **status** para acompanhar o ciclo de vida dos projetos
- **Arquive** projetos antigos para manter a lista limpa
- **Datas de início e término** ajudam a acompanhar o progresso vs. planejamento
- Use a aba de **Documentação** para manter requisitos e especificações junto ao projeto
