# Nodo Stop and Fail

El nodo **Stop and Fail** interrumpe inmediatamente la ejecución del flujo y marca el resultado como **fallido**. Es un nodo terminal — no tiene salidas.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `stop-and-fail` |
| **Categoría** | Utilidades |
| **Color** | ⚪ Gris (#6b7280) |
| **Entrada** | `in` |
| **Salidas** | Ninguna (nodo terminal) |

---

## Configuración

Este nodo no tiene campos de configuración. Al ser alcanzado, simplemente detiene la ejecución.

---

## Uso

Stop and Fail se utiliza típicamente al final de rutas condicionales que representan escenarios de error:

```
[If: {{ steps.api.outputs.status }} === 200]
    │ true → [Continuar testes...]
    │ false → [Log: "API retornou erro"] → [Stop and Fail]
```

```
[Switch: {{ steps.query.outputs.rowCount }}]
    │ case-0 (0) → [Log: "Nenhum dado encontrado"] → [Stop and Fail]
    │ default → [Processar dados...]
```

---

## Consejos

- Combine con un nodo **Log** antes para registrar el motivo del fallo
- Use después de validaciones críticas que deben impedir la continuación del flujo
- El estado de toda la ejecución será marcado como **failed**
