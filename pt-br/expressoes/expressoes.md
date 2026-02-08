# Sistema de Expressões

O QANode utiliza um sistema de **expressões** com a sintaxe `{{ }}` que permite injetar dados dinâmicos em qualquer campo de configuração dos nós. Com expressões, você pode acessar outputs de nós anteriores, variáveis globais e variáveis de ambiente.

---

## Sintaxe Básica

```
{{ expressão }}
```

Tudo dentro de `{{ }}` é avaliado como JavaScript no contexto de execução do fluxo.

---

## Contexto Disponível

### `steps` — Outputs de Nós Anteriores

Acesse os dados produzidos por qualquer nó que já foi executado:

```
{{ steps['Nome do Nó'].outputs.propriedade }}
```

| Componente | Descrição |
|------------|-----------|
| `steps` | Objeto com todos os nós executados |
| `['Nome do Nó']` | Label (rótulo) do nó — ver regras abaixo |
| `.outputs` | Dados de saída do nó |
| `.propriedade` | Campo específico do output |

#### Como funciona o rótulo (label) nos steps

O nome usado em `steps` é o **rótulo** (label) do nó. Todo nó recebe um rótulo padrão ao ser criado (ex: "HTTP Request", "Postgres", "Web Flow"). Você pode alterar o rótulo no painel de propriedades.

**Regra de notação:**

| Situação | Rótulo | Notação | Exemplo |
|----------|--------|---------|---------|
| Rótulo padrão (com espaço) | `HTTP Request` | Colchetes `['...']` | `steps['HTTP Request'].outputs.status` |
| Rótulo customizado simples | `login` | Ponto `.` | `steps.login.outputs.json.token` |
| Rótulo customizado com espaço | `Consulta DB` | Colchetes `['...']` | `steps['Consulta DB'].outputs.rows` |
| Rótulo removido (vazio) | — | Usa `node.type` | `steps['http-request'].outputs.status` |

> **Dica:** Se você renomear o rótulo do nó para uma única palavra sem espaços (ex: `login`, `api`, `query`), pode usar a notação com ponto, que é mais simples: `steps.login.outputs.json.token`

**Nós duplicados:** Se houver dois nós com o mesmo rótulo, o segundo recebe um sufixo numérico (ex: "HTTP Request 2", "HTTP Request 3").

#### Rótulos padrão dos nós

| Tipo de Nó | Rótulo Padrão |
|------------|--------------|
| HTTP Request | `HTTP Request` |
| Postgres | `Postgres` |
| MySQL | `MySQL` |
| MariaDB | `MariaDB` |
| Oracle | `Oracle` |
| MongoDB | `MongoDB` |
| Web Flow | `Web Flow` |
| Smart Locators | `Smart Locators` |
| SSH Command | `SSH Command` |
| Custom JavaScript | `Custom JavaScript` |
| Set Variable | `Set Variable` |
| Log | `Log` |
| If / Switch / Loop / Merge | `If` / `Switch` / `Loop` / `Merge` |

**Exemplos com rótulos padrão:**

```
// Status HTTP (rótulo padrão "HTTP Request" — precisa de colchetes)
{{ steps['HTTP Request'].outputs.status }}  →  200

// Corpo JSON da resposta
{{ steps['HTTP Request'].outputs.json.name }}  →  "João"

// Array do banco de dados
{{ steps['Postgres'].outputs.rows[0].email }}  →  "joao@exemplo.com"

// Contagem de registros
{{ steps['Postgres'].outputs.rowCount }}  →  5

// Texto extraído de página web
{{ steps['Web Flow'].outputs.extracts.titulo }}  →  "Dashboard"

// Resultado de asserção
{{ steps['Web Flow'].outputs.asserts.loginOk }}  →  true

// Session ID do navegador
{{ steps['Smart Locators'].outputs.sessionId }}  →  "abc-123"

// Saída SSH
{{ steps['SSH Command'].outputs.stdout }}  →  "active (running)"
{{ steps['SSH Command'].outputs.exitCode }}  →  0
```

**Exemplos com rótulos customizados (sem espaço — notação com ponto):**

```
// Se você renomear o nó HTTP Request para "api":
{{ steps.api.outputs.status }}  →  200
{{ steps.api.outputs.json.name }}  →  "João"

// Se você renomear o nó Custom JavaScript para "transform":
{{ steps.transform.outputs.result }}  →  { "total": 42 }

// Se você renomear o nó Postgres para "query":
{{ steps.query.outputs.rows[0].email }}  →  "joao@exemplo.com"
```

