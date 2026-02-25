# Sistema de Expresiones

QANode utiliza un sistema de **expresiones** con la sintaxis `{{ }}` que permite inyectar datos dinámicos en cualquier campo de configuración de los nodos. Con expresiones, puedes acceder a outputs de nodos anteriores, variables globales y variables de entorno.

---

## Sintaxis Básica

```
{{ expresión }}
```

Todo lo que está dentro de `{{ }}` se evalúa como JavaScript en el contexto de ejecución del flujo.

---

## Contexto Disponible

### `steps` — Outputs de Nodos Anteriores

Accede a los datos producidos por cualquier nodo que ya fue ejecutado:

```
{{ steps["Nombre del Nodo"].outputs.propiedad }}
```

| Componente | Descripción |
|------------|-------------|
| `steps` | Objeto con todos los nodos ejecutados |
| `["Nombre del Nodo"]` | Identificador del nodo — ver reglas abajo |
| `.outputs` | Datos de salida del nodo |
| `.propiedad` | Campo específico del output |

#### Cómo funciona el identificador en steps

El nombre usado en `steps` es el **identificador** del nodo. Por defecto, todo nodo usa su **tipo** en minúsculas como identificador (ej: `if`, `http-request`, `postgres-query`). Puedes cambiar el identificador definiendo un **rótulo personalizado** en el panel de propiedades.

**Regla de notación:**

| Situación | Identificador | Notación | Ejemplo |
|-----------|---------------|----------|---------|
| Tipo por defecto simple | `if`, `merge`, `loop` | Punto `.` | `steps.if.outputs.result` |
| Tipo por defecto con guion | `http-request`, `postgres-query` | Corchetes `["..."]` | `steps["http-request"].outputs.status` |
| Rótulo personalizado simple | `login`, `api`, `query` | Punto `.` | `steps.login.outputs.json.token` |
| Rótulo personalizado con espacio | `Login API`, `Consulta DB` | Corchetes `["..."]` | `steps["Consulta DB"].outputs.rows` |

> **Consejo:** Si renombras el rótulo del nodo a una sola palabra sin espacios (ej: `login`, `api`, `query`), puedes usar la notación con punto, que es más simple: `steps.login.outputs.json.token`

**Nodos duplicados:** Si hay dos nodos con el mismo identificador, el segundo recibe un sufijo numérico (ej: "http-request 2", "http-request 3").

#### Identificadores por defecto de los nodos (tipos)

| Tipo de Nodo | Identificador por Defecto | Notación |
|--------------|--------------------------|----------|
| If | `if` | `steps.if` |
| Switch | `switch` | `steps.switch` |
| Loop | `loop` | `steps.loop` |
| Merge | `merge` | `steps.merge` |
| HTTP Request | `http-request` | `steps["http-request"]` |
| Postgres | `postgres-query` | `steps["postgres-query"]` |
| MySQL | `mysql-query` | `steps["mysql-query"]` |
| MariaDB | `mariadb-query` | `steps["mariadb-query"]` |
| Oracle | `oracle-query` | `steps["oracle-query"]` |
| MongoDB | `mongodb-query` | `steps["mongodb-query"]` |
| Web Flow | `web-flow` | `steps["web-flow"]` |
| Smart Locators | `smart-locators` | `steps["smart-locators"]` |
| SSH Command | `ssh-command` | `steps["ssh-command"]` |
| Custom JavaScript | `custom-js` | `steps["custom-js"]` |
| Set Variable | `set-variable` | `steps["set-variable"]` |
| Log | `log` | `steps.log` |

**Ejemplos con identificadores por defecto:**

```
// Estado HTTP (tipo "http-request" — requiere corchetes)
{{ steps["http-request"].outputs.status }}  →  200

// Cuerpo JSON de la respuesta
{{ steps["http-request"].outputs.json.name }}  →  "João"

// Array de base de datos
{{ steps["postgres-query"].outputs.rows[0].email }}  →  "joao@exemplo.com"

// Cantidad de registros
{{ steps["postgres-query"].outputs.rowCount }}  →  5

// Texto extraído de página web
{{ steps["web-flow"].outputs.extracts.titulo }}  →  "Dashboard"

// Resultado de aserción
{{ steps["web-flow"].outputs.asserts.loginOk }}  →  true

// Session ID del navegador
{{ steps["smart-locators"].outputs.sessionId }}  →  "abc-123"

// Salida SSH
{{ steps["ssh-command"].outputs.stdout }}  →  "active (running)"
{{ steps["ssh-command"].outputs.exitCode }}  →  0

// Nodo If
{{ steps.if.outputs._branch }}  →  "true" o "false"

// Nodo Loop
{{ steps.loop.outputs._loopCount }}  →  10
```

**Ejemplos con rótulos personalizados (sin espacio — notación con punto):**

```
// Si renombras el nodo HTTP Request a "api":
{{ steps.api.outputs.status }}  →  200
{{ steps.api.outputs.json.name }}  →  "João"

// Si renombras el nodo Custom JavaScript a "transform":
{{ steps.transform.outputs.result }}  →  { "total": 42 }

// Si renombras el nodo Postgres a "query":
{{ steps.query.outputs.rows[0].email }}  →  "joao@exemplo.com"
```

### `variables` — Variables Globales

Accede a variables registradas en el sistema:

```
{{ variables.NOMBRE_DE_VARIABLE }}
```

**Ejemplos:**

```
// String
{{ variables.API_URL }}  →  "https://api.exemplo.com"

// Número
{{ variables.DEFAULT_TIMEOUT }}  →  30000

// Boolean
{{ variables.FEATURE_FLAG }}  →  true

// JSON (acceso a propiedades)
{{ variables.CONFIG.retries }}  →  3
{{ variables.CONFIG.timeout }}  →  5000
```

---

## Modos de Uso

### Expresión Única (Valor Directo)

Cuando el campo completo es una expresión, el valor retornado mantiene su tipo original (número, objeto, array, boolean):

```
Campo: {{ steps["postgres-query"].outputs.rows }}
Resultado: [{ "id": 1, "name": "João" }, { "id": 2, "name": "Maria" }]  (array)

Campo: {{ steps["http-request"].outputs.status }}
Resultado: 200  (número)

Campo: {{ steps["web-flow"].outputs.asserts.loginOk }}
Resultado: true  (boolean)
```

