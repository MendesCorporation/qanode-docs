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
{{ steps.extract.outputs.extracts.nome.toUpperCase() }}  →  "JOÃO"

// Minúsculas
{{ steps.extract.outputs.extracts.nome.toLowerCase() }}  →  "joão"

// Primeira letra maiúscula
{{ steps.extract.outputs.extracts.nome.charAt(0).toUpperCase() + steps.extract.outputs.extracts.nome.slice(1).toLowerCase() }}  →  "João"

// Substring
{{ steps.extract.outputs.extracts.texto.substring(0, 100) }}  →  "Primeiros 100 caracteres..."

// Slice (permite índices negativos)
{{ steps.extract.outputs.extracts.texto.slice(0, 50) }}  →  "Primeiros 50 caracteres"
{{ steps.extract.outputs.extracts.texto.slice(-10) }}  →  "Últimos 10"

// Substituição
{{ steps.extract.outputs.extracts.url.replace('http:', 'https:') }}
{{ steps.extract.outputs.extracts.texto.replaceAll('a', 'A') }}  →  substitui todas as ocorrências

// Concatenação (usando rótulo padrão com colchetes)
{{ steps['Postgres'].outputs.rows[0].firstName + ' ' + steps['Postgres'].outputs.rows[0].lastName }}

// Template strings
{{ `${steps['Postgres'].outputs.rows[0].firstName} ${steps['Postgres'].outputs.rows[0].lastName}` }}

// Tamanho
{{ steps.extract.outputs.extracts.texto.length }}  →  150

// Verificar se contém
{{ steps.extract.outputs.extracts.email.includes('@') }}  →  true
{{ steps.extract.outputs.extracts.url.startsWith('https://') }}  →  true
{{ steps.extract.outputs.extracts.arquivo.endsWith('.pdf') }}  →  true

// Buscar posição
{{ steps.extract.outputs.extracts.email.indexOf('@') }}  →  5
{{ steps.extract.outputs.extracts.texto.lastIndexOf('palavra') }}  →  última ocorrência

// Dividir string
{{ steps.extract.outputs.extracts.tags.split(',') }}  →  ['tag1', 'tag2', 'tag3']
{{ steps.extract.outputs.extracts.nome.split('') }}  →  ['J', 'o', 'ã', 'o']

// Remover espaços
{{ steps.extract.outputs.extracts.nome.trim() }}  →  remove espaços do início e fim
{{ steps.extract.outputs.extracts.nome.trimStart() }}  →  remove espaços do início
{{ steps.extract.outputs.extracts.nome.trimEnd() }}  →  remove espaços do fim

// Repetir
{{ 'abc'.repeat(3) }}  →  "abcabcabc"

// Preencher com zeros
{{ steps.api.outputs.json.id.toString().padStart(5, '0') }}  →  "00042"
{{ steps.api.outputs.json.code.padEnd(10, '-') }}  →  "ABC-------"

// Extrair caractere
{{ steps.extract.outputs.extracts.nome.charAt(0) }}  →  "J"
{{ steps.extract.outputs.extracts.nome[0] }}  →  "J"
{{ steps.extract.outputs.extracts.nome.at(-1) }}  →  último caractere
```

### Operações com Números

```
// Cálculo (nó HTTP Request renomeado para "cart")
{{ steps.cart.outputs.json.subtotal * 1.1 }}  →  adiciona 10%
{{ steps.cart.outputs.json.price * steps.cart.outputs.json.quantity }}  →  total

// Arredondamento (nó renomeado para "api")
{{ Math.round(steps.api.outputs.json.score) }}  →  arredonda para inteiro mais próximo
{{ Math.ceil(steps.api.outputs.json.score) }}  →  arredonda para cima
{{ Math.floor(steps.api.outputs.json.score) }}  →  arredonda para baixo
{{ Math.trunc(steps.api.outputs.json.score) }}  →  remove decimais

// Casas decimais
{{ Math.round(steps.api.outputs.json.score * 100) / 100 }}  →  2 casas decimais
{{ steps.api.outputs.json.price.toFixed(2) }}  →  "10.50" (string)
{{ parseFloat(steps.api.outputs.json.price.toFixed(2)) }}  →  10.5 (número)

// Formatação (nó Postgres renomeado para "query")
{{ steps.query.outputs.rowCount.toString().padStart(5, '0') }}  →  "00042"

// Valor absoluto
{{ Math.abs(steps.api.outputs.json.balance) }}  →  sempre positivo

// Máximo e mínimo
{{ Math.max(10, 20, 30) }}  →  30
{{ Math.min(10, 20, 30) }}  →  10
{{ Math.max(...steps.query.outputs.rows.map(r => r.age)) }}  →  maior idade

// Potência e raiz
{{ Math.pow(2, 3) }}  →  8 (2³)
{{ 2 ** 3 }}  →  8 (2³)
{{ Math.sqrt(16) }}  →  4 (raiz quadrada)
{{ Math.cbrt(27) }}  →  3 (raiz cúbica)

