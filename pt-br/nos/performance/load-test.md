# Nó Load Test

O nó **Load Test** executa testes de carga e performance contra endpoints HTTP, simulando múltiplos usuários virtuais (VUs) em paralelo. Gera evidências visuais em PNG com métricas detalhadas e suporta critérios de aprovação automáticos via limiares.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `load-test` |
| **Categoria** | Performance |
| **Cor** | 🟫 Âmbar (#92400E) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Tipos de Teste

Cada tipo de teste gera um perfil de carga diferente automaticamente a partir dos campos **VUs** e **Duração**.

| Tipo | Descrição | Quando Usar |
|------|-----------|-------------|
| **Smoke** | 1 VU, duração configurada | Validar que o endpoint responde antes de rodar testes maiores |
| **Load** | Ramp up → sustentação → ramp down | Verificar comportamento sob carga normal esperada |
| **Stress** | Ramp agressivo até o limite | Identificar o ponto onde o sistema começa a degradar |
| **Spike** | Pico súbito de VUs | Testar reação a picos repentinos de tráfego (ex: Black Friday) |
| **Soak** | Carga sustentada por longa duração | Detectar memory leaks e degradação gradual |
| **Breakpoint** | Escada progressiva de VUs | Encontrar o ponto exato de ruptura do sistema |

---

## Configuração

### Requisição

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Credencial** | `string` | Credencial HTTP salva (preenche URL e auth automaticamente) |
| **Método** | `string` | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| **URL** | `string` | Endpoint alvo (suporta `{{ }}`) |
| **Auth** | `string` | Tipo de autenticação manual (se não usar credencial) |
| **Headers** | `object` | Cabeçalhos adicionais da requisição |
| **Body** | `any` | Corpo da requisição (para POST, PUT, PATCH) |

### Configuração de Carga

| Campo | Tipo | Padrão | Descrição |
|-------|------|--------|-----------|
| **VUs** | `number` | `10` | Número de usuários virtuais simultâneos |
| **Duração (s)** | `number` | `30` | Duração total do teste em segundos |
| **Think Time (ms)** | `number` | `0` | Pausa entre requisições de cada VU |
| **Timeout (ms)** | `number` | `30000` | Tempo máximo de espera por resposta |

> Para o tipo **Smoke**, o campo VUs é fixo em 1 e não é exibido.  
> Para o tipo **Soak**, o padrão de duração é 1800s (30 min).  
> Para o tipo **Breakpoint**, o campo VUs representa o **máximo** de VUs que serão atingidos.

#### Como o Breakpoint funciona

O teste divide a duração total em passos de ~30s, aumentando os VUs progressivamente:

```
VUs: 10 | Duração: 60s  →  2 passos de 30s
  Passo 1: 0s → 30s  →  5 VUs
  Passo 2: 30s → 60s →  10 VUs
```

### Stages Customizadas

Ative **Custom** na seção Stages para definir manualmente o perfil de carga:

| Campo | Descrição |
|-------|-----------|
| **Duração (s)** | Duração desta stage em segundos |
| **Target VUs** | Número de VUs ao final desta stage |

Exemplo de stages para um teste stress manual:

| Duração | Target VUs | Descrição |
|---------|-----------|-----------|
| 30s | 10 | Ramp up inicial |
| 60s | 50 | Carga sustentada |
| 30s | 100 | Stress |
| 15s | 0 | Ramp down |

---

## Autenticação

### Usando Credenciais Salvas

Selecione uma credencial do tipo **HTTP/API**. A URL base e os dados de autenticação são aplicados automaticamente:

1. Selecione a credencial no campo **Credencial**
2. A URL base é preenchida automaticamente no campo **URL**
3. Complete com o path do endpoint: `/api/checkout`

### Autenticação Manual

| Tipo | Campos | Resultado |
|------|--------|-----------|
| **Bearer Token** | Token | Header `Authorization: Bearer {token}` |
| **Basic Auth** | Usuário + Senha | Header `Authorization: Basic {base64}` |
| **API Key** | Header Name + Token | Header customizado com o token |

---

## Limiares (Thresholds)

Limiares definem critérios de aprovação automáticos. Se qualquer limiar não for atingido, o nó é marcado como **FALHOU**.

| Métrica | Descrição |
|---------|-----------|
| `p50` | Percentil 50 de latência (ms) |
| `p95` | Percentil 95 de latência (ms) |
| `p99` | Percentil 99 de latência (ms) |
| `avgDuration` | Latência média (ms) |
| `errorRate` | Taxa de erros (%) |
| `rps` | Requisições por segundo |

| Operador | Significado |
|----------|-------------|
| `<` | Menor que |
| `≤` | Menor ou igual |
| `>` | Maior que |
| `≥` | Maior ou igual |

**Exemplos de limiares comuns:**

| Limiar | Significado |
|--------|-------------|
| `p95 < 500` | 95% das respostas em menos de 500ms |
| `errorRate < 1` | Taxa de erro menor que 1% |
| `rps > 10` | Mínimo de 10 requisições por segundo |
| `p99 < 2000` | 99% das respostas em menos de 2s |

> Sem limiares configurados, o nó sempre passa (desde que o endpoint responda).

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `passed` | `boolean` | `true` se todos os limiares foram atingidos |
| `testType` | `string` | Tipo de teste executado |
| `metrics` | `object` | Métricas consolidadas do teste |
| `thresholds` | `array` | Resultado de cada limiar configurado |
| `stages` | `array` | Stages executadas (auto ou customizadas) |

### Estrutura do objeto `metrics`

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

### Acessando os Outputs

```
// Verificar se passou
{{ steps["load-test"].outputs.passed }}  →  true

// Total de requisições
{{ steps["load-test"].outputs.metrics.totalRequests }}  →  1141

// Latência p95
{{ steps["load-test"].outputs.metrics.p95 }}  →  451

// Taxa de erro
{{ steps["load-test"].outputs.metrics.errorRate }}  →  0.0

// RPS
{{ steps["load-test"].outputs.metrics.rps }}  →  34.49
```

---

## Evidências Geradas

O nó gera automaticamente **dois gráficos PNG** como evidência da execução:

### 1. Relatório de Resumo (`load-test-report.png`)

Visão consolidada com:
- Cards de métricas (p50, p95, p99, Avg, RPS, Requests, Errors, Error Rate)
- Gráfico de barras horizontais com distribuição de latência
- Gráfico de throughput ao longo do tempo (req/s)
- Pills de status de cada limiar configurado
- Rodapé com stages executadas

### 2. Timeline RPS × Latência (`load-test-timeline.png`)

Gráfico dual-axis com:
- **Barras azuis** (eixo esquerdo): RPS ao longo do tempo
- **Linha laranja** (eixo direito): Latência média ao longo do tempo
- **Linha roxa tracejada** (eixo direito): Latência p95 ao longo do tempo

> O gráfico de timeline é especialmente útil para **Breakpoint** e **Stress**, onde é possível visualizar exatamente em que momento a latência começa a subir em resposta ao aumento de carga.

---

## Exemplos Práticos

### Smoke test — Validação básica

```
Tipo: Smoke
URL: https://api.exemplo.com/health
Método: GET
Duração: 10s
```

Ideal para rodar no início de um fluxo de testes — garante que o ambiente está no ar.

### Load test — Carga normal com limiares

```
Tipo: Load
URL: https://api.exemplo.com/products
Método: GET
VUs: 20
Duração: 60s
Limiares:
  - p95 < 500
  - errorRate < 1
```

### Stress test — Limites do sistema

```
Tipo: Stress
URL: https://api.exemplo.com/checkout
Método: POST
VUs: 100
Duração: 120s
Body: { "productId": "123", "quantity": 1 }
Auth: Bearer → {{ variables.API_TOKEN }}
Limiares:
  - p99 < 2000
  - errorRate < 5
```

### Breakpoint — Ponto de ruptura

```
Tipo: Breakpoint
URL: https://api.exemplo.com/search
Método: GET
Max VUs: 200
Duração: 300s
Limiares:
  - p95 < 1000
  - errorRate < 2
```

O sistema irá aumentar progressivamente de 1 até 200 VUs em passos de ~30s. Quando p95 ultrapassar 1000ms ou erros superarem 2%, o nó marca como falha — indicando o ponto de ruptura.

### Encadeando com outros nós

```
[Load Test: Smoke]
    │ outputs.passed = true
    ▼
[If: {{ steps.smoke.outputs.passed }}]
    │ true  → [Load Test: Load completo]
    │ false → [Log: "Smoke falhou — ambiente indisponível"]
```

---

## Isolamento de Fila

O nó Load Test é executado em uma fila separada (`qanode-load-tests`) para não interferir nos outros fluxos em execução.

Para configurar um worker dedicado ao Load Test:

```bash
WORKER_QUEUES=load-tests node dist/start.js
```

Para um worker que processa ambas as filas:

```bash
WORKER_QUEUES=executions,load-tests node dist/start.js
```

---

## Dicas

- **Comece pelo Smoke** antes de rodar testes de carga — garante que o endpoint está respondendo corretamente
- **Configure limiares** para que o teste falhe automaticamente quando o sistema degradar, sem precisar analisar os números manualmente
- **Use o gráfico de timeline** para identificar o momento exato de degradação em testes Breakpoint e Stress
- **Think Time** simula comportamento humano — útil para testes Soak onde você quer carga contínua mas realista
- **Credenciais salvas** facilitam a execução em diferentes ambientes (staging, produção) sem alterar o fluxo
- Para testes confiáveis, **até 100 VUs** por instância de worker. Acima disso, considere um worker dedicado