### Interpolación (Dentro de Texto)

Cuando la expresión está mezclada con texto, el resultado es siempre un string:

```
Campo: Hola, {{ steps["postgres-query"].outputs.rows[0].name }}!
Resultado: "Hola, João!"  (string)

Campo: {{ variables.API_URL }}/api/users/{{ steps["http-request"].outputs.json.id }}
Resultado: "https://api.exemplo.com/api/users/42"  (string)

Campo: Estado: {{ steps["http-request"].outputs.status }} - {{ steps["http-request"].outputs.json.message }}
Resultado: "Estado: 200 - OK"  (string)
```

---

## Expresiones JavaScript

Dentro de `{{ }}`, cualquier expresión JavaScript válida funciona:

### Operaciones con Strings

```
// Mayúsculas (nodo Web Flow renombrado a "extract")
{{ steps.extract.outputs.extracts.nome.toUpperCase() }}  →  "JOÃO"

// Minúsculas
{{ steps.extract.outputs.extracts.nome.toLowerCase() }}  →  "joão"

// Primera letra mayúscula
{{ steps.extract.outputs.extracts.nome.charAt(0).toUpperCase() + steps.extract.outputs.extracts.nome.slice(1).toLowerCase() }}  →  "João"

// Substring
{{ steps.extract.outputs.extracts.texto.substring(0, 100) }}  →  "Primeros 100 caracteres..."

// Slice (permite índices negativos)
{{ steps.extract.outputs.extracts.texto.slice(0, 50) }}  →  "Primeros 50 caracteres"
{{ steps.extract.outputs.extracts.texto.slice(-10) }}  →  "Últimos 10"

// Reemplazo
{{ steps.extract.outputs.extracts.url.replace("http:", "https:") }}
{{ steps.extract.outputs.extracts.texto.replaceAll("a", "A") }}  →  reemplaza todas las ocurrencias

// Concatenación (usando rótulo por defecto con corchetes)
{{ steps["postgres-query"].outputs.rows[0].firstName + " " + steps["postgres-query"].outputs.rows[0].lastName }}

// Template strings
{{ `${steps["postgres-query"].outputs.rows[0].firstName} ${steps["postgres-query"].outputs.rows[0].lastName}` }}

// Longitud
{{ steps.extract.outputs.extracts.texto.length }}  →  150

// Verificar si contiene
{{ steps.extract.outputs.extracts.email.includes("@") }}  →  true
{{ steps.extract.outputs.extracts.url.startsWith("https://") }}  →  true
{{ steps.extract.outputs.extracts.arquivo.endsWith(".pdf") }}  →  true

// Buscar posición
{{ steps.extract.outputs.extracts.email.indexOf("@") }}  →  5
{{ steps.extract.outputs.extracts.texto.lastIndexOf("palavra") }}  →  última ocurrencia

// Dividir string
{{ steps.extract.outputs.extracts.tags.split(",") }}  →  ["tag1", "tag2", "tag3"]
{{ steps.extract.outputs.extracts.nome.split("") }}  →  ["J", "o", "ã", "o"]

// Eliminar espacios
{{ steps.extract.outputs.extracts.nome.trim() }}  →  elimina espacios al inicio y al final
{{ steps.extract.outputs.extracts.nome.trimStart() }}  →  elimina espacios al inicio
{{ steps.extract.outputs.extracts.nome.trimEnd() }}  →  elimina espacios al final

// Repetir
{{ "abc".repeat(3) }}  →  "abcabcabc"

// Rellenar con ceros
{{ steps.api.outputs.json.id.toString().padStart(5, "0") }}  →  "00042"
{{ steps.api.outputs.json.code.padEnd(10, "-") }}  →  "ABC-------"

// Extraer carácter
{{ steps.extract.outputs.extracts.nome.charAt(0) }}  →  "J"
{{ steps.extract.outputs.extracts.nome[0] }}  →  "J"
{{ steps.extract.outputs.extracts.nome.at(-1) }}  →  último carácter
```

### Operaciones con Números

```
// Cálculo (nodo HTTP Request renombrado a "cart")
{{ steps.cart.outputs.json.subtotal * 1.1 }}  →  agrega 10%
{{ steps.cart.outputs.json.price * steps.cart.outputs.json.quantity }}  →  total

// Redondeo (nodo renombrado a "api")
{{ Math.round(steps.api.outputs.json.score) }}  →  redondea al entero más cercano
{{ Math.ceil(steps.api.outputs.json.score) }}  →  redondea hacia arriba
{{ Math.floor(steps.api.outputs.json.score) }}  →  redondea hacia abajo
{{ Math.trunc(steps.api.outputs.json.score) }}  →  elimina decimales

// Decimales
{{ Math.round(steps.api.outputs.json.score * 100) / 100 }}  →  2 decimales
{{ steps.api.outputs.json.price.toFixed(2) }}  →  "10.50" (string)
{{ parseFloat(steps.api.outputs.json.price.toFixed(2)) }}  →  10.5 (número)

// Formateo (nodo Postgres renombrado a "query")
{{ steps.query.outputs.rowCount.toString().padStart(5, "0") }}  →  "00042"

// Valor absoluto
{{ Math.abs(steps.api.outputs.json.balance) }}  →  siempre positivo

// Máximo y mínimo
{{ Math.max(10, 20, 30) }}  →  30
{{ Math.min(10, 20, 30) }}  →  10
{{ Math.max(...steps.query.outputs.rows.map(r => r.age)) }}  →  edad máxima

// Potencia y raíz
{{ Math.pow(2, 3) }}  →  8 (2³)
{{ 2 ** 3 }}  →  8 (2³)
{{ Math.sqrt(16) }}  →  4 (raíz cuadrada)
{{ Math.cbrt(27) }}  →  3 (raíz cúbica)

// Número aleatorio
{{ Math.random() }}  →  0.0 a 1.0
{{ Math.floor(Math.random() * 100) }}  →  0 a 99
{{ Math.floor(Math.random() * 10) + 1 }}  →  1 a 10

// Signo
{{ Math.sign(steps.api.outputs.json.balance) }}  →  -1, 0 o 1

// Conversión
{{ parseInt("42") }}  →  42
{{ parseInt("42.7") }}  →  42
{{ parseFloat("42.7") }}  →  42.7
{{ Number("42.7") }}  →  42.7
{{ parseInt("ff", 16) }}  →  255 (hexadecimal)

// Verificación
{{ Number.isNaN(steps.api.outputs.json.value) }}  →  true si NaN
{{ Number.isFinite(steps.api.outputs.json.value) }}  →  true si finito
{{ Number.isInteger(steps.api.outputs.json.value) }}  →  true si entero

// Constantes
{{ Math.PI }}  →  3.141592653589793
{{ Math.E }}  →  2.718281828459045
```

