# Nó Wait

O nó **Wait** pausa a execução do fluxo por um tempo determinado ou até que uma condição seja atendida.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `wait` |
| **Categoria** | Utilitários |
| **Cor** | ⚪ Cinza (#6b7280) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Modos

### Duração (Duration)

Aguarda por um tempo fixo em milissegundos.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Duração (ms)** | `number` | Tempo de espera em milissegundos |

**Exemplo:** Aguardar 5 segundos
```
Modo: duration
Duração: 5000
```

### Até Condição (Until)

Aguarda até que uma condição se torne verdadeira, verificando periodicamente.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Condição** | `string` | Expressão a ser avaliada |
| **Intervalo (ms)** | `number` | Intervalo entre verificações (padrão: 1000) |
| **Timeout (ms)** | `number` | Tempo máximo de espera (padrão: 30000) |

**Exemplo:** Aguardar até que variável esteja definida
```
Modo: until
Condição: {{ variables.processComplete === true }} 
Intervalo: 2000
Timeout: 60000
```

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `waited` | `number` | Tempo real de espera (ms) |
| `conditionMet` | `boolean` | Se a condição foi atendida (modo until) |

---

## Exemplos Práticos

### Aguardar entre requisições

```
[HTTP Request: POST /start-job]
    │
    ▼
[Wait: 10000ms]
    │
    ▼
[HTTP Request: GET /job-status]
```

### Polling até conclusão

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

## Dicas

- Use **duração** para esperas simples (ex: aguardar propagação)
- Use **até condição** para polling (verificar status periodicamente)
- Sempre defina um **timeout** no modo "until" para evitar esperas infinitas