// Número aleatório
{{ Math.random() }}  →  0.0 a 1.0
{{ Math.floor(Math.random() * 100) }}  →  0 a 99
{{ Math.floor(Math.random() * 10) + 1 }}  →  1 a 10

// Sinal
{{ Math.sign(steps.api.outputs.json.balance) }}  →  -1, 0 ou 1

// Conversão
{{ parseInt('42') }}  →  42
{{ parseInt('42.7') }}  →  42
{{ parseFloat('42.7') }}  →  42.7
{{ Number('42.7') }}  →  42.7
{{ parseInt('ff', 16) }}  →  255 (hexadecimal)

// Verificação
{{ Number.isNaN(steps.api.outputs.json.value) }}  →  true se NaN
{{ Number.isFinite(steps.api.outputs.json.value) }}  →  true se finito
{{ Number.isInteger(steps.api.outputs.json.value) }}  →  true se inteiro

// Constantes
{{ Math.PI }}  →  3.141592653589793
{{ Math.E }}  →  2.718281828459045
```

### Operações com Arrays

```
// Tamanho
{{ steps['Postgres'].outputs.rows.length }}  →  5

// Primeiro/Último
{{ steps['Postgres'].outputs.rows[0].name }}  →  primeiro
{{ steps['Postgres'].outputs.rows[steps['Postgres'].outputs.rows.length - 1].name }}  →  último
{{ steps['Postgres'].outputs.rows.at(-1).name }}  →  último (mais simples)
{{ steps['Postgres'].outputs.rows.at(-2).name }}  →  penúltimo

// Verificar se contém
{{ steps['Postgres'].outputs.rows.some(r => r.email === 'joao@exemplo.com') }}  →  true se algum
{{ steps['Postgres'].outputs.rows.every(r => r.active) }}  →  true se todos
{{ steps['Postgres'].outputs.rows.includes('valor') }}  →  true se contém valor exato

// Buscar
{{ steps['Postgres'].outputs.rows.find(r => r.id === 42) }}  →  primeiro objeto que atende
{{ steps['Postgres'].outputs.rows.findIndex(r => r.id === 42) }}  →  índice do primeiro
{{ steps['Postgres'].outputs.rows.indexOf('valor') }}  →  índice de valor primitivo

// Filtrar
{{ steps['Postgres'].outputs.rows.filter(r => r.active) }}  →  apenas ativos
{{ steps['Postgres'].outputs.rows.filter(r => r.age > 18) }}  →  maiores de 18
{{ steps['Postgres'].outputs.rows.filter(r => r.active).length }}  →  contar ativos
{{ steps['Postgres'].outputs.rows.filter(r => r.email.includes('@gmail.com')) }}  →  apenas Gmail

// Mapear (transformar)
{{ steps['Postgres'].outputs.rows.map(r => r.email) }}  →  ['email1', 'email2', ...]
{{ steps['Postgres'].outputs.rows.map(r => r.email).join(', ') }}  →  "email1, email2, ..."
{{ steps['Postgres'].outputs.rows.map(r => r.name.toUpperCase()) }}  →  nomes em maiúsculas
{{ steps['Postgres'].outputs.rows.map((r, i) => `${i + 1}. ${r.name}`) }}  →  lista numerada

// Reduzir (agregar)
{{ steps['Postgres'].outputs.rows.reduce((sum, r) => sum + r.price, 0) }}  →  soma de preços
{{ steps['Postgres'].outputs.rows.reduce((max, r) => r.age > max ? r.age : max, 0) }}  →  maior idade
{{ steps['Postgres'].outputs.rows.reduce((acc, r) => acc + r.name + ', ', '') }}  →  concatenar nomes

// Ordenar
{{ steps['Postgres'].outputs.rows.sort((a, b) => a.name.localeCompare(b.name)) }}  →  ordem alfabética
{{ steps['Postgres'].outputs.rows.sort((a, b) => a.age - b.age) }}  →  ordem crescente por idade
{{ steps['Postgres'].outputs.rows.sort((a, b) => b.price - a.price) }}  →  ordem decrescente por preço

// Reverter
{{ steps['Postgres'].outputs.rows.reverse() }}  →  inverte ordem
{{ steps['Postgres'].outputs.rows.slice().reverse() }}  →  inverte sem modificar original

// Fatiar
{{ steps['Postgres'].outputs.rows.slice(0, 10) }}  →  primeiros 10
{{ steps['Postgres'].outputs.rows.slice(-5) }}  →  últimos 5
{{ steps['Postgres'].outputs.rows.slice(2, 5) }}  →  do índice 2 ao 4

// Concatenar
{{ steps.query1.outputs.rows.concat(steps.query2.outputs.rows) }}  →  junta dois arrays
{{ [...steps.query1.outputs.rows, ...steps.query2.outputs.rows] }}  →  spread operator