### Operaciones con Arrays

```
// Longitud
{{ steps["postgres-query"].outputs.rows.length }}  →  5

// Primero/Último
{{ steps["postgres-query"].outputs.rows[0].name }}  →  primero
{{ steps["postgres-query"].outputs.rows[steps["postgres-query"].outputs.rows.length - 1].name }}  →  último
{{ steps["postgres-query"].outputs.rows.at(-1).name }}  →  último (más simple)
{{ steps["postgres-query"].outputs.rows.at(-2).name }}  →  penúltimo

// Verificar si contiene
{{ steps["postgres-query"].outputs.rows.some(r => r.email === "joao@exemplo.com") }}  →  true si alguno
{{ steps["postgres-query"].outputs.rows.every(r => r.active) }}  →  true si todos
{{ steps["postgres-query"].outputs.rows.includes("valor") }}  →  true si contiene valor exacto

// Buscar
{{ steps["postgres-query"].outputs.rows.find(r => r.id === 42) }}  →  primer objeto que cumple
{{ steps["postgres-query"].outputs.rows.findIndex(r => r.id === 42) }}  →  índice del primero
{{ steps["postgres-query"].outputs.rows.indexOf("valor") }}  →  índice de valor primitivo

// Filtrar
{{ steps["postgres-query"].outputs.rows.filter(r => r.active) }}  →  solo activos
{{ steps["postgres-query"].outputs.rows.filter(r => r.age > 18) }}  →  mayores de 18
{{ steps["postgres-query"].outputs.rows.filter(r => r.active).length }}  →  contar activos
{{ steps["postgres-query"].outputs.rows.filter(r => r.email.includes("@gmail.com")) }}  →  solo Gmail

// Mapear (transformar)
{{ steps["postgres-query"].outputs.rows.map(r => r.email) }}  →  ["email1", "email2", ...]
{{ steps["postgres-query"].outputs.rows.map(r => r.email).join(", ") }}  →  "email1, email2, ..."
{{ steps["postgres-query"].outputs.rows.map(r => r.name.toUpperCase()) }}  →  nombres en mayúsculas
{{ steps["postgres-query"].outputs.rows.map((r, i) => `${i + 1}. ${r.name}`) }}  →  lista numerada

// Reducir (agregar)
{{ steps["postgres-query"].outputs.rows.reduce((sum, r) => sum + r.price, 0) }}  →  suma de precios
{{ steps["postgres-query"].outputs.rows.reduce((max, r) => r.age > max ? r.age : max, 0) }}  →  edad máxima
{{ steps["postgres-query"].outputs.rows.reduce((acc, r) => acc + r.name + ", ", "") }}  →  concatenar nombres

// Ordenar
{{ steps["postgres-query"].outputs.rows.sort((a, b) => a.name.localeCompare(b.name)) }}  →  orden alfabético
{{ steps["postgres-query"].outputs.rows.sort((a, b) => a.age - b.age) }}  →  orden ascendente por edad
{{ steps["postgres-query"].outputs.rows.sort((a, b) => b.price - a.price) }}  →  orden descendente por precio

// Invertir
{{ steps["postgres-query"].outputs.rows.reverse() }}  →  invierte el orden
{{ steps["postgres-query"].outputs.rows.slice().reverse() }}  →  invierte sin modificar el original

// Cortar
{{ steps["postgres-query"].outputs.rows.slice(0, 10) }}  →  primeros 10
{{ steps["postgres-query"].outputs.rows.slice(-5) }}  →  últimos 5
{{ steps["postgres-query"].outputs.rows.slice(2, 5) }}  →  del índice 2 al 4

// Concatenar
{{ steps.query1.outputs.rows.concat(steps.query2.outputs.rows) }}  →  une dos arrays
{{ [...steps.query1.outputs.rows, ...steps.query2.outputs.rows] }}  →  spread operator

// Aplanar (flat)
{{ [[1, 2], [3, 4]].flat() }}  →  [1, 2, 3, 4]
{{ [[1, [2, [3]]]].flat(2) }}  →  [1, 2, 3] (2 niveles)
{{ steps.api.outputs.json.data.flatMap(item => item.tags) }}  →  todos los tags de todos los items

// Unir en string
{{ steps["postgres-query"].outputs.rows.map(r => r.name).join(", ") }}  →  "João, Maria, Pedro"
{{ steps["postgres-query"].outputs.rows.map(r => r.name).join(" | ") }}  →  "João | Maria | Pedro"

// Eliminar duplicados
{{ [...new Set(steps["postgres-query"].outputs.rows.map(r => r.city))] }}  →  ciudades únicas

// Crear array
{{ Array.from({length: 5}, (_, i) => i) }}  →  [0, 1, 2, 3, 4]
{{ Array.from({length: 5}, (_, i) => i + 1) }}  →  [1, 2, 3, 4, 5]
{{ Array(3).fill(0) }}  →  [0, 0, 0]
{{ Array(3).fill("x") }}  →  ["x", "x", "x"]

// Verificar si es array
{{ Array.isArray(steps.api.outputs.json.data) }}  →  true
```

### Operaciones con Objetos