### `variables` — Variáveis Globais

Acesse variáveis cadastradas no sistema:

```
{{ variables.NOME_DA_VARIAVEL }}
```

**Exemplos:**

```
// String
{{ variables.API_URL }}  →  "https://api.exemplo.com"

// Número
{{ variables.DEFAULT_TIMEOUT }}  →  30000

// Boolean
{{ variables.FEATURE_FLAG }}  →  true

// JSON (acesso a propriedades)
{{ variables.CONFIG.retries }}  →  3
{{ variables.CONFIG.timeout }}  →  5000
```


---

## Modos de Uso

### Expressão Única (Valor Direto)

Quando o campo inteiro é uma expressão, o valor retornado mantém seu tipo original (número, objeto, array, boolean):

```
Campo: {{ steps['Postgres'].outputs.rows }}
Resultado: [{ "id": 1, "name": "João" }, { "id": 2, "name": "Maria" }]  (array)

Campo: {{ steps['HTTP Request'].outputs.status }}
Resultado: 200  (número)

Campo: {{ steps['Web Flow'].outputs.asserts.loginOk }}
Resultado: true  (boolean)
```

### Interpolação (Dentro de Texto)

Quando a expressão está misturada com texto, o resultado é sempre uma string:

```
Campo: Olá, {{ steps['Postgres'].outputs.rows[0].name }}!
Resultado: "Olá, João!"  (string)

Campo: {{ variables.API_URL }}/api/users/{{ steps['HTTP Request'].outputs.json.id }}
Resultado: "https://api.exemplo.com/api/users/42"  (string)

Campo: Status: {{ steps['HTTP Request'].outputs.status }} - {{ steps['HTTP Request'].outputs.json.message }}
Resultado: "Status: 200 - OK"  (string)
```

---

## Expressões JavaScript

Dentro de `{{ }}`, qualquer expressão JavaScript válida funciona:

### Operações com Strings

```
// Maiúsculas (nó Web Flow renomeado para "extract")
{{ steps.extract.outputs.extracts.nome.toUpperCase() }}

// Substring
{{ steps.extract.outputs.extracts.texto.substring(0, 100) }}

// Substituição
{{ steps.extract.outputs.extracts.url.replace('http:', 'https:') }}

// Concatenação (usando rótulo padrão com colchetes)
{{ steps['Postgres'].outputs.rows[0].firstName + ' ' + steps['Postgres'].outputs.rows[0].lastName }}
```

### Operações com Números

```
// Cálculo (nó HTTP Request renomeado para "cart")
{{ steps.cart.outputs.json.subtotal * 1.1 }}

// Arredondamento (nó renomeado para "api")
{{ Math.round(steps.api.outputs.json.score * 100) / 100 }}

// Formatação (nó Postgres renomeado para "query")
{{ steps.query.outputs.rowCount.toString().padStart(5, '0') }}
```

### Operações com Arrays

```
// Tamanho
{{ steps['Postgres'].outputs.rows.length }}

// Primeiro/Último
{{ steps['Postgres'].outputs.rows[0].name }}
{{ steps['Postgres'].outputs.rows[steps['Postgres'].outputs.rows.length - 1].name }}

// Filtrar
{{ steps['Postgres'].outputs.rows.filter(r => r.active).length }}

// Mapear
{{ steps['Postgres'].outputs.rows.map(r => r.email).join(', ') }}
```

### Operações com Objetos

```
// Acesso profundo (nó renomeado para "api")
{{ steps.api.outputs.json.data.user.address.city }}

// Ou usando rótulo padrão com colchetes:
{{ steps['HTTP Request'].outputs.json.data.user.address.city }}

// Chaves
{{ Object.keys(steps.api.outputs.json).join(', ') }}

// Stringify
{{ JSON.stringify(steps.api.outputs.json) }}
```

### Condicionais (Ternário)

```
// Valor condicional
{{ steps['HTTP Request'].outputs.status === 200 ? 'Sucesso' : 'Falha' }}

// Fallback para null/undefined
{{ steps['Web Flow'].outputs.extracts.titulo || 'Sem título' }}
```

---

## Uso nos Nós

### URL de Navegação

