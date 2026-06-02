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
│       └── Artefatos e arquivos
├── Componentes
│   ├── Contrato de Entrada
│   ├── Fluxo Reutilizável
│   └── Contrato de Saída
└── Suítes
    ├── Cenários ordenados
    ├── Agendamento (Cron)
    └── Execuções de Suíte
```

---

## Projetos

[Projetos](../../assets/images/conceitos-projetos.mp4)

Um **projeto** é o nível mais alto de organização. Ele agrupa cenários de teste e suítes relacionados a um sistema, funcionalidade ou equipe.

Cada projeto possui:
- **Nome** e **descrição**
- **Status**: Ativo, Concluído, Cancelado ou Arquivado
- **Datas**: Início e término planejados
- **Documentação**: Arquivos anexados (requisitos, especificações, etc.)
- **Estatísticas**: Número de cenários, suítes e execuções recentes

---

## Cenários (Fluxos)

[Cenários](../../assets/images/conceitos-cenarios.mp4)

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

[Nós](../../assets/images/conceitos-nos.mp4)

**Nós** são os blocos fundamentais de um cenário. Cada nó executa uma ação específica e pode produzir outputs que são consumidos por nós seguintes.

### Categorias de Nós

| Categoria | Cor | Nós |
|-----------|-----|-----|
| **Controle de Fluxo** | 🟡 Amarelo | If, Switch, Loop |
| **Web** | 🔵 Azul | Smart Web Flow, Web Flow, Smart Locators |
| **Mobile** | 🔴 Vermelho Claro | Mobile Flow |
| **API** | 🟣 Roxo | HTTP Request |
| **Banco de Dados** | 🟢 Verde | PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, MongoDB |
| **Arquivos** | 🟤 Dourado | Gerar Arquivo, Extrair Arquivo, Base64 para Arquivo, Arquivo para Base64 |
| **Infraestrutura** | 🔵 Azul Claro | SSH Command |
| **Performance** | 🟫 Âmbar | Load Test |
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
{{ steps["http-request"].outputs.body.token }}    → Token de uma resposta API
{{ steps.login.outputs.extracts.userName }}   → Texto extraído de uma página
{{ steps.query.outputs.rows[0].email }}       → Primeiro email do resultado SQL
```

> Para detalhes completos, veja a documentação de [Expressões](../expressoes/expressoes.md).

---

## Componentes Reutilizáveis

[Componentes](../../assets/images/conceitos-componentes.mp4)

Um **componente** é um fluxo reutilizável com contrato de entrada e saída. Ele permite encapsular uma sequência que aparece em vários cenários, como login, preparação de massa, validação comum ou consulta auxiliar.

Componentes possuem:

- **Input**: campos que o cenário precisa informar;
- **Fluxo interno**: nós executados dentro do componente;
- **Output**: valor retornado ao cenário;
- **Status**: rascunho ou publicado.

Depois de publicado, o componente aparece na paleta do editor de cenários e pode ser arrastado para o canvas como um nó comum.

> Para detalhes, veja [Componentes Reutilizáveis](../componentes/visao-geral.md).

---

## Conexões (Edges)

[Conexões](../../assets/images/conceitos-conectar-nos.mp4)

**Conexões** ligam a saída de um nó à entrada de outro, definindo a ordem de execução. O QANode executa os nós em **ordem topológica** — ou seja, um nó só é executado depois que todos os nós conectados à sua entrada foram concluídos.

### Conexões Condicionais

Alguns nós possuem múltiplas saídas condicionais:

- **If** → saídas `true` e `false`
- **Switch** → saídas para cada case + `default`
- **Loop** → saídas `loop` (próxima iteração) e `done` (fim do loop)

Apenas o caminho correspondente à condição avaliada será executado. Nós em caminhos não ativados serão **pulados**.

---

## Execuções (Runs)

[Execuções](../../assets/images/conceitos-execucao-fluxos.mp4)

Uma **execução** é uma instância de um cenário sendo processado. Ao executar um cenário, o QANode:

1. Resolve a ordem topológica dos nós
2. Executa cada nó sequencialmente
3. Avalia expressões e injeta dados dos outputs anteriores
4. Registra logs, screenshots, arquivos e artefatos
5. Disponibiliza os resultados da execução

Cada execução possui:
- **Status**: `running`, `success` ou `failed`
- **Duração**: Tempo total de execução
- **Passos**: Resultado individual de cada nó
- **Artefatos**: Screenshots e arquivos gerados
- **Logs**: Mensagens detalhadas de cada passo

---

## Suítes de Teste

[Suítes](../../assets/images/conceitos-suites.mp4)

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

[Variáveis](../../assets/images/conceitos-variaveis.mp4)

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

[Credenciais](../../assets/images/conceitos-credenciais.mp4)

**Credenciais** armazenam informações de conexão de forma segura e centralizada. Tipos suportados:

- **HTTP/API** — URL base, autenticação, headers
- **PostgreSQL**, **MySQL**, **MariaDB**, **SQL Server**, **Oracle** — Dados de conexão ao banco
- **MongoDB** — URI ou dados de conexão
- **SSH** — Host, usuário, senha ou chave privada

Credenciais podem ser referenciadas por nós que precisam de conexão, evitando que senhas fiquem expostas nos fluxos.


---

## Próximos Passos

Agora que você conhece os conceitos fundamentais:

- [Editor de Fluxos](../editor-fluxos/visao-geral.md) — Aprenda a usar o editor visual
- [Referência de Nós](../nos/controle-fluxo/if.md) — Conheça todos os nós em detalhes
- [Expressões](../expressoes/expressoes.md) — Domine o sistema de interpolação
