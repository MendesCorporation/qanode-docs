# Nó Oracle

O nó **Oracle** permite executar consultas e operações no banco de dados Oracle Database.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `oracle-query` |
| **Categoria** | Banco de Dados |
| **Cor** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Conexão

### Usando Credenciais Salvas (Recomendado)

Selecione uma credencial do tipo **Oracle**.

### Configuração Manual

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Connection String** | `string` | URI completa (TNS ou Easy Connect) |
| **Host** | `string` | Endereço do servidor |
| **Porta** | `number` | Porta (padrão: 1521) |
| **Service Name** | `string` | Nome do serviço Oracle |
| **Usuário** | `string` | Usuário |
| **Senha** | `string` | Senha |

**Connection String (Easy Connect):**
```
host:1521/service_name
```

---

## Modos de Consulta

Suporta os mesmos presets dos outros nós de banco de dados:

- **Custom SQL** — SQL livre com expressões `{{ }}` diretamente na query
- **SELECT** — Query builder visual
- **EXISTS** — Verificar existência
- **COUNT** — Contar registros
- **ASSERT** — Verificar valor
- **INSERT** — Inserir registros
- **UPDATE** — Atualizar registros
- **DELETE** — Remover registros

**Exemplo de Custom SQL com variável:**

```sql
SELECT *
FROM users
WHERE email = '{{ variables.EMAIL_TESTE }}'
  AND active = 1
```

Use aspas para textos e datas. Para números e flags, normalmente use a expressão sem aspas:

```sql
SELECT *
FROM orders
WHERE customer_id = {{ variables.CUSTOMER_ID }}
```

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `rows` | `array` | Registros retornados |
| `rowCount` | `number` | Número de registros afetados/retornados |

---

## Diferenças do Oracle

- **Service Name** em vez de database name
- Sintaxe SQL Oracle (ex: `ROWNUM` em vez de `LIMIT`)
- O query builder visual gera SQL compatível automaticamente
