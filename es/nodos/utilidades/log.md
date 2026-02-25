# Nodo Log

El nodo **Log** registra un mensaje en el log de ejecución. Útil para depuración, seguimiento del progreso y documentación de valores durante la ejecución.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `log` |
| **Categoría** | Utilidades |
| **Color** | ⚪ Gris (#6b7280) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Configuración

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Mensaje** | `string` | Texto a registrar (soporta `{{ }}`) |

---

## Ejemplos

### Log simple

```
Mensaje: Test started successfully
```

### Log con datos dinámicos

```
Mensaje: Usuario encontrado: {{ steps.query.outputs.rows[0].name }} ({{ steps.query.outputs.rows[0].email }})
```

### Log para depuración

```
Mensaje: Estado de la API: {{ steps["http-request"].outputs.status }}, Body: {{ JSON.stringify(steps["http-request"].outputs.json) }}
```

---

## Consejos

- Use logs estratégicamente para **documentar valores** en puntos críticos del flujo
- Los mensajes aparecen en los **resultados de la ejecución** y en el **reporte PDF**
- Combine con **If/Switch** para logs condicionales:
  ```
  [If: status === 200]
      │ true → [Log: "Sucesso"]
      │ false → [Log: "Falha: status {{ steps.api.outputs.status }}"]
  ```
