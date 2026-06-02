# Nó PostgreSQL

O nó **PostgreSQL** permite executar consultas e operações no banco de dados PostgreSQL. Oferece tanto um **query builder visual** quanto a opção de escrever **SQL customizado**.

---

## Visão Geral

[Visão Geral](../../assets/images/nos-postgres-visao-geral.mp4)

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `postgres-query` |
| **Categoria** | Banco de Dados |
| **Cor** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Conexão

### Usando Credenciais Salvas (Recomendado)

Selecione uma credencial do tipo **PostgreSQL** no campo **Credencial**. A credencial contém todas as informações de conexão.

### Configuração Manual

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Connection String** | `string` | URI completa (alternativa) |
| **Host** | `string` | Endereço do servidor |
| **Porta** | `number` | Porta (padrão: 5432) |
| **Banco de Dados** | `string` | Nome do banco |
| **Usuário** | `string` | Usuário de conexão |
| **Senha** | `string` | Senha de conexão |
| **SSL** | `boolean` | Usar conexão SSL |

**Connection String:**
```
postgresql://usuario:senha@host:5432/banco?sslmode=require
```

---

## Modos de Consulta (Presets)

### Custom SQL

Escreva SQL livremente no campo **SQL**. Para usar dados dinâmicos, coloque expressões `{{ }}` diretamente na query.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **SQL** | `string` | Consulta SQL (suporta `{{ }}`) |

**Exemplo com variável:**
```sql
SELECT *
FROM users
WHERE email = '{{ variables.EMAIL_TESTE }}'
  AND active = true;
```

Se `variables.EMAIL_TESTE` for `joao@exemplo.com`, o SQL executado será equivalente a:

```sql
SELECT *
FROM users
WHERE email = 'joao@exemplo.com'
  AND active = true;
```

**Exemplo com output de outro nó:**

```sql
SELECT *
FROM users
WHERE id = {{ steps.api.outputs.body.id }};
```

Para textos, datas e valores tratados como string, coloque aspas ao redor da expressão. Para números e booleanos, normalmente use a expressão sem aspas.

> **Cuidado:** como a expressão é aplicada diretamente no SQL, use valores controlados e revise aspas em campos de texto. Evite usar conteúdo livre digitado por usuário final sem validação.

---

### SELECT (Visual Builder)

Construa consultas SELECT sem escrever SQL:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tabela** | `string` | Tabela de origem |
| **Colunas** | `array` | Colunas a selecionar (vazio = todas) |
| **Condições WHERE** | `array` | Filtros |
| **ORDER BY** | `object` | Coluna e direção de ordenação |
| **LIMIT** | `number` | Limite de registros |

**Condições WHERE:**

| Campo | Operador | Valor |
|-------|----------|-------|
| `email` | `=` | `joao@exemplo.com` |
| `age` | `>` | `18` |
| `name` | `LIKE` | `%Silva%` |

---

### EXISTS

Verifica se existe pelo menos um registro que corresponda aos critérios.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tabela** | `string` | Tabela |
| **Condições WHERE** | `array` | Critérios de busca |

Retorna `rowCount: 1` se existe, `rowCount: 0` se não existe.

---

### COUNT

Conta registros que correspondem aos critérios.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tabela** | `string` | Tabela |
| **Condições WHERE** | `array` | Critérios de contagem |

---

### ASSERT

Verifica se um valor no banco corresponde ao esperado. Ideal para validações de dados.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tabela** | `string` | Tabela |
| **Coluna** | `string` | Coluna a verificar |
| **Operador** | `string` | Operação de comparação |
| **Valor** | `string` | Valor esperado |
| **Condições WHERE** | `array` | Filtros para localizar o registro |

---

### INSERT

Insere novos registros.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tabela** | `string` | Tabela de destino |
| **Valores** | `array` | Pares coluna-valor |

**Exemplo:**

| Coluna | Valor |
|--------|-------|
| `name` | `Maria` |
| `email` | `maria@exemplo.com` |
| `active` | `true` |

---

### UPDATE

Atualiza registros existentes.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tabela** | `string` | Tabela |
| **Sets** | `array` | Pares coluna-valor a atualizar |
| **Condições WHERE** | `array` | Filtros |

---

### DELETE

Remove registros.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tabela** | `string` | Tabela |
| **Condições WHERE** | `array` | Filtros (obrigatório por segurança) |

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `rows` | `array` | Array de registros retornados |
| `rowCount` | `number` | Número de registros afetados/retornados |

### Acessando os Outputs

```
// Todos os registros
{{ steps.query.outputs.rows }}

// Primeiro registro
{{ steps.query.outputs.rows[0] }}

// Campo específico do primeiro registro
{{ steps.query.outputs.rows[0].name }}  →  "João"
{{ steps.query.outputs.rows[0].email }}  →  "joao@exemplo.com"

// Contagem
{{ steps.query.outputs.rowCount }}  →  5
```

---

## Exemplos Práticos

### Validar dados após criação via API

```
[HTTP Request: POST /api/users → body: { "name": "João" }]
    │
    ▼
[PostgreSQL: SELECT * FROM users WHERE name = 'João']
    │
    ▼
[If: {{ steps.query.outputs.rowCount }} > 0]
    │ true → [Log: "Usuário criado com sucesso no banco"]
    │ false → [Stop and Fail]
```

### Preparar dados para teste

```
[PostgreSQL: INSERT INTO test_data (key, value) VALUES ('token', 'abc123')]
    │
    ▼
[HTTP Request: GET /api/verify?token={{ steps.insert.outputs.rows[0].token }}]
```

### Limpar dados após teste

```
[... testes ...]
    │
    ▼
[PostgreSQL: DELETE FROM test_data WHERE created_by = 'automated-test']
```

---

## Dicas

- **Use credenciais salvas** para gerenciar conexões de forma centralizada
- Em **Custom SQL**, use expressões `{{ }}` diretamente na query e revise aspas para textos e datas
- O **query builder visual** é ideal para operações simples e é menos propenso a erros de sintaxe
- **Use expressões** nos valores: `{{ steps.api.outputs.body.id }}` para usar dados de nós anteriores
- **Teste a conexão** antes de executar o fluxo usando o botão de teste na tela de credenciais
