# Nodo Switch

El nodo **Switch** permite crear bifurcaciones múltiples basadas en el valor de una expresión. Es útil cuando hay más de dos caminos posibles (donde el If no sería suficiente).

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `switch` |
| **Categoría** | Control de Flujo |
| **Color** | Amarillo (#f59e0b) |
| **Entrada** | `in` |
| **Salidas** | `case-0`, `case-1`, ..., `default` (dinámicas) |

---

## Configuración

### Modo Simple

En el modo simple, se define una expresión y valores para cada case:

**Campo: Expresión**
```javascript
{{ steps["http-request"].outputs.status }}
```

**Cases:**

| Case | Valor | Etiqueta (Opcional) |
|------|-------|---------------------|
| Case 0 | `200` | Éxito |
| Case 1 | `404` | No Encontrado |
| Case 2 | `500` | Error del Servidor |
| Default | — | Otros |

La expresión es evaluada y comparada con cada valor de case. Si ningún case coincide, el camino `default` es activado.

### Modo Visual Builder

El modo visual permite usar operadores diferentes para cada case:

| Campo | Descripción |
|-------|-------------|
| **Campo de Comparación** | Expresión a evaluar |
| **Operador** | Operación de comparación por case |
| **Valor** | Valor esperado |

**Operadores disponibles:**

| Operador | Descripción |
|----------|-------------|
| `===` | Igual (estricto) |
| `!==` | Diferente |
| `>` | Mayor que |
| `<` | Menor que |
| `>=` | Mayor o igual |
| `<=` | Menor o igual |
| `includes` | Contiene |
| `startsWith` | Empieza con |
| `endsWith` | Termina con |

---

## Salidas

| Salida | Tipo | Descripción |
|--------|------|-------------|
| `_branch` | `string` | El nombre del case activado |

---

## Comportamiento

1. La expresión es evaluada
2. El resultado es comparado con cada case en orden
3. El **primer case** que coincida tendrá su camino activado
4. Si ningún case coincide, el camino `default` es activado
5. Los nodos en los caminos no activados serán **omitidos**

---

## Ejemplo Práctico

### Enrutamiento por estado HTTP

```
[HTTP Request]
    │
    ▼
[Switch: {{ steps["http-request"].outputs.status }}]
    │ case-0 (200) → [Log: "Sucesso"] → [Extrair dados]
    │ case-1 (401) → [Log: "Não autorizado"] → [Refazer login]
    │ case-2 (404) → [Log: "Não encontrado"]
    │ default → [Log: "Erro inesperado"] → [Stop and Fail]
```

### Enrutamiento por tipo de usuario

```
[PostgreSQL: SELECT role FROM users WHERE id = '123']
    │
    ▼
[Switch: {{ steps.query.outputs.rows[0].role }}]
    │ case-0 ("admin") → [Flujo Admin]
    │ case-1 ("user") → [Flujo Usuario]
    │ default → [Flujo Visitante]
```

---

## Consejos

- Usa **etiquetas descriptivas** en los cases para facilitar la lectura del flujo
- El case `default` siempre debe estar conectado para manejar valores inesperados
- Para solo dos caminos, prefiere el nodo [If](if.md)
- Los cases son evaluados en orden — coloca los más comunes primero
