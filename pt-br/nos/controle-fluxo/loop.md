# Nó Loop

O nó **Loop** permite repetir a execução de um trecho do fluxo múltiplas vezes. Suporta três modos de repetição: por contagem, por array e por condição (while).

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `loop` |
| **Categoria** | Controle de Fluxo |
| **Cor** | 🟡 Amarelo (#f59e0b) |
| **Entrada** | `in` |
| **Saídas** | `loop` (corpo do loop), `done` (saída após conclusão) |

---

## Modos de Loop

### 1. Contagem (Count)

Repete um número fixo de vezes.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Contagem** | `string/number` | Número de iterações |
| **Índice Inicial** | `number` | Valor inicial do índice (padrão: 0) |

**Exemplo:** Repetir 5 vezes
```
Contagem: 5
Índice Inicial: 0
```

A cada iteração, o índice é incrementado (0, 1, 2, 3, 4).

### 2. Array

Itera sobre cada item de um array.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Expressão do Array** | `string` | Expressão que retorna um array |

**Exemplo:** Iterar sobre resultados de uma query
```
{{ steps.query.outputs.rows }}
```

A cada iteração, o item atual e o índice ficam disponíveis nos outputs do loop.

### 3. Enquanto (While)

Repete enquanto uma condição for verdadeira.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Condição** | `string` | Expressão avaliada a cada iteração |
| **Máximo de Iterações** | `number` | Limite de segurança (padrão: 100) |

Assim como o nó If, o modo while suporta **modo simples** (expressão JavaScript) e **modo visual builder**.

**Exemplo:** Repetir enquanto houver próxima página
```javascript
{{ steps["http-request"].outputs.json.hasNextPage }} === true
```

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `_loopItem` | `any` | Item iterado no modo array |
| `_loopIndex` | `any` | Index da ultima iteração |
| `_loopCount` | `number` | Número total de iterações |

---

## Comportamento

1. O loop avalia se deve continuar (baseado no modo)
2. Se sim, executa todos os nós conectados à saída **loop**
3. Ao final do corpo do loop, retorna ao nó Loop para a próxima iteração
4. Quando o loop termina, executa os nós conectados à saída **done**

### Fluxo Visual

```
        ┌─────────────────────────────────┐
        │                                 │
        ▼                                 │
[Loop] ──loop──→ [Ação 1] → [Ação 2] ────┘
   │
   done
   │
   ▼
[Próximo Nó]
```

> **Nota:** Nós no corpo do loop são executados a cada iteração. Use expressões para acessar o item/índice atual.

---

## Exemplo Prático

### Iterar sobre resultados de banco de dados

```
[PostgreSQL: SELECT * FROM users WHERE active = true]
    │
    ▼
[Loop (array): {{ steps.query.outputs.rows }}]
    │ loop → [HTTP Request: POST /api/notify]
    │             Body: { "email": "{{ steps.loop.outputs._loopItems[steps.loop.outputs._loopCount].email }}" }
    │
    │ done → [Log: "Todas as notificações enviadas"]
```

### Loop com contagem

```
[Loop (count): 3 vezes]
    │ loop → [HTTP Request: GET /api/retry-endpoint]
    │             → [If: status === 200]
    │                   true → (break/success)
    │
    │ done → [Log: "Tentativas esgotadas"]
```

---

## Dicas

- Sempre configure o **máximo de iterações** no modo while para evitar loops infinitos
- Para arrays grandes, considere o impacto de performance
- Use nós de **Log** dentro do loop para acompanhar o progresso
- No modo array, os itens do array ficam acessíveis via `_loopItems`