// Achatar (flat)
{{ [[1, 2], [3, 4]].flat() }}  →  [1, 2, 3, 4]
{{ [[1, [2, [3]]]].flat(2) }}  →  [1, 2, 3] (2 níveis)
{{ steps.api.outputs.json.data.flatMap(item => item.tags) }}  →  todos os tags de todos os items

// Juntar em string
{{ steps['Postgres'].outputs.rows.map(r => r.name).join(', ') }}  →  "João, Maria, Pedro"
{{ steps['Postgres'].outputs.rows.map(r => r.name).join(' | ') }}  →  "João | Maria | Pedro"

// Remover duplicatas
{{ [...new Set(steps['Postgres'].outputs.rows.map(r => r.city))] }}  →  cidades únicas

// Criar array
{{ Array.from({length: 5}, (_, i) => i) }}  →  [0, 1, 2, 3, 4]
{{ Array.from({length: 5}, (_, i) => i + 1) }}  →  [1, 2, 3, 4, 5]
{{ Array(3).fill(0) }}  →  [0, 0, 0]
{{ Array(3).fill('x') }}  →  ['x', 'x', 'x']

// Verificar se é array
{{ Array.isArray(steps.api.outputs.json.data) }}  →  true
```

### Operações com Objetos

```
// Acesso profundo (nó renomeado para "api")
{{ steps.api.outputs.json.data.user.address.city }}
// Ou usando rótulo padrão com colchetes:
{{ steps['HTTP Request'].outputs.json.data.user.address.city }}

// Optional chaining (evita erro se propriedade não existe)
{{ steps.api.outputs.json?.data?.user?.name }}  →  undefined se não existir
{{ steps.api.outputs.json?.data?.user?.name ?? 'Sem nome' }}  →  valor padrão

// Chaves (keys)
{{ Object.keys(steps.api.outputs.json) }}  →  ['id', 'name', 'email']
{{ Object.keys(steps.api.outputs.json).length }}  →  quantidade de propriedades
{{ Object.keys(steps.api.outputs.json).join(', ') }}  →  "id, name, email"

// Valores (values)
{{ Object.values(steps.api.outputs.json) }}  →  [42, 'João', 'joao@exemplo.com']

// Entradas (entries)
{{ Object.entries(steps.api.outputs.json) }}  →  [['id', 42], ['name', 'João'], ...]
{{ Object.entries(steps.api.outputs.json).map(([k, v]) => `${k}: ${v}`).join(', ') }}

// Verificar propriedade
{{ 'email' in steps.api.outputs.json }}  →  true se propriedade existe
{{ Object.hasOwn(steps.api.outputs.json, 'email') }}  →  true se propriedade própria

// Mesclar objetos
{{ Object.assign({}, steps.api.outputs.json, {newProp: 'value'}) }}
{{ {...steps.api.outputs.json, newProp: 'value'} }}  →  spread operator

// Stringify (converter para JSON string)
{{ JSON.stringify(steps.api.outputs.json) }}  →  '{"id":42,"name":"João"}'
{{ JSON.stringify(steps.api.outputs.json, null, 2) }}  →  formatado com indentação

// Parse (converter de JSON string)
{{ JSON.parse(steps.api.outputs.body) }}  →  objeto JavaScript
{{ JSON.parse(steps.api.outputs.body).data.user.name }}  →  acesso após parse

// Extrair propriedades específicas
{{ (({id, name}) => ({id, name}))(steps.api.outputs.json) }}  →  apenas id e name

// Contar propriedades
{{ Object.keys(steps.api.outputs.json).length }}  →  quantidade de campos

// Transformar objeto em array
{{ Object.entries(steps.api.outputs.json).map(([k, v]) => ({key: k, value: v})) }}
```

### Condicionais (Ternário)

```
// Valor condicional
{{ steps['HTTP Request'].outputs.status === 200 ? 'Sucesso' : 'Falha' }}

// Múltiplas condições
{{ steps.api.outputs.json.age >= 18 ? 'Adulto' : 'Menor' }}
{{ steps.api.outputs.json.score >= 90 ? 'A' : steps.api.outputs.json.score >= 80 ? 'B' : 'C' }}

// Com operadores lógicos
{{ (steps.api.outputs.json.age >= 18 && steps.api.outputs.json.active) ? 'Permitido' : 'Negado' }}
{{ (steps.api.outputs.status === 200 || steps.api.outputs.status === 201) ? 'OK' : 'Erro' }}

// Fallback para null/undefined
{{ steps['Web Flow'].outputs.extracts.titulo || 'Sem título' }}
{{ steps.api.outputs.json.name || 'Anônimo' }}
{{ steps.api.outputs.json.config || {} }}  →  objeto vazio se não existir

// Nullish coalescing (apenas null/undefined, não valores falsy)
{{ steps.api.outputs.json.count ?? 0 }}  →  0 apenas se null/undefined
{{ steps.api.outputs.json.name ?? 'Sem nome' }}

// Verificação de existência
{{ steps.api.outputs.json.email ? 'Email: ' + steps.api.outputs.json.email : 'Sem email' }}

