# Exemplos de Pipeline

> **Disponível em:** QANode Enterprise

Esta página traz exemplos prontos de integração com os provedores mais comuns.

---

## GitHub Actions

Exemplo completo rodando uma suíte, aguardando o resultado e baixando o relatório PDF consolidado.

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
            --suite-name "Regressão Login" \
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

### Por que capturar `run_id` desse jeito

O padrão acima evita um erro comum: disparar a suíte duas vezes.

O fluxo correto é:

1. iniciar a run
2. esperar a conclusão
3. extrair o `id` da mesma run retornada em JSON
4. baixar o relatório usando esse `id`

---

## Azure DevOps

Exemplo equivalente em Azure DevOps:

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
      --suite-name "Regressão Login" \
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

## Exemplo com override de variável

Use este padrão quando a aplicação sob teste é publicada dinamicamente durante o job:

```bash
npx @qanode/cli run suite \
  --project-name "Task Manager" \
  --suite-name "Suite Task Manager" \
  --var base_url_ci=https://preview-da-build.exemplo.com \
  --wait
```

---

## Exemplo com override de credencial

Use quando o pipeline receber um segredo temporário:

```bash
npx @qanode/cli run scenario \
  --project-name "Checkout" \
  --scenario-name "Login API" \
  --credential api-main.token=$API_TOKEN \
  --wait
```

---

## Relatório consolidado

O comando:

```bash
npx @qanode/cli runs artifacts --run-id RUN_ID --out ./artifacts
```

prioriza o relatório PDF consolidado da execução e salva o arquivo com nome único:

```bash
report_<runId>.pdf
```

Esse arquivo é o mais indicado para:

- anexar ao job
- auditar o resultado da execução
- enviar para outro sistema

---

## Recomendações operacionais

- Salve `QANODE_URL` e `QANODE_TOKEN` em **Secrets**
- Use `--wait` quando quiser que o pipeline falhe automaticamente junto com a execução
- Use `--json` quando precisar capturar o ID da run ou armazenar dados adicionais
- Use `--timeout` para evitar jobs presos por tempo excessivo
- Reutilize sempre o mesmo `run_id` ao baixar o relatório