```
// Acceso profundo (nodo renombrado a "api")
{{ steps.api.outputs.json.data.user.address.city }}

// O usando rótulo por defecto con corchetes:
{{ steps["http-request"].outputs.json.data.user.address.city }}

// Optional chaining (evita error si la propiedad no existe)
{{ steps.api.outputs.json?.data?.user?.name }}  →  undefined si no existe
{{ steps.api.outputs.json?.data?.user?.name ?? "Sin nombre" }}  →  valor por defecto

// Claves (keys)
{{ Object.keys(steps.api.outputs.json) }}  →  ["id", "name", "email"]
{{ Object.keys(steps.api.outputs.json).length }}  →  cantidad de propiedades
{{ Object.keys(steps.api.outputs.json).join(", ") }}  →  "id, name, email"

// Valores (values)
{{ Object.values(steps.api.outputs.json) }}  →  [42, "João", "joao@exemplo.com"]

// Entradas (entries)
{{ Object.entries(steps.api.outputs.json) }}  →  [["id", 42], ["name", "João"], ...]
{{ Object.entries(steps.api.outputs.json).map(([k, v]) => `${k}: ${v}`).join(", ") }}

// Verificar propiedad
{{ "email" in steps.api.outputs.json }}  →  true si la propiedad existe
{{ Object.hasOwn(steps.api.outputs.json, "email") }}  →  true si es propiedad propia

// Fusionar objetos
{{ Object.assign({}, steps.api.outputs.json, {newProp: "value"}) }}
{{ {...steps.api.outputs.json, newProp: "value"} }}  →  spread operator

// Stringify (convertir a JSON string)
{{ JSON.stringify(steps.api.outputs.json) }}  →  '{"id":42,"name":"João"}'
{{ JSON.stringify(steps.api.outputs.json, null, 2) }}  →  formateado con indentación

// Parse (convertir de JSON string)
{{ JSON.parse(steps.api.outputs.body) }}  →  objeto JavaScript
{{ JSON.parse(steps.api.outputs.body).data.user.name }}  →  acceso después del parse

// Extraer propiedades específicas
{{ (({id, name}) => ({id, name}))(steps.api.outputs.json) }}  →  solo id y name

// Contar propiedades
{{ Object.keys(steps.api.outputs.json).length }}  →  cantidad de campos

// Transformar objeto en array
{{ Object.entries(steps.api.outputs.json).map(([k, v]) => ({key: k, value: v})) }}
```

### Condicionales (Ternario)

```
// Valor condicional
{{ steps["http-request"].outputs.status === 200 ? "Éxito" : "Fallo" }}

// Múltiples condiciones
{{ steps.api.outputs.json.age >= 18 ? "Adulto" : "Menor" }}
{{ steps.api.outputs.json.score >= 90 ? "A" : steps.api.outputs.json.score >= 80 ? "B" : "C" }}

// Con operadores lógicos
{{ (steps.api.outputs.json.age >= 18 && steps.api.outputs.json.active) ? "Permitido" : "Denegado" }}
{{ (steps.api.outputs.status === 200 || steps.api.outputs.status === 201) ? "OK" : "Error" }}

// Fallback para null/undefined
{{ steps["web-flow"].outputs.extracts.titulo || "Sin título" }}
{{ steps.api.outputs.json.name || "Anónimo" }}
{{ steps.api.outputs.json.config || {} }}  →  objeto vacío si no existe

// Nullish coalescing (solo null/undefined, no valores falsy)
{{ steps.api.outputs.json.count ?? 0 }}  →  0 solo si null/undefined
{{ steps.api.outputs.json.name ?? "Sin nombre" }}

// Verificación de existencia
{{ steps.api.outputs.json.email ? "Email: " + steps.api.outputs.json.email : "Sin email" }}

// Formateo condicional
{{ steps.api.outputs.json.price > 1000 ? "$ " + (steps.api.outputs.json.price / 1000).toFixed(1) + "k" : "$ " + steps.api.outputs.json.price }}
```

### Operaciones Lógicas y de Comparación

```
// Comparación
{{ 5 > 3 }}  →  true
{{ 10 < 20 }}  →  true
{{ 5 >= 5 }}  →  true
{{ 3 <= 2 }}  →  false
{{ "abc" === "abc" }}  →  true (igualdad estricta)
{{ 10 !== 5 }}  →  true (diferente)
{{ "5" == 5 }}  →  true (igualdad laxa)
{{ "5" === 5 }}  →  false (tipos diferentes)

// Operadores lógicos
{{ true && false }}  →  false (Y lógico)
{{ true || false }}  →  true (O lógico)
{{ !true }}  →  false (NO lógico)
{{ !!value }}  →  convierte a boolean

// Comparaciones con datos
{{ steps.api.outputs.status === 200 }}  →  true/false
{{ steps.api.outputs.json.age > 18 }}  →  true/false
{{ steps["postgres-query"].outputs.rowCount > 0 }}  →  true/false
{{ steps.api.outputs.json.email.includes("@") }}  →  true/false

// Múltiples condiciones
{{ steps.api.outputs.status === 200 && steps.api.outputs.json.success }}
{{ steps.api.outputs.status >= 200 && steps.api.outputs.status < 300 }}
{{ steps.api.outputs.json.role === "admin" || steps.api.outputs.json.role === "moderator" }}

// Negación
{{ !steps.api.outputs.json.disabled }}  →  true si no está deshabilitado
{{ !(steps.api.outputs.status >= 400) }}  →  true si no es un error
```

### Fecha y Hora