// Formatação condicional
{{ steps.api.outputs.json.price > 1000 ? 'R$ ' + (steps.api.outputs.json.price / 1000).toFixed(1) + 'k' : 'R$ ' + steps.api.outputs.json.price }}
```

### Operações Lógicas e Comparação

```
// Comparação
{{ 5 > 3 }}  →  true
{{ 10 < 20 }}  →  true
{{ 5 >= 5 }}  →  true
{{ 3 <= 2 }}  →  false
{{ 'abc' === 'abc' }}  →  true (igualdade estrita)
{{ 10 !== 5 }}  →  true (diferente)
{{ '5' == 5 }}  →  true (igualdade frouxa)
{{ '5' === 5 }}  →  false (tipos diferentes)

// Operadores lógicos
{{ true && false }}  →  false (E lógico)
{{ true || false }}  →  true (OU lógico)
{{ !true }}  →  false (NÃO lógico)
{{ !!value }}  →  converte para boolean

// Comparações com dados
{{ steps.api.outputs.status === 200 }}  →  true/false
{{ steps.api.outputs.json.age > 18 }}  →  true/false
{{ steps['Postgres'].outputs.rowCount > 0 }}  →  true/false
{{ steps.api.outputs.json.email.includes('@') }}  →  true/false

// Múltiplas condições
{{ steps.api.outputs.status === 200 && steps.api.outputs.json.success }}
{{ steps.api.outputs.status >= 200 && steps.api.outputs.status < 300 }}
{{ steps.api.outputs.json.role === 'admin' || steps.api.outputs.json.role === 'moderator' }}

// Negação
{{ !steps.api.outputs.json.disabled }}  →  true se não desabilitado
{{ !(steps.api.outputs.status >= 400) }}  →  true se não é erro
```

### Data e Hora

```
// Data atual
{{ new Date() }}  →  objeto Date
{{ Date.now() }}  →  timestamp em milissegundos
{{ new Date().toISOString() }}  →  "2024-01-15T10:30:00.000Z"
{{ new Date().toLocaleString('pt-BR') }}  →  "15/01/2024 10:30:00"
{{ new Date().toLocaleDateString('pt-BR') }}  →  "15/01/2024"
{{ new Date().toLocaleTimeString('pt-BR') }}  →  "10:30:00"

// Componentes de data
{{ new Date().getFullYear() }}  →  2024
{{ new Date().getMonth() }}  →  0-11 (Janeiro = 0)
{{ new Date().getMonth() + 1 }}  →  1-12
{{ new Date().getDate() }}  →  1-31 (dia do mês)
{{ new Date().getDay() }}  →  0-6 (Domingo = 0)
{{ new Date().getHours() }}  →  0-23
{{ new Date().getMinutes() }}  →  0-59
{{ new Date().getSeconds() }}  →  0-59
{{ new Date().getMilliseconds() }}  →  0-999

// Criar datas
{{ new Date('2024-01-15') }}  →  data específica
{{ new Date(2024, 0, 15) }}  →  15 de Janeiro de 2024
{{ new Date(2024, 0, 15, 10, 30, 0) }}  →  com hora
{{ new Date(Date.now() + 86400000) }}  →  amanhã (+ 1 dia em ms)

// Cálculos com datas
{{ new Date().getTime() + (24 * 60 * 60 * 1000) }}  →  timestamp de amanhã
{{ new Date().getTime() - (7 * 24 * 60 * 60 * 1000) }}  →  timestamp de 7 dias atrás
{{ Math.floor((new Date('2024-12-31') - new Date()) / (1000 * 60 * 60 * 24)) }}  →  dias até fim do ano

// Formatação customizada
{{ `${new Date().getDate()}/${new Date().getMonth() + 1}/${new Date().getFullYear()}` }}  →  "15/1/2024"
{{ `${new Date().getHours()}:${new Date().getMinutes().toString().padStart(2, '0')}` }}  →  "10:05"

// Comparação de datas
{{ new Date(steps.api.outputs.json.expiresAt) > new Date() }}  →  true se não expirou
{{ new Date(steps.api.outputs.json.createdAt).getTime() > Date.now() - 86400000 }}  →  criado nas últimas 24h
```

### Conversão de Tipos

```
// Para String
{{ String(123) }}  →  "123"
{{ String(true) }}  →  "true"
{{ String(null) }}  →  "null"
{{ (123).toString() }}  →  "123"
{{ (123).toString(16) }}  →  "7b" (hexadecimal)

// Para Número
{{ Number('123') }}  →  123
{{ Number('12.5') }}  →  12.5
{{ Number(true) }}  →  1
{{ Number(false) }}  →  0
{{ parseInt('123') }}  →  123
{{ parseInt('123.45') }}  →  123
{{ parseFloat('123.45') }}  →  123.45
{{ parseInt('ff', 16) }}  →  255 (hexadecimal)
{{ parseInt('1010', 2) }}  →  10 (binário)

