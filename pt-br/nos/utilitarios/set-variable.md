# Nó Set Variable

O nó **Set Variable** permite definir ou atualizar variáveis em tempo de execução. As variáveis definidas ficam disponíveis para todos os nós seguintes no fluxo.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `set-variable` |
| **Categoria** | Utilitários |
| **Cor** | ⚪ Cinza (#6b7280) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Configuração

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Chave** | `string` | Nome da variável |
| **Valor** | `any` | Valor a ser atribuído (suporta `{{ }}`) |

---

## Uso

### Definir variável simples

```
Chave: token
Valor: {{ steps.login.outputs.json.token }}
```

### Definir variável com cálculo

```
Chave: totalPrice
Valor: {{ steps.cart.outputs.json.subtotal * 1.1 }}
```

### Acessar a variável em nós seguintes

Após definir, a variável fica acessível via:
```
{{ variables.token }}
{{ variables.totalPrice }}
```

---

## Exemplos

### Salvar token de autenticação

```
[HTTP Request: POST /login] → [Set Variable: token = {{ steps.login.outputs.json.token }}]
    │
    ▼
[HTTP Request: GET /profile → Header: Authorization = Bearer {{ variables.token }}]
```

### Preparar URL dinâmica

```
[Set Variable: baseUrl = https://{{ variables.ENVIRONMENT }}.exemplo.com]
    │
    ▼
[HTTP Request: GET {{ variables.baseUrl }}/api/users]
```

---

## Dicas

- Use **nomes descritivos** para variáveis: `authToken` é melhor que `t`
- Variáveis definidas com Set Variable **sobrescrevem** variáveis globais de mesmo nome durante a execução
- O valor suporta **qualquer expressão** `{{ }}`, incluindo cálculos e acesso a outputs
