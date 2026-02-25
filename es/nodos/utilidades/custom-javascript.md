# Nodo Custom JavaScript

El nodo **Custom JavaScript** permite ejecutar código JavaScript personalizado durante la ejecución del flujo. Ideal para transformaciones de datos, cálculos complejos y lógica no cubierta por los nodos nativos.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `custom-js` |
| **Categoría** | Utilidades |
| **Color** | ⚪ Gris (#6b7280) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Configuración

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Código** | `string` | Código JavaScript a ejecutar |

El editor de código usa el **Monaco Editor** (el mismo de VS Code) con:
- Syntax highlighting
- Autocomplete para `steps`, `variables`, `return`, `console.log`
- Validación de sintaxis

---

## Contexto de Ejecución

El código tiene acceso a:

| Variable | Tipo | Descripción |
|----------|------|-------------|
| `steps` | `object` | Outputs de todos los nodos anteriores |
| `variables` | `object` | Variables globales y de runtime |

### Acceder a datos de nodos anteriores

```javascript
// Output de un HTTP Request
const userData = steps["http-request"].outputs.json;
const userName = userData.name;

// Output de una consulta PostgreSQL
const rows = steps["postgres-query"].outputs.rows;
const firstUser = rows[0];

// Output de extracción web
const title = steps["web-flow"].outputs.extracts.pageTitle;
```

### Acceder a variables

```javascript
const apiUrl = variables.API_URL;
const environment = variables.ENVIRONMENT;
```

---

## Retornar Valores

El código **debe retornar** un valor usando `return`. El valor retornado queda disponible en los outputs:

```javascript
// Retornar un valor simple
return "Hello World";

// Retornar un objeto
return {
  fullName: steps["Query"].outputs.rows[0].first_name + " " + steps["Query"].outputs.rows[0].last_name,
  isAdmin: steps["Query"].outputs.rows[0].role === "admin"
};

// Retornar un array
return steps["Query"].outputs.rows.map(row => row.email);
```

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `result` | `any` | Valor retornado por el código |

### Acceder al resultado

```
{{ steps["custom-js"].outputs.result }}
{{ steps["custom-js"].outputs.result.fullName }}
{{ steps["custom-js"].outputs.result[0] }}
```

---

## Ejemplos Prácticos

### Transformar datos de una API

```javascript
const items = steps["http-request"].outputs.json.items;

// Filtrar y transformar
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

### Generar datos de prueba

```javascript
const timestamp = Date.now();
return {
  email: `test-${timestamp}@exemplo.com`,
  username: `user_${timestamp}`,
  password: `Pass${timestamp}!`
};
```

### Validación compleja

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

### Formatear un reporte

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

## Limitaciones

- El código se ejecuta en un **entorno sandbox** — sin acceso a:
  - Sistema de archivos (fs)
  - Módulos Node.js (require/import)
  - Red (fetch, http)
  - Variables de entorno (use `variables` en lugar de `process.env`)
- Para operaciones de red, use el nodo **HTTP Request**
- Para operaciones de base de datos, use los nodos de **Base de Datos**
- Para operaciones de archivos, use el nodo **SSH Command**

---

## Consejos

- Use nombres descriptivos para el nodo: "Transformar Datos", "Generar Credenciales", "Validar Respuesta"
- **Siempre retorne** un valor — sin `return`, el output será `undefined`
- Para depuración, use `console.log()` — los mensajes aparecen en los logs del nodo
- El Monaco Editor ofrece autocomplete — escriba `steps.` para ver los nodos disponibles
- Úselo para **transformar datos** entre nodos, no para lógica de negocio compleja
