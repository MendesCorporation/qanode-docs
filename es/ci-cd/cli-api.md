# CLI y API del CI/CD

> **Disponible en:** QANode Enterprise

Esta página muestra cómo usar la CLI oficial de QANode y las rutas dedicadas de CI/CD para integrar el producto con pipelines corporativos.

---

## CLI oficial

El paquete oficial es:

```bash
@qanode/cli
```

Puede usarse sin instalación global:

```bash
npx @qanode/cli help
```

O instalarse como dependencia de desarrollo:

```bash
npm install --save-dev @qanode/cli
```

Luego ejecute:

```bash
npx qanode help
```

> **Recomendación:** En pipelines, `npx @qanode/cli ...` normalmente es suficiente y evita depender de un paquete global preinstalado.

---

## Variables de entorno

Las dos variables principales son:

| Variable | Descripción |
|----------|-------------|
| `QANODE_URL` | URL pública de QANode |
| `QANODE_TOKEN` | Token de integración o token de sesión |

Ejemplo:

```bash
export QANODE_URL=https://qanode.empresa.com
export QANODE_TOKEN=qnt_xxxxx
```

### Sobre la URL

`QANODE_URL` debe ser la misma dirección pública que el equipo ya utiliza en el navegador.

No es necesario conocer el puerto interno de la API cuando su empresa usa:

- reverse proxy
- balanceador
- un dominio público único para frontend + backend

En desarrollo local, `http://localhost:3000` puede funcionar cuando el frontend reenvía `/api`.

---

## Comandos principales

### Validar autenticación

```bash
npx @qanode/cli auth me
```

### Ejecutar escenario por ID

```bash
npx @qanode/cli run scenario --scenario-id SCENARIO_ID --wait
```

### Ejecutar escenario por proyecto y nombre

```bash
npx @qanode/cli run scenario \
  --project-name "Checkout" \
  --scenario-name "Login API" \
  --wait
```

### Ejecutar suite por ID

```bash
npx @qanode/cli run suite --suite-id SUITE_ID --wait
```

### Ejecutar suite por proyecto y nombre

```bash
npx @qanode/cli run suite \
  --project-name "Backoffice" \
  --suite-name "Regresión Login" \
  --wait
```

### Consultar una ejecución

```bash
npx @qanode/cli runs get --run-id RUN_ID
```

### Esperar una ejecución existente

```bash
npx @qanode/cli runs wait --run-id RUN_ID --timeout 300
```

### Descargar el informe consolidado

```bash
npx @qanode/cli runs artifacts --run-id RUN_ID --out ./artifacts
```

El comando prioriza el informe PDF consolidado y lo guarda como:

```bash
report_<runId>.pdf
```

---

## Salida de la CLI

Sin `--json`, los comandos que devuelven un resultado final imprimen solamente:

```bash
success
failed
cancelled
```

Esto es intencional para simplificar el uso en pipelines.

Con `--json`, la CLI imprime el payload completo de la API.

### Diferencia importante entre `run ... --json` y `run ... --wait --json`

Sin `--wait`:

```bash
npx @qanode/cli run suite --suite-id SUITE_ID --json
```

devuelve el payload inicial de creación de la run, que contiene `runId`.

Con `--wait`:

```bash
npx @qanode/cli run suite --suite-id SUITE_ID --wait --json
```

devuelve el objeto final de la run, que contiene `id`.

> **Importante:** Para descargar el informe, reutilice el ID de la misma ejecución ya iniciada. No dispare una segunda run solo para descubrir el identificador.

---

## Rutas dedicadas de CI/CD

Además de la CLI, QANode expone rutas específicamente pensadas para automatización:

| Método | Ruta | Uso |
|--------|------|-----|
| `POST` | `/api/ci/runs/scenario` | Inicia una ejecución de escenario |
| `POST` | `/api/ci/runs/suite` | Inicia una ejecución de suite |
| `GET` | `/api/ci/runs/:id` | Consulta el estado actual y el resultado final |

Estas rutas usan el mismo token proporcionado en `QANODE_TOKEN`.

---

## Ejecución por ID o por nombre

Puede identificar el objetivo de dos formas:

### Por ID

Más precisa e ideal para automatizaciones estables.

```bash
--scenario-id
--suite-id
--project-id
```

### Por nombre

Más amigable para lectura humana.

```bash
--project-name
--scenario-name
--suite-name
```

Cuando use nombres, prefiera informar también el proyecto para evitar ambigüedades.

---

## Timeouts y polling

Las principales flags de automatización son:

| Flag | Uso |
|------|-----|
| `--wait` | Espera la finalización de la ejecución |
| `--timeout` | Define el tiempo máximo de espera en segundos |
| `--interval` | Intervalo entre consultas de estado |

Ejemplo:

```bash
npx @qanode/cli run suite \
  --suite-id SUITE_ID \
  --wait \
  --interval 5 \
  --timeout 600
```

---

## Códigos de salida

El proceso de la CLI devuelve:

| Código | Significado |
|--------|-------------|
| `0` | Ejecución completada con éxito |
| `2` | Ejecución completada con fallo o cancelación |
| `1` | Error de la CLI, autenticación o API |

Esto permite usar la CLI como gate de pipeline sin parsing complejo.

---

## Próximos Pasos

- [Overrides por Ejecución](./overrides.md) — Entienda cómo sobrescribir variables y credenciales solo para la run actual