```
{{ variables.BASE_URL }}/login
{{ variables.BASE_URL }}/users/{{ steps['HTTP Request'].outputs.json.id }}
```

### Texto para Fill/Type

```
{{ variables.TEST_EMAIL }}
{{ steps['Custom JavaScript'].outputs.result.email }}
test-{{ Date.now() }}@exemplo.com
```

### SQL com Dados Dinâmicos

```sql
SELECT * FROM orders WHERE user_id = '{{ steps['HTTP Request'].outputs.json.userId }}'
```

> **Cuidado:** Use parâmetros ($1, ?) em vez de interpolação direta para prevenir SQL injection quando possível.

### Body de HTTP Request

```json
{
  "token": "{{ steps.login.outputs.json.token }}",
  "data": {
    "name": "{{ variables.TEST_NAME }}",
    "email": "{{ steps['Custom JavaScript'].outputs.result.email }}"
  }
}
```

> No exemplo acima, `steps.login` é um nó HTTP Request renomeado para "login".

### Condição do If

```
{{ steps['HTTP Request'].outputs.status }} === 200
{{ steps['Postgres'].outputs.rowCount }} > 0
{{ steps['Web Flow'].outputs.extracts.price }} < {{ variables.MAX_PRICE }}
```

### Mensagem de Log

```
Usuário {{ steps['Postgres'].outputs.rows[0].name }} tem {{ steps['Postgres'].outputs.rows[0].orders }} pedidos
API respondeu com status {{ steps['HTTP Request'].outputs.status }}: {{ steps['HTTP Request'].outputs.json.message }}
```

---

## Referência por Label vs ID

Você pode referenciar nós pelo **rótulo (label)** ou pelo **ID interno** do nó:

```
// Por rótulo padrão (colchetes — tem espaço no nome)
{{ steps['HTTP Request'].outputs.status }}
{{ steps['Smart Locators'].outputs.sessionId }}

// Por rótulo customizado sem espaço (ponto — mais simples)
{{ steps.login.outputs.json.token }}
{{ steps.api.outputs.status }}

// Por rótulo customizado com espaço (colchetes)
{{ steps['Consulta Usuários'].outputs.rows }}

// Por ID interno (mais estável — não muda ao renomear o nó)
{{ steps.node_abc123.outputs.json.token }}
```

> **Dica:** Renomeie seus nós com rótulos simples e descritivos (ex: `login`, `api`, `query`) para usar a notação com ponto, que é mais limpa e fácil de ler.

---

## Tratamento de Erros

Se uma expressão falha (propriedade não existe, tipo incorreto, etc.), o resultado é substituído por um marcador de erro:

```
[Error: Cannot read property 'name' of undefined]
```

Para prevenir erros, use o operador de optional chaining ou fallbacks:

```
// Optional chaining (nó renomeado para "api")
{{ steps.api.outputs.json?.user?.name }}

// Ou com rótulo padrão:
{{ steps['HTTP Request'].outputs.json?.user?.name }}

// Fallback
{{ steps['HTTP Request'].outputs.json.name || 'N/A' }}
```

> **Importante:** Se uma expressão referenciar um nó ou variável que não existe (ex: erro de digitação), o nó falhará com uma mensagem de erro clara indicando qual expressão falhou, em vez de causar comportamento inesperado.

---

## Segurança

O sistema de expressões roda em um **ambiente sandbox** com restrições:

- **Sem acesso** a `require`, `import`, `process`
- **Sem acesso** ao sistema de arquivos
- **Sem acesso** a rede (use nó HTTP Request para isso)
- Apenas `steps`, `variables` e `env` estão disponíveis

---

## Dicas

- **Renomeie nós com rótulos simples** — `{{ steps.login.outputs.json.token }}` é mais limpo que `{{ steps['HTTP Request'].outputs.json.token }}`
- **Rótulos sem espaço permitem ponto** — renomear para `api` permite `steps.api` em vez de `steps['HTTP Request']`
- **Use interpolação** para construir URLs e mensagens
- **Use expressão única** quando precisar do valor com tipo original (array, número)
- **Adicione um nó Log** para inspecionar valores durante depuração
- **Use fallbacks** (`|| 'default'`) para evitar erros com dados opcionais
- **Use optional chaining** (`?.`) para acessos profundos em objetos que podem ser nulos
- Para lógica complexa, use um nó **Custom JavaScript** e referencie o resultado
