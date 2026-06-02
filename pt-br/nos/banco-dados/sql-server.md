# Nó SQL Server

O nó **SQL Server** permite executar consultas e operações em bancos Microsoft SQL Server. Ele oferece **query builder visual** para operações comuns e também permite escrever **SQL customizado** quando você precisa de controle total.

---

## Visão Geral

[Visão Geral](../../assets/images/nos-sql-server-visao-geral.mp4)

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `sqlserver-query` |
| **Categoria** | Banco de Dados |
| **Cor** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Conexão

### Usando Credenciais Salvas (Recomendado)

Selecione uma credencial do tipo **SQL Server** no campo **Credencial**. A credencial concentra os dados de conexão e evita expor senha diretamente no nó.

### Configuração Manual

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Host** | `string` | Endereço do servidor SQL Server |
| **Porta** | `number` | Porta TCP (padrão: 1433) |
| **Banco de Dados** | `string` | Nome do database |
| **Usuário** | `string` | Usuário de conexão |
| **Senha** | `string` | Senha de conexão |
| **Encrypt** | `boolean` | Usa conexão criptografada |
| **Trust Server Certificate** | `boolean` | Aceita certificado do servidor sem validação completa |

Em ambientes corporativos, é comum usar `Encrypt` habilitado. Em ambientes internos com certificado próprio, `Trust Server Certificate` pode ser necessário.

---

## Modos de Consulta

O SQL Server suporta os mesmos presets visuais dos outros nós relacionais:

- **Custom SQL** — SQL livre com expressões `{{ }}`;
- **SELECT** — query builder visual;
- **EXISTS** — verifica existência de registro;
- **COUNT** — conta registros;
- **ASSERT** — valida valor no banco;
- **INSERT** — insere registros;
- **UPDATE** — atualiza registros;
- **DELETE** — remove registros.

O query builder gera SQL compatível com SQL Server automaticamente.

---

## Custom SQL

Use **Custom SQL** quando precisar escrever a consulta manualmente.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **SQL** | `string` | Consulta SQL, com suporte a expressões `{{ }}` |

### Usando Expressões no SQL

No painel atual do nó SQL Server, a query customizada é escrita diretamente no campo **SQL**. Para usar valores dinâmicos, escreva expressões `{{ }}` no próprio SQL.

Exemplo completo:

```sql
SELECT *
FROM dbo.Users
WHERE email = '{{ variables.EMAIL_TESTE }}'
  AND active = 1;
```

Se `variables.EMAIL_TESTE` for `joao@exemplo.com`, o SQL executado será equivalente a:

```sql
SELECT *
FROM dbo.Users
WHERE email = 'joao@exemplo.com'
  AND active = 1;
```

Para valores vindos de passos anteriores, use a mesma sintaxe:

```sql
SELECT *
FROM dbo.Users
WHERE id = {{ steps.api.outputs.body.id }};
```

Para textos, datas e valores que precisam ser tratados como string no SQL, coloque aspas:

```sql
SELECT *
FROM dbo.Orders
WHERE customer_email = '{{ steps.api.outputs.body.email }}'
  AND created_at >= '{{ variables.DATA_INICIAL }}';
```

Para números e flags, normalmente não use aspas:

```sql
SELECT *
FROM dbo.Users
WHERE id = {{ variables.USER_ID }}
  AND active = {{ variables.ACTIVE_FLAG }};
```

> **Cuidado:** como a expressão é aplicada diretamente no SQL, use valores controlados e revise aspas em campos de texto. Evite usar conteúdo livre digitado por usuário final sem validação.

---

## SELECT Visual

No preset **SELECT**, você pode montar consultas sem escrever SQL manualmente.

| Campo | Descrição |
|-------|-----------|
| **Tabela** | Tabela de origem |
| **Colunas** | Colunas retornadas; vazio retorna todas |
| **Where** | Condições de filtro |
| **Order By** | Ordenação |
| **Limit** | Quantidade máxima de registros |

No SQL Server, o limite é gerado como `TOP (...)`, não como `LIMIT`.

Exemplo gerado:

```sql
SELECT TOP (10) [id], [email], [active]
FROM [dbo].[Users]
WHERE [active] = 1;
```

---

## Diferenças do SQL Server

| Tema | Comportamento |
|------|---------------|
| **Porta padrão** | `1433` |
| **Limite de linhas** | `TOP (...)` |
| **Identificadores** | Colunas e tabelas podem ser geradas com colchetes, como `[Users]` |
| **Booleanos** | Em muitos bancos SQL Server, valores booleanos são tratados como `1` e `0` em colunas `bit` |
| **TLS/certificado** | Pode exigir `Encrypt` e `Trust Server Certificate`, dependendo do ambiente |

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `rows` | `array` | Registros retornados pela consulta |
| `rowCount` | `number` | Quantidade de linhas retornadas no primeiro recordset |
| `recordsAffected` | `array` | Linhas afetadas por comandos de escrita, quando informado pelo driver |

### Acessando Outputs

```
{{ steps["sqlserver-query"].outputs.rows }}
{{ steps["sqlserver-query"].outputs.rows[0].email }}
{{ steps["sqlserver-query"].outputs.rowCount }}
{{ steps["sqlserver-query"].outputs.recordsAffected }}
```

---

## Exemplos

### Validar Registro Criado por API

```
[HTTP Request: POST /users]
    │
    ▼
[SQL Server: SELECT * FROM dbo.Users WHERE email = '{{ steps["http-request"].outputs.body.email }}']
    │
    ▼
[If: {{ steps["sqlserver-query"].outputs.rowCount > 0 }}]
```

### Preparar Massa de Teste

```sql
INSERT INTO dbo.TestData ([key], [value])
VALUES ('pedido_teste', '{{ variables.PEDIDO_ID }}');
```

### Limpar Massa de Teste

```sql
DELETE FROM dbo.TestData
WHERE [key] = 'pedido_teste';
```

---

## Dicas

- Use credenciais salvas sempre que possível.
- Teste a conexão antes de executar o cenário.
- Coloque aspas em expressões que representam texto ou datas.
- Em ambientes com certificado interno, revise as opções `Encrypt` e `Trust Server Certificate`.
- Para consultas simples, prefira o query builder visual; para joins, procedures ou SQL avançado, use Custom SQL.