// Para Boolean
{{ Boolean(1) }}  →  true
{{ Boolean(0) }}  →  false
{{ Boolean('') }}  →  false
{{ Boolean('texto') }}  →  true
{{ Boolean(null) }}  →  false
{{ Boolean(undefined) }}  →  false
{{ !!value }}  →  converte para boolean

// Verificação de tipo
{{ typeof 'texto' }}  →  "string"
{{ typeof 123 }}  →  "number"
{{ typeof true }}  →  "boolean"
{{ typeof {} }}  →  "object"
{{ typeof [] }}  →  "object"
{{ typeof null }}  →  "object" (peculiaridade do JS)
{{ typeof undefined }}  →  "undefined"
{{ Array.isArray([1, 2]) }}  →  true
{{ Array.isArray('texto') }}  →  false
```

### Expressões Regulares (Regex)

```
// Testar padrão
{{ /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(steps.api.outputs.json.email) }}  →  valida email
{{ /^\d{3}\.\d{3}\.\d{3}-\d{2}$/.test(steps.api.outputs.json.cpf) }}  →  valida CPF formatado
{{ /^[0-9]+$/.test(steps.api.outputs.json.code) }}  →  apenas números
{{ /^[A-Z]/.test(steps.api.outputs.json.name) }}  →  começa com maiúscula

// Extrair com match
{{ steps.api.outputs.json.text.match(/\d+/g) }}  →  todos os números
{{ steps.api.outputs.json.url.match(/https?:\/\/([^\/]+)/)[1] }}  →  extrair domínio

// Substituir com regex
{{ steps.api.outputs.json.phone.replace(/\D/g, '') }}  →  remove não-dígitos
{{ steps.api.outputs.json.cpf.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, '$1.$2.$3-$4') }}  →  formata CPF
```

---

## Exemplos Práticos Avançados

### Validações

```
// Email válido
{{ /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(variables.email) }}

// CPF formatado
{{ /^\d{3}\.\d{3}\.\d{3}-\d{2}$/.test(variables.cpf) }}

// Telefone brasileiro
{{ /^\(\d{2}\) \d{4,5}-\d{4}$/.test(variables.telefone) }}

// CEP
{{ /^\d{5}-\d{3}$/.test(variables.cep) }}

// Senha forte (mínimo 8 caracteres, maiúscula, minúscula, número)
{{ variables.senha.length >= 8 && /[A-Z]/.test(variables.senha) && /[a-z]/.test(variables.senha) && /[0-9]/.test(variables.senha) }}

// URL válida
{{ /^https?:\/\/.+/.test(variables.url) }}

// Apenas letras
{{ /^[A-Za-zÀ-ÿ\s]+$/.test(variables.nome) }}

// Apenas números
{{ /^[0-9]+$/.test(variables.codigo) }}
```

### Formatações

```
// CPF
{{ variables.cpf.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, '$1.$2.$3-$4') }}  →  "123.456.789-00"

// CNPJ
{{ variables.cnpj.replace(/(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})/, '$1.$2.$3/$4-$5') }}  →  "12.345.678/0001-90"

// Telefone
{{ variables.telefone.replace(/(\d{2})(\d{5})(\d{4})/, '($1) $2-$3') }}  →  "(11) 98765-4321"

// CEP
{{ variables.cep.replace(/(\d{5})(\d{3})/, '$1-$2') }}  →  "12345-678"

// Moeda
{{ 'R$ ' + variables.preco.toFixed(2).replace('.', ',') }}  →  "R$ 10,50"
{{ 'R$ ' + variables.preco.toLocaleString('pt-BR', {minimumFractionDigits: 2}) }}  →  "R$ 1.234,56"

// Porcentagem
{{ (variables.valor * 100).toFixed(1) + '%' }}  →  "85.5%"
{{ Math.round(variables.acertos / variables.total * 100) + '%' }}  →  "85%"

// Número com separador de milhares
{{ variables.numero.toLocaleString('pt-BR') }}  →  "1.234.567"

// Primeira letra maiúscula
{{ variables.nome.charAt(0).toUpperCase() + variables.nome.slice(1).toLowerCase() }}  →  "João"

// Título (cada palavra com maiúscula)
{{ variables.titulo.split(' ').map(w => w.charAt(0).toUpperCase() + w.slice(1).toLowerCase()).join(' ') }}
```

### Manipulações Complexas

```
// Extrair domínio de email
{{ steps.api.outputs.json.email.split('@')[1] }}  →  "exemplo.com"

// Extrair domínio de URL
{{ new URL(variables.url).hostname }}  →  "www.exemplo.com"

// Iniciais do nome
{{ variables.nome.split(' ').map(n => n[0]).join('') }}  →  "JS" (João Silva)

// Contar palavras
{{ variables.texto.trim().split(/\s+/).length }}  →  quantidade de palavras

// Contar caracteres sem espaços
{{ variables.texto.replace(/\s/g, '').length }}

// Truncar texto
{{ variables.texto.length > 50 ? variables.texto.substring(0, 50) + '...' : variables.texto }}