```
// Fecha actual
{{ new Date() }}  →  objeto Date
{{ Date.now() }}  →  timestamp en milisegundos
{{ new Date().toISOString() }}  →  "2024-01-15T10:30:00.000Z"
{{ new Date().toLocaleString("pt-BR") }}  →  "15/01/2024 10:30:00"
{{ new Date().toLocaleDateString("pt-BR") }}  →  "15/01/2024"
{{ new Date().toLocaleTimeString("pt-BR") }}  →  "10:30:00"

// Componentes de fecha
{{ new Date().getFullYear() }}  →  2024
{{ new Date().getMonth() }}  →  0-11 (Enero = 0)
{{ new Date().getMonth() + 1 }}  →  1-12
{{ new Date().getDate() }}  →  1-31 (día del mes)
{{ new Date().getDay() }}  →  0-6 (Domingo = 0)
{{ new Date().getHours() }}  →  0-23
{{ new Date().getMinutes() }}  →  0-59
{{ new Date().getSeconds() }}  →  0-59
{{ new Date().getMilliseconds() }}  →  0-999

// Crear fechas
{{ new Date("2024-01-15") }}  →  fecha específica
{{ new Date(2024, 0, 15) }}  →  15 de enero de 2024
{{ new Date(2024, 0, 15, 10, 30, 0) }}  →  con hora
{{ new Date(Date.now() + 86400000) }}  →  mañana (+ 1 día en ms)

// Cálculos con fechas
{{ new Date().getTime() + (24 * 60 * 60 * 1000) }}  →  timestamp de mañana
{{ new Date().getTime() - (7 * 24 * 60 * 60 * 1000) }}  →  timestamp de hace 7 días
{{ Math.floor((new Date("2024-12-31") - new Date()) / (1000 * 60 * 60 * 24)) }}  →  días hasta fin de año

// Formateo personalizado
{{ `${new Date().getDate()}/${new Date().getMonth() + 1}/${new Date().getFullYear()}` }}  →  "15/1/2024"
{{ `${new Date().getHours()}:${new Date().getMinutes().toString().padStart(2, "0")}` }}  →  "10:05"

// Comparación de fechas
{{ new Date(steps.api.outputs.json.expiresAt) > new Date() }}  →  true si no expiró
{{ new Date(steps.api.outputs.json.createdAt).getTime() > Date.now() - 86400000 }}  →  creado en las últimas 24h
```

### Conversión de Tipos

```
// A String
{{ String(123) }}  →  "123"
{{ String(true) }}  →  "true"
{{ String(null) }}  →  "null"
{{ (123).toString() }}  →  "123"
{{ (123).toString(16) }}  →  "7b" (hexadecimal)

// A Número
{{ Number("123") }}  →  123
{{ Number("12.5") }}  →  12.5
{{ Number(true) }}  →  1
{{ Number(false) }}  →  0
{{ parseInt("123") }}  →  123
{{ parseInt("123.45") }}  →  123
{{ parseFloat("123.45") }}  →  123.45
{{ parseInt("ff", 16) }}  →  255 (hexadecimal)
{{ parseInt("1010", 2) }}  →  10 (binario)

// A Boolean
{{ Boolean(1) }}  →  true
{{ Boolean(0) }}  →  false
{{ Boolean("") }}  →  false
{{ Boolean("texto") }}  →  true
{{ Boolean(null) }}  →  false
{{ Boolean(undefined) }}  →  false
{{ !!value }}  →  convierte a boolean

// Verificación de tipo
{{ typeof "texto" }}  →  "string"
{{ typeof 123 }}  →  "number"
{{ typeof true }}  →  "boolean"
{{ typeof {} }}  →  "object"
{{ typeof [] }}  →  "object"
{{ typeof null }}  →  "object" (peculiaridad de JS)
{{ typeof undefined }}  →  "undefined"
{{ Array.isArray([1, 2]) }}  →  true
{{ Array.isArray("texto") }}  →  false
```

### Expresiones Regulares (Regex)

```
// Probar patrón
{{ /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(steps.api.outputs.json.email) }}  →  valida email
{{ /^\d{3}\.\d{3}\.\d{3}-\d{2}$/.test(steps.api.outputs.json.cpf) }}  →  valida CPF formateado
{{ /^[0-9]+$/.test(steps.api.outputs.json.code) }}  →  solo números
{{ /^[A-Z]/.test(steps.api.outputs.json.name) }}  →  comienza con mayúscula

// Extraer con match
{{ steps.api.outputs.json.text.match(/\d+/g) }}  →  todos los números
{{ steps.api.outputs.json.url.match(/https?:\/\/([^\/]+)/)[1] }}  →  extraer dominio

// Reemplazar con regex
{{ steps.api.outputs.json.phone.replace(/\D/g, "") }}  →  elimina no-dígitos
{{ steps.api.outputs.json.cpf.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, "$1.$2.$3-$4") }}  →  formatea CPF
```

---

## Ejemplos Prácticos Avanzados

### Validaciones

```
// Email válido
{{ /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(variables.email) }}

// CPF formateado
{{ /^\d{3}\.\d{3}\.\d{3}-\d{2}$/.test(variables.cpf) }}

// Teléfono brasileño
{{ /^\(\d{2}\) \d{4,5}-\d{4}$/.test(variables.telefone) }}

// Código postal (CEP)
{{ /^\d{5}-\d{3}$/.test(variables.cep) }}

// Contraseña fuerte (mínimo 8 caracteres, mayúscula, minúscula, número)
{{ variables.senha.length >= 8 && /[A-Z]/.test(variables.senha) && /[a-z]/.test(variables.senha) && /[0-9]/.test(variables.senha) }}

// URL válida
{{ /^https?:\/\/.+/.test(variables.url) }}

// Solo letras
{{ /^[A-Za-zÀ-ÿ\s]+$/.test(variables.nome) }}

// Solo números
{{ /^[0-9]+$/.test(variables.codigo) }}
```

### Formateos

```
// CPF
{{ variables.cpf.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, "$1.$2.$3-$4") }}  →  "123.456.789-00"

// CNPJ
{{ variables.cnpj.replace(/(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})/, "$1.$2.$3/$4-$5") }}  →  "12.345.678/0001-90"

// Teléfono
{{ variables.telefone.replace(/(\d{2})(\d{5})(\d{4})/, "($1) $2-$3") }}  →  "(11) 98765-4321"

// Código postal (CEP)
{{ variables.cep.replace(/(\d{5})(\d{3})/, "$1-$2") }}  →  "12345-678"

// Moneda
{{ "R$ " + variables.preco.toFixed(2).replace(".", ",") }}  →  "R$ 10,50"
{{ "R$ " + variables.preco.toLocaleString("pt-BR", {minimumFractionDigits: 2}) }}  →  "R$ 1.234,56"

// Porcentaje
{{ (variables.valor * 100).toFixed(1) + "%" }}  →  "85.5%"
{{ Math.round(variables.acertos / variables.total * 100) + "%" }}  →  "85%"

// Número con separador de miles
{{ variables.numero.toLocaleString("pt-BR") }}  →  "1.234.567"

// Primera letra mayúscula
{{ variables.nome.charAt(0).toUpperCase() + variables.nome.slice(1).toLowerCase() }}  →  "João"

// Título (cada palabra con mayúscula)
{{ variables.titulo.split(" ").map(w => w.charAt(0).toUpperCase() + w.slice(1).toLowerCase()).join(" ") }}
```

