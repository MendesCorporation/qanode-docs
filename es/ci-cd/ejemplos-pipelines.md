# Ejemplos de Pipeline

> **Disponible en:** QANode Enterprise

Esta página trae ejemplos listos para usar con los proveedores de CI/CD más comunes.

---

## GitHub Actions

Ejemplo completo ejecutando una suite, esperando el resultado y descargando el informe PDF consolidado.

```yaml
name: Run QANode Suite

on:
  workflow_dispatch:
  push:
    branches: [main, master]

jobs:
  qanode:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Run QANode suite
        id: run_suite
        run: |
          set +e
          RUN_JSON=$(npx @qanode/cli run suite \
            --project-name "Backoffice" \
            --suite-name "Regresión Login" \
            --wait \
            --timeout 300 \
            --json)
          EXIT_CODE=$?
          set -e

          echo "$RUN_JSON" > qanode-run.json
          echo "run_id=$(echo "$RUN_JSON" | jq -r '.id')" >> "$GITHUB_OUTPUT"
          exit $EXIT_CODE
        env:
          QANODE_URL: ${{ secrets.QANODE_URL }}
          QANODE_TOKEN: ${{ secrets.QANODE_TOKEN }}

      - name: Download QANode report
        if: always()
        run: |
          npx @qanode/cli runs artifacts \
            --run-id "${{ steps.run_suite.outputs.run_id }}" \
            --out ./artifacts
        env:
          QANODE_URL: ${{ secrets.QANODE_URL }}
          QANODE_TOKEN: ${{ secrets.QANODE_TOKEN }}

      - name: Upload report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: qanode-report
          path: ./artifacts/
          if-no-files-found: ignore
```

### Por qué capturar `run_id` de esta forma

Este patrón evita un error común: disparar la suite dos veces.

El flujo correcto es:

1. iniciar la run
2. esperar a que termine
3. extraer el `id` de esa misma run devuelta en JSON
4. descargar el informe usando ese `id`

---

## Azure DevOps

Ejemplo equivalente en Azure DevOps:

```yaml
steps:
- task: NodeTool@0
  inputs:
    versionSpec: '20.x'
  displayName: Setup Node.js

- script: |
    set +e
    RUN_JSON=$(npx @qanode/cli run suite \
      --project-name "Backoffice" \
      --suite-name "Regresión Login" \
      --wait \
      --timeout 300 \
      --json)
    EXIT_CODE=$?
    set -e

    echo "$RUN_JSON" > qanode-run.json
    echo "##vso[task.setvariable variable=QANODE_RUN_ID]$(echo "$RUN_JSON" | jq -r '.id')"
    exit $EXIT_CODE
  displayName: Run QANode suite
  env:
    QANODE_URL: $(QANODE_URL)
    QANODE_TOKEN: $(QANODE_TOKEN)

- script: |
    npx @qanode/cli runs artifacts \
      --run-id "$(QANODE_RUN_ID)" \
      --out ./artifacts
  displayName: Download QANode report
  condition: always()
  env:
    QANODE_URL: $(QANODE_URL)
    QANODE_TOKEN: $(QANODE_TOKEN)
```

---

## Ejemplo con override de variable

Use este patrón cuando la aplicación bajo prueba se publica dinámicamente durante el job:

```bash
npx @qanode/cli run suite \
  --project-name "Task Manager" \
  --suite-name "Suite Task Manager" \
  --var base_url_ci=https://build-preview.example.com \
  --wait
```

---

## Ejemplo con override de credencial

Use esto cuando el pipeline reciba un secreto temporal:

```bash
npx @qanode/cli run scenario \
  --project-name "Checkout" \
  --scenario-name "Login API" \
  --credential api-main.token=$API_TOKEN \
  --wait
```

---

## Informe consolidado

El comando:

```bash
npx @qanode/cli runs artifacts --run-id RUN_ID --out ./artifacts
```

prioriza el PDF consolidado de la ejecución y lo guarda con un nombre único:

```bash
report_<runId>.pdf
```

Este archivo es la mejor opción para:

- adjuntarlo al job
- auditar el resultado de la ejecución
- enviarlo a otro sistema

---

## Recomendaciones operativas

- Guarde `QANODE_URL` y `QANODE_TOKEN` en **Secrets**
- Use `--wait` cuando quiera que el pipeline falle junto con la ejecución
- Use `--json` cuando necesite capturar el ID de la run o conservar metadatos adicionales
- Use `--timeout` para evitar jobs colgados durante demasiado tiempo
- Reutilice siempre el mismo `run_id` al descargar el informe