// Remover acentos
{{ variables.texto.normalize('NFD').replace(/[\u0300-\u036f]/g, '') }}

// Slug (URL amigável)
{{ variables.titulo.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, '').replace(/[^a-z0-9]+/g, '-').replace(/^-|-$/g, '') }}

// Gerar ID único
{{ Date.now() + '-' + Math.random().toString(36).substr(2, 9) }}  →  "1705320000000-k7j3h2g1f"

// Hash simples (não criptográfico)
{{ variables.texto.split('').reduce((hash, char) => ((hash << 5) - hash) + char.charCodeAt(0), 0) }}

// Inverter string
{{ variables.texto.split('').reverse().join('') }}

// Remover duplicatas de string
{{ [...new Set(variables.texto.split(''))].join('') }}

// Capitalizar cada palavra
{{ variables.frase.replace(/\b\w/g, l => l.toUpperCase()) }}
```

### Cálculos e Estatísticas

```
// Média
{{ variables.notas.reduce((a, b) => a + b, 0) / variables.notas.length }}

// Mediana
{{ variables.notas.sort((a, b) => a - b)[Math.floor(variables.notas.length / 2)] }}

// Soma
{{ variables.valores.reduce((a, b) => a + b, 0) }}

// Máximo
{{ Math.max(...variables.valores) }}

// Mínimo
{{ Math.min(...variables.valores) }}

// Desvio padrão (simplificado)
{{ Math.sqrt(variables.valores.map(x => Math.pow(x - variables.valores.reduce((a,b) => a+b) / variables.valores.length, 2)).reduce((a,b) => a+b) / variables.valores.length) }}

// Porcentagem de crescimento
{{ ((variables.valorNovo - variables.valorAntigo) / variables.valorAntigo * 100).toFixed(1) + '%' }}

// Regra de três
{{ (variables.valor * variables.proporcao / 100) }}

// Juros simples
{{ variables.capital * variables.taxa * variables.tempo / 100 }}

// Dias entre datas
{{ Math.floor((new Date(variables.dataFim) - new Date(variables.dataInicio)) / (1000 * 60 * 60 * 24)) }}

// Idade a partir de data de nascimento
{{ Math.floor((new Date() - new Date(variables.dataNascimento)) / (1000 * 60 * 60 * 24 * 365.25)) }}

// IMC (Índice de Massa Corporal)
{{ (variables.peso / Math.pow(variables.altura, 2)).toFixed(1) }}
```

### Agrupamentos e Transformações

```
// Agrupar por propriedade
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.categoria] = acc[item.categoria] || [];
  acc[item.categoria].push(item);
  return acc;
}, {}) }}

// Contar ocorrências
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.status] = (acc[item.status] || 0) + 1;
  return acc;
}, {}) }}

// Somar por grupo
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.categoria] = (acc[item.categoria] || 0) + item.valor;
  return acc;
}, {}) }}

// Remover duplicatas
{{ [...new Set(steps.query.outputs.rows.map(r => r.email))] }}

// Ordenar objetos por múltiplos campos
{{ steps.query.outputs.rows.sort((a, b) => 
  a.categoria.localeCompare(b.categoria) || a.nome.localeCompare(b.nome)
) }}

// Filtrar e mapear em cadeia
{{ steps.query.outputs.rows
  .filter(r => r.ativo)
  .map(r => r.nome)
  .sort()
  .join(', ') }}

// Criar lookup/índice
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.id] = item;
  return acc;
}, {}) }}

// Achatar array de arrays
{{ steps.api.outputs.json.data.flatMap(item => item.tags) }}

// Particionar array (pares e ímpares)
{{ steps.query.outputs.rows.reduce((acc, item, i) => {
  acc[i % 2 === 0 ? 'pares' : 'impares'].push(item);
  return acc;
}, {pares: [], impares: []}) }}
```

---

## Uso nos Nós

### URL de Navegação

```
{{ variables.BASE_URL }}/login
{{ variables.BASE_URL }}/users/{{ steps['HTTP Request'].outputs.json.id }}
{{ variables.BASE_URL }}/products?category={{ steps.api.outputs.json.category }}&page={{ variables.page }}
```

### Texto para Fill/Type

```
{{ variables.TEST_EMAIL }}
{{ steps['Custom JavaScript'].outputs.result.email }}
test-{{ Date.now() }}@exemplo.com
{{ variables.FIRST_NAME }} {{ variables.LAST_NAME }}
```

### SQL com Dados Dinâmicos

```sql
SELECT * FROM orders WHERE user_id = '{{ steps['HTTP Request'].outputs.json.userId }}'

SELECT * FROM products 
WHERE category = '{{ variables.CATEGORY }}' 
AND price > {{ variables.MIN_PRICE }}