### Manipulaciones Complejas

```
// Extraer dominio de email
{{ steps.api.outputs.json.email.split("@")[1] }}  →  "exemplo.com"

// Extraer dominio de URL
{{ new URL(variables.url).hostname }}  →  "www.exemplo.com"

// Iniciales del nombre
{{ variables.nome.split(" ").map(n => n[0]).join("") }}  →  "JS" (João Silva)

// Contar palabras
{{ variables.texto.trim().split(/\s+/).length }}  →  cantidad de palabras

// Contar caracteres sin espacios
{{ variables.texto.replace(/\s/g, "").length }}

// Truncar texto
{{ variables.texto.length > 50 ? variables.texto.substring(0, 50) + "..." : variables.texto }}

// Eliminar acentos
{{ variables.texto.normalize("NFD").replace(/[\u0300-\u036f]/g, "") }}

// Slug (URL amigable)
{{ variables.titulo.toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "").replace(/[^a-z0-9]+/g, "-").replace(/^-|-$/g, "") }}

// Generar ID único
{{ Date.now() + "-" + Math.random().toString(36).substr(2, 9) }}  →  "1705320000000-k7j3h2g1f"

// Hash simple (no criptográfico)
{{ variables.texto.split("").reduce((hash, char) => ((hash << 5) - hash) + char.charCodeAt(0), 0) }}

// Invertir string
{{ variables.texto.split("").reverse().join("") }}

// Eliminar caracteres duplicados de string
{{ [...new Set(variables.texto.split(""))].join("") }}

// Capitalizar cada palabra
{{ variables.frase.replace(/\b\w/g, l => l.toUpperCase()) }}
```

### Cálculos y Estadísticas

```
// Promedio
{{ variables.notas.reduce((a, b) => a + b, 0) / variables.notas.length }}

// Mediana
{{ variables.notas.sort((a, b) => a - b)[Math.floor(variables.notas.length / 2)] }}

// Suma
{{ variables.valores.reduce((a, b) => a + b, 0) }}

// Máximo
{{ Math.max(...variables.valores) }}

// Mínimo
{{ Math.min(...variables.valores) }}

// Desviación estándar (simplificada)
{{ Math.sqrt(variables.valores.map(x => Math.pow(x - variables.valores.reduce((a,b) => a+b) / variables.valores.length, 2)).reduce((a,b) => a+b) / variables.valores.length) }}

// Porcentaje de crecimiento
{{ ((variables.valorNovo - variables.valorAntigo) / variables.valorAntigo * 100).toFixed(1) + "%" }}

// Regla de tres
{{ (variables.valor * variables.proporcao / 100) }}

// Interés simple
{{ variables.capital * variables.taxa * variables.tempo / 100 }}

// Días entre fechas
{{ Math.floor((new Date(variables.dataFim) - new Date(variables.dataInicio)) / (1000 * 60 * 60 * 24)) }}

// Edad a partir de fecha de nacimiento
{{ Math.floor((new Date() - new Date(variables.dataNascimento)) / (1000 * 60 * 60 * 24 * 365.25)) }}

// IMC (Índice de Masa Corporal)
{{ (variables.peso / Math.pow(variables.altura, 2)).toFixed(1) }}
```

### Agrupamientos y Transformaciones

```
// Agrupar por propiedad
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.categoria] = acc[item.categoria] || [];
  acc[item.categoria].push(item);
  return acc;
}, {}) }}

// Contar ocurrencias
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.status] = (acc[item.status] || 0) + 1;
  return acc;
}, {}) }}

// Sumar por grupo
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.categoria] = (acc[item.categoria] || 0) + item.valor;
  return acc;
}, {}) }}

// Eliminar duplicados
{{ [...new Set(steps.query.outputs.rows.map(r => r.email))] }}

// Ordenar objetos por múltiples campos
{{ steps.query.outputs.rows.sort((a, b) => a.categoria.localeCompare(b.categoria) || a.nome.localeCompare(b.nome)) }}

// Filtrar y mapear en cadena
{{ steps.query.outputs.rows.filter(r => r.ativo).map(r => r.nome).sort().join(", ") }}

// Crear lookup/índice
{{ steps.query.outputs.rows.reduce((acc, item) => {
  acc[item.id] = item;
  return acc;
}, {}) }}

// Aplanar array de arrays
{{ steps.api.outputs.json.data.flatMap(item => item.tags) }}

// Particionar array (pares e impares)
{{ steps.query.outputs.rows.reduce((acc, item, i) => {
  acc[i % 2 === 0 ? "pares" : "impares"].push(item);
  return acc;
}, {pares: [], impares: []}) }}
```

---

## Uso en los Nodos

### URL de Navegación

```
{{ variables.BASE_URL }}/login
{{ variables.BASE_URL }}/users/{{ steps["http-request"].outputs.json.id }}
{{ variables.BASE_URL }}/products?category={{ steps.api.outputs.json.category }}&page={{ variables.page }}
```

### Texto para Fill/Type

```
{{ variables.TEST_EMAIL }}
{{ steps["custom-js"].outputs.result.email }}
test-{{ Date.now() }}@exemplo.com
{{ variables.FIRST_NAME }} {{ variables.LAST_NAME }}
```

### SQL con Datos Dinámicos

```sql
SELECT * FROM orders WHERE user_id = '{{ steps["http-request"].outputs.json.userId }}'

SELECT * FROM products WHERE category = '{{ variables.CATEGORY }}' AND price > {{ variables.MIN_PRICE }}

INSERT INTO logs (user_id, action, timestamp)
VALUES ('{{ steps.login.outputs.json.userId }}', 'login', '{{ new Date().toISOString() }}')
```

> **Cuidado:** Usa parámetros ($1, ?) en lugar de interpolación directa para prevenir SQL injection cuando sea posible.

### Body de HTTP Request

