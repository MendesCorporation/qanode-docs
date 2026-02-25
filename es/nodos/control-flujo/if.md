# Nodo If

El nodo **If** permite crear bifurcaciones condicionales en el flujo. Evalúa una expresión y dirige la ejecución hacia el camino **true** (verdadero) o **false** (falso).

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `if` |
| **Categoría** | Control de Flujo |
| **Color** | Amarillo (#f59e0b) |
| **Entrada** | `in` |
| **Salidas** | `true`, `false` |

---

## Configuración

### Modo Simple

En el modo simple, se escribe una expresión JavaScript que será evaluada como verdadera o falsa:

**Campo: Condición**
```javascript
{{ steps["http-request"].outputs.status === 200 }}
```

La expresión tiene acceso al contexto de ejecución:
- `steps` — Salidas de nodos anteriores
- `variables` — Variables globales

**Ejemplos:**

```javascript
// Verificar estado HTTP
{{ steps["http-request"].outputs.status === 200}}

// Verificar si el texto contiene un valor
{{ steps.extract.outputs.extracts.title.includes("Bem-vindo") }}

// Verificar valor numérico
{{ steps.query.outputs.rowCount > 0 }}

// Verificar booleano
{{ variables.featureEnabled === true }}

// Combinación con AND/OR
{{ steps.api.outputs.status  === 200 && steps.api.outputs.json.active  === true }}
```

### Modo Visual Builder

El modo visual permite construir condiciones sin escribir código:

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| **Campo** | Expresión a evaluar | `steps["http-request"].outputs.status` |
| **Operador** | Operación de comparación | `===` |
| **Valor** | Valor esperado | `200` |
| **Lógica** | Operador entre condiciones | `AND` / `OR` |

**Operadores disponibles:**

| Operador | Descripción |
|----------|-------------|
| `===` | Igual (estricto) |
| `!==` | Diferente (estricto) |
| `==` | Igual (con conversión) |
| `!=` | Diferente (con conversión) |
| `>` | Mayor que |
| `<` | Menor que |
| `>=` | Mayor o igual |
| `<=` | Menor o igual |
| `includes` | Contiene (para strings) |
| `startsWith` | Empieza con |
| `endsWith` | Termina con |

Puedes agregar múltiples condiciones combinadas con **AND** (todas deben ser verdaderas) o **OR** (al menos una debe ser verdadera).

---

## Salidas

| Salida | Tipo | Descripción |
|--------|------|-------------|
| `_branch` | `string` | El camino tomado: `"true"` o `"false"` |

---

## Comportamiento

1. La expresión es evaluada usando los datos del contexto actual
2. Si el resultado es **truthy** (verdadero, número > 0, string no vacío), la salida `true` es activada
3. Si el resultado es **falsy** (false, 0, null, undefined, string vacío), la salida `false` es activada
4. Los nodos conectados al camino **no activado** serán **omitidos**

---

## Ejemplo Práctico

### Verificar respuesta de API

```
[HTTP Request: GET /api/user/1]
    │
    ▼
[If: status === 200]
    │ true → [Log: "Usuário encontrado: {{ steps["http-request"].outputs.json.name }}"]
    │ false → [Log: "Usuário não encontrado"] → [Stop and Fail]
```

### Validación de datos

```
[PostgreSQL: SELECT count(*) FROM orders WHERE user_id = '123']
    │
    ▼
[If: rowCount > 0]
    │ true → [Log: "Pedidos encontrados"]
    │ false → [Log: "Nenhum pedido"]
```

---

## Consejos

- Usa el **modo visual builder** siempre que sea posible — es más legible y menos propenso a errores
- Nombra el nodo de forma descriptiva: "Si login exitoso", "Si el registro existe"
- Recuerda que las expresiones `{{ }}` son evaluadas antes de la comparación
- Para condiciones complejas, considera usar un nodo de **Custom JavaScript** antes y evaluar el resultado en el If
