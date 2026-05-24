# Nó MySQL

O nó **MySQL** permite executar consultas e operações no banco de dados MySQL. A interface e funcionalidades são idênticas ao nó [PostgreSQL](postgresql.md).

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `mysql-query` |
| **Categoria** | Banco de Dados |
| **Cor** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Conexão

### Usando Credenciais Salvas (Recomendado)

Selecione uma credencial do tipo **MySQL** no campo **Credencial**.

### Configuração Manual

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Connection String** | `string` | URI completa |
| **Host** | `string` | Endereço do servidor |
| **Porta** | `number` | Porta (padrão: 3306) |
| **Banco de Dados** | `string` | Nome do banco |
| **Usuário** | `string` | Usuário |
| **Senha** | `string` | Senha |
| **SSL** | `boolean` | Usar conexão SSL |

---

## Modos de Consulta

O MySQL suporta os mesmos presets do PostgreSQL:

- **Custom SQL** — SQL livre com expressões `{{ }}` diretamente na query
- **SELECT** — Query builder visual
- **EXISTS** — Verificar existência
- **COUNT** — Contar registros
- **ASSERT** — Verificar valor
- **INSERT** — Inserir registros
- **UPDATE** — Atualizar registros
- **DELETE** — Remover registros

> Para detalhes completos de cada modo, consulte a documentação do nó [PostgreSQL](postgresql.md).

**Exemplo de Custom SQL com variável:**

```sql
SELECT *
FROM users
WHERE email = '{{ variables.EMAIL_TESTE }}'
  AND active = 1;
```

Use aspas para textos e datas. Para números e flags, normalmente use a expressão sem aspas:

```sql
SELECT *
FROM orders
WHERE customer_id = {{ variables.CUSTOMER_ID }};
```

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `rows` | `array` | Registros retornados |
| `rowCount` | `number` | Número de registros afetados/retornados |

---

## Dicas

- Em **Custom SQL**, escreva a query no campo **SQL** e use `{{ }}` para dados dinâmicos
- Revise aspas em campos de texto e datas
- O query builder visual gera SQL correto para MySQL automaticamente
- Para mais detalhes sobre cada modo de consulta, veja a documentação do [PostgreSQL](postgresql.md)
