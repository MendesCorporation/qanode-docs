# CLI e API do CI/CD

> **Disponível em:** QANode Enterprise

Esta página mostra como usar a CLI oficial do QANode e as rotas de CI/CD para integrar o produto com pipelines corporativos.

---

## CLI oficial

O pacote oficial é:

```bash
@qanode/cli
```

Você pode usar sem instalação global:

```bash
npx @qanode/cli help
```

Ou instalar como dependência de desenvolvimento:

```bash
npm install --save-dev @qanode/cli
```

Depois disso:

```bash
npx qanode help
```

> **Recomendação:** Em pipelines, `npx @qanode/cli ...` costuma ser suficiente e evita depender de instalação global prévia.

---

## Variáveis de ambiente

As duas variáveis principais são:

| Variável | Descrição |
|----------|-----------|
| `QANODE_URL` | URL pública do QANode |
| `QANODE_TOKEN` | Token de integração ou token de sessão |

Exemplo:

```bash
export QANODE_URL=https://qanode.empresa.com
export QANODE_TOKEN=qnt_xxxxx
```

### Sobre a URL

O valor de `QANODE_URL` deve ser o endereço público já usado pelo time no navegador.

Não é necessário conhecer a porta interna da API quando sua empresa usa:

- reverse proxy
- balanceador
- domínio único para frontend + backend

Em desenvolvimento local, `http://localhost:3000` pode funcionar quando o frontend estiver encaminhando `/api`.

---

## Comandos principais

### Validar autenticação

```bash
npx @qanode/cli auth me
```

### Rodar cenário por ID

```bash
npx @qanode/cli run scenario --scenario-id SCENARIO_ID --wait
```

### Rodar cenário por projeto e nome

```bash
npx @qanode/cli run scenario \
  --project-name "Checkout" \
  --scenario-name "Login API" \
  --wait
```

### Rodar suíte por ID

```bash
npx @qanode/cli run suite --suite-id SUITE_ID --wait
```

### Rodar suíte por projeto e nome

```bash
npx @qanode/cli run suite \
  --project-name "Backoffice" \
  --suite-name "Regressão Login" \
  --wait
```

### Consultar execução

```bash
npx @qanode/cli runs get --run-id RUN_ID
```

### Aguardar uma execução existente

```bash
npx @qanode/cli runs wait --run-id RUN_ID --timeout 300
```

### Baixar relatório consolidado

```bash
npx @qanode/cli runs artifacts --run-id RUN_ID --out ./artifacts
```

O comando prioriza o PDF consolidado e salva o arquivo como:

```bash
report_<runId>.pdf
```

---

## Saída da CLI

Sem `--json`, os comandos que retornam resultado final imprimem apenas:

```bash
success
failed
cancelled
```

Isso é intencional para facilitar o uso em pipeline.

Com `--json`, a CLI imprime o payload completo da API.

### Diferença importante entre `run ... --json` e `run ... --wait --json`

Sem `--wait`:

```bash
npx @qanode/cli run suite --suite-id SUITE_ID --json
```

retorna o payload inicial da criação da run, contendo `runId`.

Com `--wait`:

```bash
npx @qanode/cli run suite --suite-id SUITE_ID --wait --json
```

retorna a run final, contendo `id`.

> **Importante:** Para baixar o relatório, reaproveite o ID da mesma execução já iniciada. Não dispare uma segunda run só para descobrir o identificador.

---

## Rotas dedicadas de CI/CD

Além da CLI, o QANode oferece rotas específicas para automação:

| Método | Rota | Uso |
|--------|------|-----|
| `POST` | `/api/ci/runs/scenario` | Inicia uma execução de cenário |
| `POST` | `/api/ci/runs/suite` | Inicia uma execução de suíte |
| `GET` | `/api/ci/runs/:id` | Consulta o status e o resultado final |

Essas rotas usam o mesmo token informado em `QANODE_TOKEN`.

---

## Execução por ID ou por nome

Você pode identificar o alvo de duas formas:

### Por ID

Mais precisa e ideal para automações estáveis.

```bash
--scenario-id
--suite-id
--project-id
```

### Por nome

Mais amigável para leitura humana.

```bash
--project-name
--scenario-name
--suite-name
```

Quando usar nomes, prefira informar também o projeto para evitar ambiguidades.

---

## Timeouts e polling

Os parâmetros principais para automação são:

| Flag | Uso |
|------|-----|
| `--wait` | Espera o término da execução |
| `--timeout` | Define o tempo máximo de espera em segundos |
| `--interval` | Intervalo entre consultas de status |

Exemplo:

```bash
npx @qanode/cli run suite \
  --suite-id SUITE_ID \
  --wait \
  --interval 5 \
  --timeout 600
```

---

## Códigos de saída

O processo do CLI retorna:

| Código | Significado |
|--------|-------------|
| `0` | Execução concluída com sucesso |
| `2` | Execução concluída com falha ou cancelamento |
| `1` | Erro do CLI, da autenticação ou da API |

Isso permite usar a CLI como gate de pipeline sem parsing complexo.

---

## Próximos Passos

- [Overrides por Execução](./overrides.md) — Entenda como sobrescrever variáveis e credenciais só para a run atual
