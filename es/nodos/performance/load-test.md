# Nodo Load Test

El nodo **Load Test** ejecuta pruebas de carga y rendimiento contra endpoints HTTP, simulando múltiples usuarios virtuales (VUs) en paralelo. Genera evidencias visuales en PNG con métricas detalladas y soporta criterios de aprobación automáticos mediante umbrales.

---

## Visión General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `load-test` |
| **Categoría** | Performance |
| **Color** | 🟫 Ámbar (#92400E) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Tipos de Prueba

Cada tipo genera un perfil de carga diferente automáticamente a partir de los campos **VUs** y **Duración**.

| Tipo | Descripción | Cuándo Usar |
|------|-------------|-------------|
| **Smoke** | 1 VU, duración configurada | Validar que el endpoint responde antes de ejecutar pruebas mayores |
| **Load** | Ramp up → sostenimiento → ramp down | Verificar el comportamiento bajo carga normal esperada |
| **Stress** | Ramp agresivo hasta el límite | Identificar el punto donde el sistema comienza a degradarse |
| **Spike** | Pico súbito de VUs | Probar la reacción ante picos repentinos de tráfico (ej: Black Friday) |
| **Soak** | Carga sostenida por larga duración | Detectar memory leaks y degradación gradual |
| **Breakpoint** | Escalera progresiva de VUs | Encontrar el punto exacto de quiebre del sistema |

---

## Configuración

### Solicitud

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Credencial** | `string` | Credencial HTTP guardada (rellena URL y auth automáticamente) |
| **Método** | `string` | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| **URL** | `string` | Endpoint objetivo (soporta `{{ }}`) |
| **Auth** | `string` | Tipo de autenticación manual (si no usa credencial) |
| **Headers** | `object` | Cabeceras adicionales de la solicitud |
| **Body** | `any` | Cuerpo de la solicitud (para POST, PUT, PATCH) |

### Configuración de Carga

| Campo | Tipo | Valor por defecto | Descripción |
|-------|------|-------------------|-------------|
| **VUs** | `number` | `10` | Número de usuarios virtuales simultáneos |
| **Duración (s)** | `number` | `30` | Duración total de la prueba en segundos |
| **Think Time (ms)** | `number` | `0` | Pausa entre solicitudes de cada VU |
| **Timeout (ms)** | `number` | `30000` | Tiempo máximo de espera por respuesta |

> Para **Smoke**, el campo VUs está fijo en 1 y no se muestra.  
> Para **Soak**, la duración por defecto es 1800s (30 min).  
> Para **Breakpoint**, VUs representa el **máximo** de VUs que se alcanzarán.

#### Cómo funciona el Breakpoint

La prueba divide la duración total en pasos de ~30s, aumentando los VUs progresivamente:

```
VUs: 10 | Duración: 60s  →  2 pasos de 30s
  Paso 1: 0s → 30s  →  5 VUs
  Paso 2: 30s → 60s →  10 VUs
```

### Stages Personalizadas

Active **Custom** en la sección Stages para definir manualmente el perfil de carga:

| Campo | Descripción |
|-------|-------------|
| **Duración (s)** | Duración de esta stage en segundos |
| **Target VUs** | Número de VUs al final de esta stage |

Ejemplo de stages para una prueba de stress manual:

| Duración | Target VUs | Descripción |
|----------|-----------|-------------|
| 30s | 10 | Ramp up inicial |
| 60s | 50 | Carga sostenida |
| 30s | 100 | Stress |
| 15s | 0 | Ramp down |

---

## Autenticación

### Usando Credenciales Guardadas

Seleccione una credencial de tipo **HTTP/API**. La URL base y los datos de autenticación se aplican automáticamente:

1. Seleccione la credencial en el campo **Credencial**
2. La URL base se completa automáticamente en el campo **URL**
3. Complete con el path del endpoint: `/api/checkout`

### Autenticación Manual

| Tipo | Campos | Resultado |
|------|--------|-----------|
| **Bearer Token** | Token | Header `Authorization: Bearer {token}` |
| **Basic Auth** | Usuario + Contraseña | Header `Authorization: Basic {base64}` |
| **API Key** | Header Name + Token | Header personalizado con el token |

---

## Umbrales (Thresholds)

Los umbrales definen criterios de aprobación automáticos. Si algún umbral no se cumple, el nodo se marca como **FALLIDO**.

| Métrica | Descripción |
|---------|-------------|
| `p50` | Percentil 50 de latencia (ms) |
| `p95` | Percentil 95 de latencia (ms) |
| `p99` | Percentil 99 de latencia (ms) |
| `avgDuration` | Latencia promedio (ms) |
| `errorRate` | Tasa de errores (%) |
| `rps` | Solicitudes por segundo |

| Operador | Significado |
|----------|-------------|
| `<` | Menor que |
| `≤` | Menor o igual |
| `>` | Mayor que |
| `≥` | Mayor o igual |

**Ejemplos de umbrales comunes:**

| Umbral | Significado |
|--------|-------------|
| `p95 < 500` | 95% de las respuestas en menos de 500ms |
| `errorRate < 1` | Tasa de error menor al 1% |
| `rps > 10` | Mínimo 10 solicitudes por segundo |
| `p99 < 2000` | 99% de las respuestas en menos de 2s |

> Sin umbrales configurados, el nodo siempre pasa (mientras el endpoint responda).

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `passed` | `boolean` | `true` si todos los umbrales se cumplieron |
| `testType` | `string` | Tipo de prueba ejecutada |
| `metrics` | `object` | Métricas consolidadas de la prueba |
| `thresholds` | `array` | Resultado de cada umbral configurado |
| `stages` | `array` | Stages ejecutadas (auto o personalizadas) |

### Estructura del objeto `metrics`

```json
{
  "totalRequests": 1141,
  "errorCount": 0,
  "errorRate": 0.0,
  "rps": 34.49,
  "avgDuration": 212,
  "minDuration": 98,
  "maxDuration": 668,
  "p50": 182,
  "p90": 332,
  "p95": 451,
  "p99": 579
}
```

### Accediendo a los Outputs

```
// Verificar si pasó
{{ steps["load-test"].outputs.passed }}  →  true

// Total de solicitudes
{{ steps["load-test"].outputs.metrics.totalRequests }}  →  1141

// Latencia p95
{{ steps["load-test"].outputs.metrics.p95 }}  →  451

// Tasa de error
{{ steps["load-test"].outputs.metrics.errorRate }}  →  0.0

// RPS
{{ steps["load-test"].outputs.metrics.rps }}  →  34.49
```

---

## Evidencias Generadas

El nodo genera automáticamente **dos gráficos PNG** como evidencia de la ejecución:

### 1. Reporte de Resumen (`load-test-report.png`)

Vista consolidada con:
- Tarjetas de métricas (p50, p95, p99, Avg, RPS, Requests, Errors, Error Rate)
- Gráfico de barras horizontales con distribución de latencia
- Gráfico de throughput a lo largo del tiempo (req/s)
- Pills de estado para cada umbral configurado
- Pie de página con las stages ejecutadas

### 2. Timeline RPS × Latencia (`load-test-timeline.png`)

Gráfico de doble eje con:
- **Barras azules** (eje izquierdo): RPS a lo largo del tiempo
- **Línea naranja** (eje derecho): Latencia promedio a lo largo del tiempo
- **Línea morada discontinua** (eje derecho): Latencia p95 a lo largo del tiempo

> El gráfico de timeline es especialmente útil para pruebas **Breakpoint** y **Stress**, donde se puede visualizar exactamente en qué momento la latencia comienza a subir en respuesta al aumento de carga.

---

## Ejemplos Prácticos

### Smoke test — Validación básica

```
Tipo: Smoke
URL: https://api.ejemplo.com/health
Método: GET
Duración: 10s
```

Ideal para ejecutar al inicio de un flujo de pruebas — garantiza que el entorno está disponible.

### Load test — Carga normal con umbrales

```
Tipo: Load
URL: https://api.ejemplo.com/products
Método: GET
VUs: 20
Duración: 60s
Umbrales:
  - p95 < 500
  - errorRate < 1
```

### Stress test — Límites del sistema

```
Tipo: Stress
URL: https://api.ejemplo.com/checkout
Método: POST
VUs: 100
Duración: 120s
Body: { "productId": "123", "quantity": 1 }
Auth: Bearer → {{ variables.API_TOKEN }}
Umbrales:
  - p99 < 2000
  - errorRate < 5
```

### Breakpoint — Punto de quiebre

```
Tipo: Breakpoint
URL: https://api.ejemplo.com/search
Método: GET
Max VUs: 200
Duración: 300s
Umbrales:
  - p95 < 1000
  - errorRate < 2
```

El sistema aumentará progresivamente de 1 a 200 VUs en pasos de ~30s. Cuando p95 supere 1000ms o los errores superen el 2%, el nodo marca como fallido — indicando el punto de quiebre.

### Encadenando con otros nodos

```
[Load Test: Smoke]
    │ outputs.passed = true
    ▼
[If: {{ steps.smoke.outputs.passed }}]
    │ true  → [Load Test: Carga completa]
    │ false → [Log: "Smoke falló — entorno no disponible"]
```

---

## Aislamiento de Cola

El nodo Load Test se ejecuta en una cola separada (`qanode-load-tests`) para no interferir con otros flujos en ejecución.

Para configurar un worker dedicado al Load Test:

```bash
WORKER_QUEUES=load-tests node dist/start.js
```

Para un worker que procese ambas colas:

```bash
WORKER_QUEUES=executions,load-tests node dist/start.js
```

---

## Consejos

- **Comience por Smoke** antes de ejecutar pruebas de carga — garantiza que el endpoint responde correctamente
- **Configure umbrales** para que la prueba falle automáticamente cuando el sistema se degrade, sin necesidad de analizar los números manualmente
- **Use el gráfico de timeline** para identificar el momento exacto de degradación en pruebas Breakpoint y Stress
- **Think Time** simula comportamiento humano — útil para pruebas Soak donde se desea carga continua pero realista
- **Las credenciales guardadas** facilitan la ejecución en diferentes entornos (staging, producción) sin modificar el flujo
- Para pruebas confiables, **hasta 100 VUs** por instancia de worker. Por encima de eso, considere un worker dedicado