```json
{
  "token": "{{ steps.login.outputs.json.token }}",
  "data": {
    "name": "{{ variables.TEST_NAME }}",
    "email": "{{ steps["custom-js"].outputs.result.email }}",
    "timestamp": "{{ new Date().toISOString() }}"
  }
}
```

> En el ejemplo anterior, `steps.login` es un nodo HTTP Request renombrado a "login".

### Headers de HTTP Request

```
Authorization: Bearer {{ steps.login.outputs.json.token }}
X-User-ID: {{ steps.api.outputs.json.userId }}
X-Request-ID: {{ Date.now() }}-{{ Math.random().toString(36).substr(2, 9) }}
```

### Condición del If

```
{{ steps["http-request"].outputs.status === 200 }}
{{ steps["postgres-query"].outputs.rowCount > 0 }}
{{ steps["web-flow"].outputs.extracts.price < variables.MAX_PRICE }}
{{ steps.api.outputs.json.role === "admin" || steps.api.outputs.json.role === "moderator" }}
{{ steps.api.outputs.json.age >= 18 && steps.api.outputs.json.active }}
```

### Expresión del Switch

```
{{ steps.api.outputs.status }}
{{ steps.api.outputs.json.userType }}
{{ Math.floor(steps.api.outputs.json.score / 10) }}
```

### Items del Loop (For Each)

```
{{ steps["postgres-query"].outputs.rows }}
{{ steps.api.outputs.json.users }}
{{ Array.from({length: 10}, (_, i) => i + 1) }}
{{ steps.api.outputs.json.data.filter(item => item.active) }}
```

### Condición del Loop (While)

```
{{ variables.contador < 10 }}
{{ steps.api.outputs.json.hasMore }}
{{ steps["postgres-query"].outputs.rowCount > 0 }}
{{ variables.tentativas < variables.MAX_TENTATIVAS && !variables.sucesso }}
```

### Mensaje de Log

```
Usuario {{ steps["postgres-query"].outputs.rows[0].name }} tiene {{ steps["postgres-query"].outputs.rows[0].orders }} pedidos

La API respondió con estado {{ steps["http-request"].outputs.status }}: {{ steps["http-request"].outputs.json.message }}

Procesados {{ variables._loopIndex + 1 }} de {{ steps.loop.outputs._loopCount }} items

Total: $ {{ steps.query.outputs.rows.reduce((sum, r) => sum + r.price, 0).toFixed(2) }}
```

### Set Variable (Definir Variable)

```
// Nombre de la variable: userId
// Valor:
{{ steps.login.outputs.json.user.id }}

// Nombre de la variable: fullName
// Valor:
{{ steps.api.outputs.json.firstName + " " + steps.api.outputs.json.lastName }}

// Nombre de la variable: processedData
// Valor:
{{ steps.query.outputs.rows.map(r => ({id: r.id, name: r.name.toUpperCase()})) }}

// Nombre de la variable: timestamp
// Valor:
{{ new Date().toISOString() }}
```

---

## Loops y Variables Especiales

Cuando un nodo está dentro de un loop, tienes acceso a variables especiales:

```
{{ variables._loopItem }}      // Item actual de la iteración
{{ variables._loopIndex }}     // Índice actual (comienza en 0)
```

También puedes acceder mediante los outputs del nodo loop:

```
{{ steps.loop.outputs._loopItem }}
{{ steps.loop.outputs._loopIndex }}
{{ steps.loop.outputs._loopCount }}  // Total de iteraciones (disponible después del loop)
```

### Ejemplos en Loops

```
// Mensaje con progreso
{{ "Procesando item " + (variables._loopIndex + 1) + " de " + steps.loop.outputs._loopCount }}

// Acceder a propiedad del item actual
{{ variables._loopItem.nome }}
{{ variables._loopItem.preco * 1.1 }}

// Usar índice en cálculos
{{ "Item #" + (variables._loopIndex + 1) + ": " + variables._loopItem.name }}

// Condición basada en el índice
{{ variables._loopIndex === 0 ? "Primero" : variables._loopIndex === steps.loop.outputs._loopCount - 1 ? "Último" : "Medio" }}

// Usar item en URL
{{ variables.BASE_URL }}/api/users/{{ variables._loopItem.id }}

// Usar item en SQL
SELECT * FROM orders WHERE user_id = '{{ variables._loopItem.userId }}'
```

---

## Manejo de Errores

Si una expresión falla (la propiedad no existe, tipo incorrecto, etc.), el resultado se reemplaza por un marcador de error:

```
[Error: Cannot read property 'name' of undefined]
```

Para prevenir errores, usa el operador de optional chaining o fallbacks:

```
// Optional chaining (nodo renombrado a "api")
{{ steps.api.outputs.json?.user?.name }}

// O con rótulo por defecto:
{{ steps["http-request"].outputs.json?.user?.name }}

// Fallback con OR
{{ steps["http-request"].outputs.json.name || "N/A" }}

// Fallback con nullish coalescing
{{ steps.api.outputs.json.count ?? 0 }}

// Verificación antes de acceder
{{ steps.api.outputs.json.user ? steps.api.outputs.json.user.name : "Sin usuario" }}

// Verificación de array vacío
{{ steps.query.outputs.rows.length > 0 ? steps.query.outputs.rows[0].name : "Sin resultados" }}
```

> **Importante:** Si una expresión hace referencia a un nodo o variable que no existe (ej: error de escritura), el nodo fallará con un mensaje de error claro indicando qué expresión falló, en lugar de causar un comportamiento inesperado.

---

## Seguridad

El sistema de expresiones se ejecuta en un **entorno sandbox** con restricciones:

- **Sin acceso** a `require`, `import`, `process`
- **Sin acceso** al sistema de archivos
- **Sin acceso** a la red (usa el nodo HTTP Request para eso)
- Solo `steps` y `variables` están disponibles en el contexto

---

## Consejos y Buenas Prácticas

### 1. Renombra los Nodos con Rótulos Simples

```
// ❌ Difícil de leer
{{ steps["http-request"].outputs.json.token }}

// ✅ Más limpio (renombra el nodo a "login")
{{ steps.login.outputs.json.token }}
```

### 2. Usa Optional Chaining para Evitar Errores