INSERT INTO logs (user_id, action, timestamp) 
VALUES ('{{ steps.login.outputs.json.userId }}', 'login', '{{ new Date().toISOString() }}')
```

> **Cuidado:** Use parâmetros ($1, ?) em vez de interpolação direta para prevenir SQL injection quando possível.

### Body de HTTP Request

```json
{
  "token": "{{ steps.login.outputs.json.token }}",
  "data": {
    "name": "{{ variables.TEST_NAME }}",
    "email": "{{ steps['Custom JavaScript'].outputs.result.email }}",
    "timestamp": "{{ new Date().toISOString() }}"
  }
}
```

> No exemplo acima, `steps.login` é um nó HTTP Request renomeado para "login".

### Headers de HTTP Request

```
Authorization: Bearer {{ steps.login.outputs.json.token }}
X-User-ID: {{ steps.api.outputs.json.userId }}
X-Request-ID: {{ Date.now() }}-{{ Math.random().toString(36).substr(2, 9) }}
```

### Condição do If

```
{{ steps['HTTP Request'].outputs.status === 200 }}
{{ steps['Postgres'].outputs.rowCount > 0 }}
{{ steps['Web Flow'].outputs.extracts.price < variables.MAX_PRICE }}
{{ steps.api.outputs.json.role === 'admin' || steps.api.outputs.json.role === 'moderator' }}
{{ steps.api.outputs.json.age >= 18 && steps.api.outputs.json.active }}
```

### Expressão do Switch

```
{{ steps.api.outputs.status }}
{{ steps.api.outputs.json.userType }}
{{ Math.floor(steps.api.outputs.json.score / 10) }}
```

### Items do Loop (For Each)

```
{{ steps['Postgres'].outputs.rows }}
{{ steps.api.outputs.json.users }}
{{ Array.from({length: 10}, (_, i) => i + 1) }}
{{ steps.api.outputs.json.data.filter(item => item.active) }}
```

### Condição do Loop (While)

```
{{ variables.contador < 10 }}
{{ steps.api.outputs.json.hasMore }}
{{ steps['Postgres'].outputs.rowCount > 0 }}
{{ variables.tentativas < variables.MAX_TENTATIVAS && !variables.sucesso }}
```

### Mensagem de Log

```
Usuário {{ steps['Postgres'].outputs.rows[0].name }} tem {{ steps['Postgres'].outputs.rows[0].orders }} pedidos

API respondeu com status {{ steps['HTTP Request'].outputs.status }}: {{ steps['HTTP Request'].outputs.json.message }}

Processados {{ _loopIndex + 1 }} de {{ _loopCount }} items

Total: R$ {{ steps.query.outputs.rows.reduce((sum, r) => sum + r.price, 0).toFixed(2) }}
```

### Set Variable (Definir Variável)

```
// Nome da variável: userId
// Valor:
{{ steps.login.outputs.json.user.id }}

// Nome da variável: fullName
// Valor:
{{ steps.api.outputs.json.firstName + ' ' + steps.api.outputs.json.lastName }}

// Nome da variável: processedData
// Valor:
{{ steps.query.outputs.rows.map(r => ({id: r.id, name: r.name.toUpperCase()})) }}

// Nome da variável: timestamp
// Valor:
{{ new Date().toISOString() }}
```

---

## Loops e Variáveis Especiais

Quando um nó está dentro de um loop, você tem acesso a variáveis especiais:

```
{{ _loopItem }}      // Item atual da iteração
{{ _loopIndex }}     // Índice atual (começa em 0)
{{ _loopCount }}     // Total de iterações (disponível após loop)
```

### Exemplos em Loops

```
// Mensagem com progresso
{{ 'Processando item ' + (_loopIndex + 1) + ' de ' + _loopCount }}

// Acessar propriedade do item atual
{{ _loopItem.nome }}
{{ _loopItem.preco * 1.1 }}

// Usar índice em cálculos
{{ 'Item #' + (_loopIndex + 1) + ': ' + _loopItem.name }}

// Condição baseada no índice
{{ _loopIndex === 0 ? 'Primeiro' : _loopIndex === _loopCount - 1 ? 'Último' : 'Meio' }}

// Usar item em URL
{{ variables.BASE_URL }}/api/users/{{ _loopItem.id }}

// Usar item em SQL
SELECT * FROM orders WHERE user_id = '{{ _loopItem.userId }}'
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

// Fallback com OR
{{ steps['HTTP Request'].outputs.json.name || 'N/A' }}

// Fallback com nullish coalescing
{{ steps.api.outputs.json.count ?? 0 }}

// Verificação antes de acessar
{{ steps.api.outputs.json.user ? steps.api.outputs.json.user.name : 'Sem usuário' }}

// Verificação de array vazio
{{ steps.query.outputs.rows.length > 0 ? steps.query.outputs.rows[0].name : 'Nenhum resultado' }}
```

> **Importante:** Se uma expressão referenciar um nó ou variável que não existe (ex: erro de digitação), o nó falhará com uma mensagem de erro clara indicando qual expressão falhou, em vez de causar comportamento inesperado.

---

## Segurança

O sistema de expressões roda em um **ambiente sandbox** com restrições:

- **Sem acesso** a `require`, `import`, `process`
- **Sem acesso** ao sistema de arquivos
- **Sem acesso** a rede (use nó HTTP Request para isso)
- Apenas `steps`, `variables` estão disponíveis no contexto

---

## Dicas e Boas Práticas

### 1. Renomeie Nós com Rótulos Simples

```
// ❌ Difícil de ler
{{ steps['HTTP Request'].outputs.json.token }}

