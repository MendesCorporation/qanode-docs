# Nó Switch

O nó **Switch** permite criar desvios múltiplos baseados no valor de uma expressão. É útil quando há mais de dois caminhos possíveis (onde o If não seria suficiente).

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `switch` |
| **Categoria** | Controle de Fluxo |
| **Cor** | 🟡 Amarelo (#f59e0b) |
| **Entrada** | `in` |
| **Saídas** | `case-0`, `case-1`, ..., `default` (dinâmicas) |

---

## Configuração

### Modo Simples

No modo simples, você define uma expressão e valores para cada case:

**Campo: Expressão**
```javascript
{{ steps['HTTP Request'].outputs.status }}
```

**Cases:**

| Case | Valor | Label (Opcional) |
|------|-------|------------------|
| Case 0 | `200` | Sucesso |
| Case 1 | `404` | Não Encontrado |
| Case 2 | `500` | Erro do Servidor |
| Default | — | Outros |

A expressão é avaliada e comparada com cada valor de case. Se nenhum case corresponder, o caminho `default` é ativado.

### Modo Visual Builder

O modo visual permite usar operadores diferentes para cada case:

| Campo | Descrição |
|-------|-----------|
| **Campo de Comparação** | Expressão a ser avaliada |
| **Operador** | Operação de comparação por case |
| **Valor** | Valor esperado |

**Operadores disponíveis:**

| Operador | Descrição |
|----------|-----------|
| `===` | Igual (estrito) |
| `!==` | Diferente |
| `>` | Maior que |
| `<` | Menor que |
| `>=` | Maior ou igual |
| `<=` | Menor ou igual |
| `includes` | Contém |
| `startsWith` | Começa com |
| `endsWith` | Termina com |

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `_branch` | `string` | O nome do case ativado |

---

## Comportamento

1. A expressão é avaliada
2. O resultado é comparado com cada case na ordem
3. O **primeiro case** que corresponder terá seu caminho ativado
4. Se nenhum case corresponder, o caminho `default` é ativado
5. Nós nos caminhos não ativados serão **pulados**

---

## Exemplo Prático

### Roteamento por status HTTP

```
[HTTP Request]
    │
    ▼
[Switch: {{ steps['HTTP Request'].outputs.status }}]
    │ case-0 (200) → [Log: "Sucesso"] → [Extrair dados]
    │ case-1 (401) → [Log: "Não autorizado"] → [Refazer login]
    │ case-2 (404) → [Log: "Não encontrado"]
    │ default → [Log: "Erro inesperado"] → [Stop and Fail]
```

### Roteamento por tipo de usuário

```
[PostgreSQL: SELECT role FROM users WHERE id = '123']
    │
    ▼
[Switch: {{ steps.query.outputs.rows[0].role }}]
    │ case-0 ("admin") → [Fluxo Admin]
    │ case-1 ("user") → [Fluxo Usuário]
    │ default → [Fluxo Visitante]
```

---

## Dicas

- Use **labels descritivos** nos cases para facilitar a leitura do fluxo
- O case `default` sempre deve ser conectado para tratar valores inesperados
- Para apenas dois caminhos, prefira o nó [If](if.md)
- Cases são avaliados na ordem — coloque os mais comuns primeiro