```
// ❌ Puede dar error si user no existe
{{ steps.api.outputs.json.user.name }}

// ✅ Retorna undefined si no existe
{{ steps.api.outputs.json?.user?.name }}

// ✅ Con valor por defecto
{{ steps.api.outputs.json?.user?.name ?? "Sin nombre" }}
```

### 3. Usa Interpolación para Construir URLs y Mensajes

```
// ✅ URL dinámica
{{ variables.BASE_URL }}/api/users/{{ steps.api.outputs.json.id }}

// ✅ Mensaje formateado
Estado: {{ steps.api.outputs.status }} - {{ steps.api.outputs.json.message }}
```

### 4. Usa Expresión Única para Mantener los Tipos

```
// ✅ Retorna array (expresión única)
{{ steps.query.outputs.rows }}

// ❌ Retorna string (interpolación)
Rows: {{ steps.query.outputs.rows }}
```

### 5. Agrega un Nodo Log para Depuración

```
// Log para inspeccionar valores
{{ JSON.stringify(steps.api.outputs.json, null, 2) }}

// Log de variables
Variables: {{ JSON.stringify(variables) }}

// Log de item del loop
Item actual: {{ JSON.stringify(variables._loopItem) }}
```

### 6. Usa Fallbacks para Datos Opcionales

```
{{ steps.api.outputs.json.name || "Anónimo" }}
{{ steps.api.outputs.json.config || {} }}
{{ steps.api.outputs.json.items || [] }}
```

### 7. Para Lógica Compleja, Usa Custom JavaScript

Si la expresión se está volviendo muy compleja, considera:
- Usar un nodo **Custom JavaScript** para procesar los datos
- Dividir la lógica en múltiples nodos **Set Variable**
- Usar nodos de control (If, Switch, Loop)

```
// ❌ Expresión demasiado compleja
{{ steps.query.outputs.rows.filter(r => r.active && r.age > 18).map(r => ({...r, fullName: r.firstName + " " + r.lastName})).sort((a, b) => a.fullName.localeCompare(b.fullName)) }}

// ✅ Usa Custom JavaScript
// En el nodo Custom JavaScript:
const activeAdults = steps.query.outputs.rows
  .filter(r => r.active && r.age > 18)
  .map(r => ({...r, fullName: `${r.firstName} ${r.lastName}`}))
  .sort((a, b) => a.fullName.localeCompare(b.fullName));

return { processedUsers: activeAdults };

// Luego referencia:
{{ steps.transform.outputs.result.processedUsers }}
```

### 8. Convierte Tipos Cuando Sea Necesario

```
// ❌ Concatenación no deseada
{{ "10" + "5" }}  // "105"

// ✅ Suma numérica
{{ Number("10") + Number("5") }}  // 15
{{ parseInt("10") + parseInt("5") }}  // 15
```

### 9. Valida Antes de Usar

```
{{ Array.isArray(variables.lista) && variables.lista.length > 0 ? variables.lista[0] : "Lista vacía" }}

{{ typeof steps.api.outputs.json.data === "object" ? JSON.stringify(steps.api.outputs.json.data) : "Datos inválidos" }}
```

### 10. Usa Template Strings para Mayor Legibilidad

```
// ❌ Difícil de leer
{{ "Nombre: " + variables.nome + ", Edad: " + variables.idade + ", Email: " + variables.email }}

// ✅ Más legible
{{ `Nombre: ${variables.nome}, Edad: ${variables.idade}, Email: ${variables.email}` }}
```

---

## Ejemplos Completos

### Ejemplo 1: Flujo de Autenticación con API

```
Paso 1 (http-request): Login
  URL: https://api.example.com/login
  Body: { "user": "admin", "pass": "secret" }

Paso 2 (set-variable): Guardar Token
  Key: authToken
  Value: {{ steps.login.outputs.json.token }}

Paso 3 (http-request): Obtener Datos de Usuario
  URL: https://api.example.com/user
  Headers: { "Authorization": "Bearer {{ variables.authToken }}" }
```

### Ejemplo 2: Consulta de Base de Datos con Lógica Condicional

```
Paso 1 (postgres-query): Obtener Usuarios
  SQL: SELECT * FROM users WHERE active = true

Paso 2 (if): Verificar si existen usuarios
  Condición: {{ steps["postgres-query"].outputs.rowCount > 0 }}

Paso 3 (loop): Procesar cada usuario (rama true)
  Array: {{ steps["postgres-query"].outputs.rows }}

Paso 4 (http-request): Enviar notificación (dentro del loop)
  URL: https://api.example.com/notify
  Body: { "email": "{{ variables._loopItem.email }}" }
```

### Ejemplo 3: Web Scraping con Extracción de Datos

```
Paso 1 (web-flow): Extraer datos del producto
  Navigate: https://example.com/products
  Extract: productName (selector: .product-title)
  Extract: productPrice (selector: .price)

Paso 2 (set-variable): Guardar info del producto
  Key: product
  Value: {{ { name: steps["web-flow"].outputs.extracts.productName, price: steps["web-flow"].outputs.extracts.productPrice } }}

Paso 3 (postgres-query): Guardar en base de datos
  SQL: INSERT INTO products (name, price) VALUES ($1, $2)
  Params: {{ [variables.product.name, variables.product.price] }}
```

---

## Limitaciones

- Las expresiones se evalúan en un entorno sandbox seguro
- No es posible acceder a las APIs del navegador (window, document, etc.)
- No es posible hacer solicitudes HTTP directamente (usa nodos específicos)
- No es posible importar módulos externos
- Las funciones asíncronas (async/await) no son compatibles en expresiones
- No es posible modificar el contexto global ni las variables de otros nodos

---

## Conclusión

El sistema de expresiones de QANode ofrece todo el poder de JavaScript para crear flujos dinámicos e inteligentes. Con la sintaxis `{{ }}`, puedes:

- Acceder a datos de nodos anteriores y variables
- Realizar cálculos y transformaciones complejas
- Validar y formatear datos
- Crear lógica condicional
- Manipular strings, arrays y objetos
- Trabajar con fechas y números

Recuerda siempre usar `{{ }}` para delimitar tus expresiones y aprovecha todo el poder de JavaScript para crear pruebas robustas y eficientes.
