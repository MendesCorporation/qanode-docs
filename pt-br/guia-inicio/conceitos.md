# Conceitos Fundamentais

Antes de se aprofundar no QANode, é importante entender os conceitos-chave que formam a base da plataforma.

---

## Visão Geral da Hierarquia

```
Projetos
├── Cenários (Fluxos)
│   ├── Nós (Nodes)
│   │   ├── Configuração
│   │   ├── Conexões (Edges)
│   │   └── Outputs/Inputs
│   └── Execuções (Runs)
│       ├── Passos (Steps)
│       ├── Logs
│       ├── Screenshots
│       └── Relatório PDF
└── Suítes
    ├── Cenários ordenados
    ├── Agendamento (Cron)
    └── Execuções de Suíte
```

---

## Projetos

Um **projeto** é o nível mais alto de organização. Ele agrupa cenários de teste e suítes relacionados a um sistema, funcionalidade ou equipe.

Cada projeto possui:
- **Nome** e **descrição**
- **Status**: Ativo, Concluído, Cancelado ou Arquivado
- **Datas**: Início e término planejados
- **Documentação**: Arquivos anexados (requisitos, especificações, etc.)
- **Estatísticas**: Número de cenários, suítes e execuções recentes

---

## Cenários (Fluxos)

Um **cenário** (também chamado de **fluxo**) é um caso de teste representado visualmente como um grafo de nós conectados. Cada cenário descreve uma sequência de ações a serem executadas — como navegar em um site, chamar uma API ou consultar um banco de dados.

Cenários são compostos por:
- **Nós (Nodes)**: Ações individuais (clicar, digitar, fazer requisição, etc.)
- **Conexões (Edges)**: Ligações entre nós que definem a ordem de execução
- **Configurações**: Parâmetros de cada nó

### Exemplo Visual

```
[Navigate] → [Fill email] → [Fill senha] → [Click login] → [Assert welcome]
```

Neste exemplo, cada caixa é um nó e as setas representam conexões indicando o fluxo de execução.

---

## Nós (Nodes)

**Nós** são os blocos fundamentais de um cenário. Cada nó executa uma ação específica e pode produzir outputs que são consumidos por nós seguintes.

### Categorias de Nós

| Categoria | Cor | Nós |
|-----------|-----|-----|
| **Controle de Fluxo** | 🟡 Amarelo | If, Switch, Loop, Merge |
| **Web** | 🔵 Azul | Web Flow, Smart Locators |
| **API** | 🟣 Roxo | HTTP Request |
| **Banco de Dados** | 🟢 Verde | PostgreSQL, MySQL, MariaDB, Oracle, MongoDB |
| **Infraestrutura** | 🔵 Azul Claro | SSH Command |
| **Utilitários** | ⚪ Cinza | Set Variable, Log, Wait, Stop and Fail, Custom JavaScript |
| **Nós Customizados** | 🩷 Rosa | Nós de provedores externos |

### Anatomia de um Nó

Cada nó possui:

- **Handles de Entrada** (●): Pontos onde conexões de outros nós chegam
- **Handles de Saída** (●): Pontos de onde conexões saem para outros nós
- **Configuração**: Campos específicos do tipo de nó
- **Outputs**: Dados produzidos após a execução (acessíveis via expressões)
- **Label**: Nome identificador do nó no fluxo

### Outputs e o Sistema de Expressões

Quando um nó é executado, ele produz **outputs** — dados que podem ser usados por nós seguintes. Para acessar esses dados, use o sistema de **expressões** com a sintaxe `{{ }}`:

```
{{ steps["Nome do Nó"].outputs.propriedade }}
```

**Exemplos:**
```
{{ steps["http-request"].outputs.json.token }}    → Token de uma resposta API
{{ steps.login.outputs.extracts.userName }}   → Texto extraído de uma página
{{ steps.query.outputs.rows[0].email }}       → Primeiro email do resultado SQL
```

> Para detalhes completos, veja a documentação de [Expressões](../expressoes/expressoes.md).

---

## Conexões (Edges)

**Conexões** ligam a saída de um nó à entrada de outro, definindo a ordem de execução. O QANode executa os nós em **ordem topológica** — ou seja, um nó só é executado depois que todos os nós conectados à sua entrada foram concluídos.

### Conexões Condicionais

Alguns nós possuem múltiplas saídas condicionais:

- **If** → saídas `true` e `false`
- **Switch** → saídas para cada case + `default`
- **Loop** → saídas `loop` (próxima iteração) e `done` (fim do loop)

Apenas o caminho correspondente à condição avaliada será executado. Nós em caminhos não ativados serão **pulados**.

---

## Execuções (Runs)

Uma **execução** é uma instância de um cenário sendo processado. Ao executar um cenário, o QANode:

1. Resolve a ordem topológica dos nós
2. Executa cada nó sequencialmente
3. Avalia expressões e injeta dados dos outputs anteriores
4. Registra logs, screenshots e artefatos
5. Gera um relatório PDF ao final

Cada execução possui:
- **Status**: `running`, `success` ou `failed`
- **Duração**: Tempo total de execução
- **Passos**: Resultado individual de cada nó
- **Artefatos**: Screenshots, PDFs e outros arquivos gerados
- **Logs**: Mensagens detalhadas de cada passo

---

## Suítes de Teste

Uma **suíte** agrupa vários cenários para execução sequencial. Suítes são úteis para:

- Executar múltiplos cenários em ordem
- Agendar execuções periódicas (diárias, horárias, etc.)
- Parar na primeira falha (opcional)

### Agendamento

Suítes podem ser agendadas usando **expressões cron**:

| Expressão | Significado |
|-----------|-------------|
| `0 8 * * *` | Todos os dias às 8h |
| `0 */2 * * *` | A cada 2 horas |
| `0 9 * * 1-5` | Segunda a sexta às 9h |
| `30 18 * * 5` | Sexta-feira às 18:30 |

---

## Variáveis

**Variáveis** são valores globais reutilizáveis em qualquer cenário. Tipos suportados:

| Tipo | Uso |
|------|-----|
| `string` | Textos (URLs, nomes, mensagens) |
| `number` | Números (timeouts, limites) |
| `boolean` | Flags (true/false) |
| `json` | Objetos e arrays complexos |
| `secret` | Valores criptografados (senhas, tokens) |

Acesse variáveis nos seus fluxos com: `{{ variables.nomeVariavel }}`

---

## Credenciais

**Credenciais** armazenam informações de conexão de forma segura e centralizada. Tipos suportados:

- **HTTP/API** — URL base, autenticação, headers
- **PostgreSQL**, **MySQL**, **MariaDB**, **Oracle** — Dados de conexão ao banco
- **MongoDB** — URI ou dados de conexão
- **SSH** — Host, usuário, senha ou chave privada

Credenciais podem ser referenciadas por nós que precisam de conexão, evitando que senhas fiquem expostas nos fluxos.


---

## Próximos Passos

Agora que você conhece os conceitos fundamentais:

- [Editor de Fluxos](../editor-fluxos/visao-geral.md) — Aprenda a usar o editor visual
- [Referência de Nós](../nos/controle-fluxo/if.md) — Conheça todos os nós em detalhes
- [Expressões](../expressoes/expressoes.md) — Domine o sistema de interpolação
