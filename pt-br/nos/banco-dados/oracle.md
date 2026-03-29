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

- **Custom SQL** — SQL livre com parâmetros
- **SELECT** — Query builder visual
- **EXISTS** — Verificar existência
- **COUNT** — Contar registros
- **ASSERT** — Verificar valor
- **INSERT** — Inserir registros
- **UPDATE** — Atualizar registros
- **DELETE** — Remover registros

**Nota:** No Oracle, os placeholders usam `:1`, `:2` (bind variables):

```sql
SELECT * FROM users WHERE email = :1 AND active = :2
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
- Bind variables usam `:1`, `:2` em vez de `$1` ou `?`
- Sintaxe SQL Oracle (ex: `ROWNUM` em vez de `LIMIT`)
- O query builder visual gera SQL compatível automaticamente
