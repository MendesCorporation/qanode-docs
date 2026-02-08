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

- **Custom SQL** — SQL livre com parâmetros
- **SELECT** — Query builder visual
- **EXISTS** — Verificar existência
- **COUNT** — Contar registros
- **ASSERT** — Verificar valor
- **INSERT** — Inserir registros
- **UPDATE** — Atualizar registros
- **DELETE** — Remover registros

> Para detalhes completos de cada modo, consulte a documentação do nó [PostgreSQL](postgresql.md).

**Nota:** No MySQL, os placeholders de parâmetros usam `?` em vez de `$1`, `$2`:

```sql
SELECT * FROM users WHERE email = ? AND active = ?
```

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `rows` | `array` | Registros retornados |
| `rowCount` | `number` | Número de registros afetados/retornados |

---

## Dicas

- Lembre-se da diferença de placeholders: MySQL usa `?`, PostgreSQL usa `$1`, `$2`
- O query builder visual gera SQL correto para MySQL automaticamente
- Para mais detalhes sobre cada modo de consulta, veja a documentação do [PostgreSQL](postgresql.md)