// ✅ Mais limpo (renomeie o nó para "login")
{{ steps.login.outputs.json.token }}
```

### 2. Use Optional Chaining para Evitar Erros

```
// ❌ Pode dar erro se user não existir
{{ steps.api.outputs.json.user.name }}

// ✅ Retorna undefined se não existir
{{ steps.api.outputs.json?.user?.name }}

// ✅ Com valor padrão
{{ steps.api.outputs.json?.user?.name ?? 'Sem nome' }}
```

### 3. Use Interpolação para Construir URLs e Mensagens

```
// ✅ URL dinâmica
{{ variables.BASE_URL }}/api/users/{{ steps.api.outputs.json.id }}

// ✅ Mensagem formatada
Status: {{ steps.api.outputs.status }} - {{ steps.api.outputs.json.message }}
```

### 4. Use Expressão Única para Manter Tipos

```
// ✅ Retorna array (expressão única)
{{ steps.query.outputs.rows }}

// ❌ Retorna string (interpolação)
Rows: {{ steps.query.outputs.rows }}
```

### 5. Adicione Nó Log para Depuração

```
// Log para inspecionar valores
{{ JSON.stringify(steps.api.outputs.json, null, 2) }}

// Log de variáveis
Variables: {{ JSON.stringify(variables) }}

// Log de item do loop
Item atual: {{ JSON.stringify(_loopItem) }}
```

### 6. Use Fallbacks para Dados Opcionais

```
{{ steps.api.outputs.json.name || 'Anônimo' }}
{{ steps.api.outputs.json.config || {} }}
{{ steps.api.outputs.json.items || [] }}
```

### 7. Para Lógica Complexa, Use Custom JavaScript

Se a expressão está ficando muito complexa, considere:
- Usar um nó **Custom JavaScript** para processar os dados
- Dividir a lógica em múltiplos nós **Set Variable**
- Usar nós de controle (If, Switch, Loop)

```
// ❌ Expressão muito complexa
{{ steps.query.outputs.rows.filter(r => r.active && r.age > 18).map(r => ({...r, fullName: r.firstName + ' ' + r.lastName})).sort((a, b) => a.fullName.localeCompare(b.fullName)) }}

// ✅ Use Custom JavaScript
// No nó Custom JavaScript:
const activeAdults = ctx.steps.query.outputs.rows
  .filter(r => r.active && r.age > 18)
  .map(r => ({...r, fullName: `${r.firstName} ${r.lastName}`}))
  .sort((a, b) => a.fullName.localeCompare(b.fullName));

return { processedUsers: activeAdults };

// Depois referencie:
{{ steps.transform.outputs.processedUsers }}
```

### 8. Converta Tipos Quando Necessário

```
// ❌ Concatenação indesejada
{{ '10' + '5' }}  // '105'

// ✅ Soma numérica
{{ Number('10') + Number('5') }}  // 15
{{ parseInt('10') + parseInt('5') }}  // 15
```

### 9. Valide Antes de Usar

```
{{ Array.isArray(variables.lista) && variables.lista.length > 0 
  ? variables.lista[0] 
  : 'Lista vazia' }}

{{ typeof steps.api.outputs.json.data === 'object' 
  ? JSON.stringify(steps.api.outputs.json.data) 
  : 'Dados inválidos' }}
```

### 10. Use Template Strings para Legibilidade

```
// ❌ Difícil de ler
{{ 'Nome: ' + variables.nome + ', Idade: ' + variables.idade + ', Email: ' + variables.email }}

// ✅ Mais legível
{{ `Nome: ${variables.nome}, Idade: ${variables.idade}, Email: ${variables.email}` }}
```

---

## Limitações

- Expressões são avaliadas em um ambiente sandbox seguro
- Não é possível acessar APIs do navegador (window, document, etc)
- Não é possível fazer requisições HTTP diretamente (use nós específicos)
- Não é possível importar módulos externos
- Funções assíncronas (async/await) não são suportadas em expressões
- Não é possível modificar o contexto global ou variáveis de outros nós

---

## Conclusão

O sistema de expressões do QANode oferece todo o poder do JavaScript para criar fluxos dinâmicos e inteligentes. Com a sintaxe `{{ }}`, você pode:

- Acessar dados de nós anteriores e variáveis
- Realizar cálculos e transformações complexas
- Validar e formatar dados
- Criar lógica condicional
- Manipular strings, arrays e objetos
- Trabalhar com datas e números

Lembre-se de sempre usar `{{ }}` para delimitar suas expressões e aproveite todo o poder do JavaScript para criar testes robustos e eficientes!
