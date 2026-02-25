# Nodo Wait

El nodo **Wait** pausa la ejecución del flujo durante un tiempo determinado o hasta que se cumpla una condición.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `wait` |
| **Categoría** | Utilidades |
| **Color** | ⚪ Gris (#6b7280) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Modos

### Duración (Duration)

Espera un tiempo fijo en milisegundos.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Duración (ms)** | `number` | Tiempo de espera en milisegundos |

**Ejemplo:** Esperar 5 segundos
```
Modo: duration
Duración: 5000
```

### Hasta Condición (Until)

Espera hasta que una condición se vuelva verdadera, verificando periódicamente.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Condición** | `string` | Expresión a evaluar |
| **Intervalo (ms)** | `number` | Intervalo entre verificaciones (predeterminado: 1000) |
| **Timeout (ms)** | `number` | Tiempo máximo de espera (predeterminado: 30000) |

**Ejemplo:** Esperar hasta que una variable esté definida
```
Modo: until
Condición: {{ variables.processComplete === true }}
Intervalo: 2000
Timeout: 60000
```

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `waited` | `number` | Tiempo real de espera (ms) |
| `conditionMet` | `boolean` | Si la condición fue cumplida (modo until) |

---

## Ejemplos Prácticos

### Esperar entre solicitudes

```
[HTTP Request: POST /start-job]
    │
    ▼
[Wait: 10000ms]
    │
    ▼
[HTTP Request: GET /job-status]
```

### Polling hasta la finalización

```
[HTTP Request: POST /process]
    │
    ▼
[Wait Until: {{ steps.checkStatus.outputs.json.status }} === "done"]
    │ Intervalo: 5000ms, Timeout: 120000ms
    ▼
[If: conditionMet]
    │ true → [Log: "Processamento concluído"]
    │ false → [Log: "Timeout!"] → [Stop and Fail]
```

---

## Consejos

- Use **duración** para esperas simples (ej: esperar propagación)
- Use **hasta condición** para polling (verificar el estado periódicamente)
- Siempre defina un **timeout** en el modo "until" para evitar esperas infinitas
