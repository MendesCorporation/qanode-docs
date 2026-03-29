# Nó If

O nó **If** permite criar desvios condicionais no fluxo. Ele avalia uma expressão e direciona a execução para o caminho **true** (verdadeiro) ou **false** (falso).

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `if` |
| **Categoria** | Controle de Fluxo |
| **Cor** | 🟡 Amarelo (#f59e0b) |
| **Entrada** | `in` |
| **Saídas** | `true`, `false` |

---

## Configuração

### Modo Simples

No modo simples, você escreve uma expressão JavaScript que será avaliada como verdadeira ou falsa:

**Campo: Condição**
```javascript
{{ steps["http-request"].outputs.status === 200 }} 
```

A expressão tem acesso ao contexto de execução:
- `steps` — Outputs de nós anteriores
- `variables` — Variáveis globais

**Exemplos:**

```javascript
// Verificar status HTTP
{{ steps["http-request"].outputs.status === 200}} 

// Verificar se texto contém valor
{{ steps.extract.outputs.extracts.title.includes("Bem-vindo") }}

// Verificar valor numérico
{{ steps.query.outputs.rowCount > 0 }} 

// Verificar booleano
{{ variables.featureEnabled === true }} 

// Combinação com AND/OR
{{ steps.api.outputs.status  === 200 && steps.api.outputs.json.active  === true }}
```

### Modo Visual Builder

O modo visual permite construir condições sem escrever código:

| Campo | Descrição | Exemplo |
|-------|-----------|---------|
| **Campo** | Expressão a ser avaliada | `steps["http-request"].outputs.status` |
| **Operador** | Operação de comparação | `===` |
| **Valor** | Valor esperado | `200` |
| **Lógica** | Operador entre condições | `AND` / `OR` |

**Operadores disponíveis:**

| Operador | Descrição |
|----------|-----------|
| `===` | Igual (estrito) |
| `!==` | Diferente (estrito) |
| `==` | Igual (com conversão) |
| `!=` | Diferente (com conversão) |
| `>` | Maior que |
| `<` | Menor que |
| `>=` | Maior ou igual |
| `<=` | Menor ou igual |
| `includes` | Contém (para strings) |
| `startsWith` | Começa com |
| `endsWith` | Termina com |

Você pode adicionar múltiplas condições combinadas com **AND** (todas devem ser verdadeiras) ou **OR** (pelo menos uma deve ser verdadeira).

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `_branch` | `string` | O caminho tomado: `"true"` ou `"false"` |

---

## Comportamento

1. A expressão é avaliada usando os dados do contexto atual
2. Se o resultado for **truthy** (verdadeiro, número > 0, string não vazia), a saída `true` é ativada
3. Se o resultado for **falsy** (false, 0, null, undefined, string vazia), a saída `false` é ativada
4. Nós conectados ao caminho **não ativado** serão **pulados**

---

## Exemplo Prático

### Verificar resposta de API

```
[HTTP Request: GET /api/user/1]
    │
    ▼
[If: status === 200]
    │ true → [Log: "Usuário encontrado: {{ steps["http-request"].outputs.json.name }}"]
    │ false → [Log: "Usuário não encontrado"] → [Stop and Fail]
```

### Validação de dados

```
[PostgreSQL: SELECT count(*) FROM orders WHERE user_id = '123']
    │
    ▼
[If: rowCount > 0]
    │ true → [Log: "Pedidos encontrados"]
    │ false → [Log: "Nenhum pedido"]
```

---

## Dicas

- Use o **modo visual builder** quando possível — é mais legível e menos propenso a erros
- Nomeie o nó de forma descritiva: "Se login bem-sucedido", "Se registro existe"
- Lembre-se que expressões `{{ }}` são avaliadas antes da comparação
- Para condições complexas, considere usar um nó **Custom JavaScript** antes e avaliar o resultado no If
