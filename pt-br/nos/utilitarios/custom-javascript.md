# Nó Custom JavaScript

O nó **Custom JavaScript** permite executar código JavaScript customizado durante a execução do fluxo. Ideal para transformações de dados, cálculos complexos e lógica que não é coberta pelos nós nativos.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `custom-js` |
| **Categoria** | Utilitários |
| **Cor** | ⚪ Cinza (#6b7280) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Configuração

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Código** | `string` | Código JavaScript a ser executado |

O editor de código usa o **Monaco Editor** (mesmo do VS Code) com:
- Syntax highlighting
- Autocomplete para `steps`, `variables`, `return`, `console.log`
- Validação de sintaxe

---

## Contexto de Execução

O código tem acesso a:

| Variável | Tipo | Descrição |
|----------|------|-----------|
| `steps` | `object` | Outputs de todos os nós anteriores |
| `variables` | `object` | Variáveis globais e de runtime |

### Acessando dados de nós anteriores

```javascript
// Output de um HTTP Request
const userData = steps["http-request"].outputs.json;
const userName = userData.name;

// Output de uma query PostgreSQL
const rows = steps["postgres-query"].outputs.rows;
const firstUser = rows[0];

// Output de extração web
const title = steps["web-flow"].outputs.extracts.pageTitle;
```

### Acessando variáveis

```javascript
const apiUrl = variables.API_URL;
const environment = variables.ENVIRONMENT;
```

---

## Retornando Valores

O código **deve retornar** um valor usando `return`. O valor retornado fica disponível nos outputs:

```javascript
// Retornar valor simples
return "Hello World";

// Retornar objeto
return {
  fullName: steps["Query"].outputs.rows[0].first_name + " " + steps["Query"].outputs.rows[0].last_name,
  isAdmin: steps["Query"].outputs.rows[0].role === "admin"
};

// Retornar array
return steps["Query"].outputs.rows.map(row => row.email);
```

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `result` | `any` | Valor retornado pelo código |

### Acessando o resultado

```
{{ steps["custom-js"].outputs.result }}
{{ steps["custom-js"].outputs.result.fullName }}
{{ steps["custom-js"].outputs.result[0] }}
```

---

## Exemplos Práticos

### Transformar dados de API

```javascript
const items = steps["http-request"].outputs.json.items;

// Filtrar e transformar
const activeItems = items
  .filter(item => item.active)
  .map(item => ({
    id: item.id,
    name: item.name.toUpperCase(),
    price: (item.price * 1.1).toFixed(2)
  }));

return {
  items: activeItems,
  count: activeItems.length,
  totalPrice: activeItems.reduce((sum, item) => sum + parseFloat(item.price), 0)
};
```

### Gerar dados de teste

```javascript
const timestamp = Date.now();
return {
  email: `test-${timestamp}@exemplo.com`,
  username: `user_${timestamp}`,
  password: `Pass${timestamp}!`
};
```

### Validação complexa

```javascript
const response = steps["API Check"].outputs.json;
const dbRecord = steps["DB Query"].outputs.rows[0];

const validations = [];

if (response.name !== dbRecord.name) {
  validations.push(`Nome diverge: API="${response.name}" DB="${dbRecord.name}"`);
}

if (response.email !== dbRecord.email) {
  validations.push(`Email diverge: API="${response.email}" DB="${dbRecord.email}"`);
}

return {
  valid: validations.length === 0,
  errors: validations,
  errorCount: validations.length
};
```

### Formatar relatório

```javascript
const runs = steps["Query Runs"].outputs.rows;

const summary = {
  total: runs.length,
  passed: runs.filter(r => r.status === 'success').length,
  failed: runs.filter(r => r.status === 'failed').length,
  passRate: 0
};

summary.passRate = ((summary.passed / summary.total) * 100).toFixed(1) + '%';

return summary;
```

---

## Limitações

- O código roda em um **ambiente sandbox** — sem acesso a:
  - Sistema de arquivos (fs)
  - Módulos Node.js (require/import)
  - Rede (fetch, http)
  - Variáveis de ambiente (use `variables` em vez de `process.env`)
- Para operações de rede, use o nó **HTTP Request**
- Para operações de banco, use os nós de **Banco de Dados**
- Para operações de arquivo, use o nó **SSH Command**

---

## Dicas

- Use nomes descritivos para o nó: "Transformar Dados", "Gerar Credenciais", "Validar Resposta"
- **Sempre retorne** um valor — sem `return`, o output será `undefined`
- Para depuração, use `console.log()` — as mensagens aparecem nos logs do nó
- O Monaco Editor oferece autocomplete — digite `steps.` para ver os nós disponíveis
- Use para **transformar dados** entre nós, não para lógica de negócio complexa
