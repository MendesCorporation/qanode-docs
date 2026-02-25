# Nodo Merge

El nodo **Merge** permite unir múltiples caminos de ejecución en un único punto. Es útil para reconectar caminos que fueron divididos por un nodo If o Switch.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `merge` |
| **Categoría** | Control de Flujo |
| **Color** | Amarillo (#f59e0b) |
| **Entradas** | `in1`, `in2` |
| **Salida** | `out` |

---

## Configuración

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Modo** | `string` | Estrategia de merge |

---

## Comportamiento

El nodo Merge espera que al menos uno de los caminos de entrada sea activado antes de continuar. Cuando un camino condicional (If/Switch) activa solo una rama, el Merge reconoce que la otra rama fue omitida y no queda esperando indefinidamente.

---

## Ejemplo Práctico

### Reconectando después de If

```
                      ┌── true ──→ [Procesar A] ──┐
[If: condición] ──────┤                            ├──→ [Merge] → [Log: "Continuando"]
                      └── false ─→ [Procesar B] ──┘
```

En este ejemplo:
1. El nodo If evalúa la condición
2. Solo uno de los caminos (A o B) será ejecutado
3. El Merge recibe el resultado del camino activado
4. El flujo continúa normalmente después del Merge

### Reconectando después de Switch

```
[Switch] ──case-0──→ [Acción 1] ──┐
         ──case-1──→ [Acción 2] ──┼──→ [Merge] → [Siguiente]
         ──default─→ [Acción 3] ──┘
```

---

## Consejos

- Usa Merge siempre que necesites reconectar caminos divididos
- El Merge es especialmente útil para evitar la duplicación de nodos que deben ejecutarse independientemente del camino tomado
- Nombra el Merge de forma descriptiva: "Merge después de validación", "Unir caminos"
