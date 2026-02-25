# Nodo Set Variable

El nodo **Set Variable** permite definir o actualizar variables en tiempo de ejecución. Las variables definidas quedan disponibles para todos los nodos siguientes en el flujo.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `set-variable` |
| **Categoría** | Utilidades |
| **Color** | ⚪ Gris (#6b7280) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Configuración

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Clave** | `string` | Nombre de la variable |
| **Valor** | `any` | Valor a asignar (soporta `{{ }}`) |

---

## Uso

### Definir una variable simple

```
Clave: token
Valor: {{ steps.login.outputs.json.token }}
```

### Definir una variable con cálculo

```
Clave: totalPrice
Valor: {{ steps.cart.outputs.json.subtotal * 1.1 }}
```

### Acceder a la variable en nodos siguientes

Después de definirla, la variable es accesible mediante:
```
{{ variables.token }}
{{ variables.totalPrice }}
```

---

## Ejemplos

### Guardar un token de autenticación

```
[HTTP Request: POST /login] → [Set Variable: token = {{ steps.login.outputs.json.token }}]
    │
    ▼
[HTTP Request: GET /profile → Header: Authorization = Bearer {{ variables.token }}]
```

### Preparar una URL dinámica

```
[Set Variable: baseUrl = https://{{ variables.ENVIRONMENT }}.exemplo.com]
    │
    ▼
[HTTP Request: GET {{ variables.baseUrl }}/api/users]
```

---

## Consejos

- Use **nombres descriptivos** para las variables: `authToken` es mejor que `t`
- Las variables definidas con Set Variable **sobreescriben** las variables globales con el mismo nombre durante la ejecución
- El valor soporta **cualquier expresión** `{{ }}`, incluyendo cálculos y acceso a outputs
