# Nó MariaDB

O nó **MariaDB** permite executar consultas e operações no banco de dados MariaDB. A interface e funcionalidades são idênticas ao nó [MySQL](mysql.md).

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `mariadb-query` |
| **Categoria** | Banco de Dados |
| **Cor** | 🟢 Verde (#22c55e) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Conexão

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Credencial** | `string` | Credencial do tipo **MariaDB** |
| **Host** | `string` | Endereço do servidor |
| **Porta** | `number` | Porta (padrão: 3306) |
| **Banco de Dados** | `string` | Nome do banco |
| **Usuário** | `string` | Usuário |
| **Senha** | `string` | Senha |
| **SSL** | `boolean` | Usar conexão SSL |

---

## Modos de Consulta

Suporta os mesmos presets: Custom SQL, SELECT, EXISTS, COUNT, ASSERT, INSERT, UPDATE, DELETE.

> O MariaDB é um fork do MySQL e usa a mesma sintaxe SQL. Para detalhes completos, consulte a documentação do [PostgreSQL](postgresql.md).

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `rows` | `array` | Registros retornados |
| `rowCount` | `number` | Número de registros afetados/retornados |
