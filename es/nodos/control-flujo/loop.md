# Nodo Loop

El nodo **Loop** permite repetir la ejecución de una sección del flujo múltiples veces. Soporta tres modos de repetición: por conteo, por array y por condición (while).

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `loop` |
| **Categoría** | Control de Flujo |
| **Color** | Amarillo (#f59e0b) |
| **Entrada** | `in` |
| **Salidas** | `loop` (cuerpo del loop), `done` (salida tras la finalización) |

---

## Modos de Loop

### 1. Conteo (Count)

Repite un número fijo de veces.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Conteo** | `string/number` | Número de iteraciones |
| **Índice Inicial** | `number` | Valor inicial del índice (predeterminado: 0) |

**Ejemplo:** Repetir 5 veces
```
Conteo: 5
Índice Inicial: 0
```

En cada iteración, el índice se incrementa (0, 1, 2, 3, 4).

### 2. Array

Itera sobre cada elemento de un array.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Expresión del Array** | `string` | Expresión que retorna un array |

**Ejemplo:** Iterar sobre resultados de una consulta
```
{{ steps.query.outputs.rows }}
```

En cada iteración, el elemento actual y el índice están disponibles en las salidas del loop.

### 3. Mientras (While)

Repite mientras una condición sea verdadera.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Condición** | `string` | Expresión evaluada en cada iteración |
| **Máximo de Iteraciones** | `number` | Límite de seguridad (predeterminado: 100) |

Al igual que el nodo If, el modo while soporta **modo simple** (expresión JavaScript) y **modo visual builder**.

**Ejemplo:** Repetir mientras haya una página siguiente
```javascript
{{ steps["http-request"].outputs.json.hasNextPage }} === true
```

---

## Salidas

| Salida | Tipo | Descripción |
|--------|------|-------------|
| `_loopItem` | `any` | Elemento iterado en modo array |
| `_loopIndex` | `any` | Índice de la última iteración |
| `_loopCount` | `number` | Número total de iteraciones |

---

## Comportamiento

1. El loop evalúa si debe continuar (basado en el modo)
2. Si es así, ejecuta todos los nodos conectados a la salida **loop**
3. Al final del cuerpo del loop, regresa al nodo Loop para la siguiente iteración
4. Cuando el loop termina, ejecuta los nodos conectados a la salida **done**

### Flujo Visual

```
        ┌─────────────────────────────────┐
        │                                 │
        ▼                                 │
[Loop] ──loop──→ [Acción 1] → [Acción 2] ─┘
   │
   done
   │
   ▼
[Siguiente Nodo]
```

> **Nota:** Los nodos en el cuerpo del loop se ejecutan en cada iteración. Usa expresiones para acceder al elemento/índice actual.

---

## Ejemplo Práctico

### Iterar sobre resultados de base de datos

```
[PostgreSQL: SELECT * FROM users WHERE active = true]
    │
    ▼
[Loop (array): {{ steps.query.outputs.rows }}]
    │ loop → [HTTP Request: POST /api/notify]
    │             Body: { "email": "{{ steps.loop.outputs._loopItems[steps.loop.outputs._loopCount].email }}" }
    │
    │ done → [Log: "Todas as notificações enviadas"]
```

### Loop con conteo

```
[Loop (count): 3 veces]
    │ loop → [HTTP Request: GET /api/retry-endpoint]
    │             → [If: status === 200]
    │                   true → (break/success)
    │
    │ done → [Log: "Tentativas esgotadas"]
```

---

## Consejos

- Configura siempre el **máximo de iteraciones** en el modo while para evitar loops infinitos
- Para arrays grandes, considera el impacto en el rendimiento
- Usa nodos de **Log** dentro del loop para hacer seguimiento del progreso
- En el modo array, los elementos del array son accesibles mediante `_loopItems`
