# Nó Stop and Fail

O nó **Stop and Fail** interrompe imediatamente a execução do fluxo e marca o resultado como **falha**. É um nó terminal — não possui saídas.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `stop-and-fail` |
| **Categoria** | Utilitários |
| **Cor** | ⚪ Cinza (#6b7280) |
| **Entrada** | `in` |
| **Saídas** | Nenhuma (nó terminal) |

---

## Configuração

Este nó não possui campos de configuração. Ao ser atingido, simplesmente para a execução.

---

## Uso

O Stop and Fail é tipicamente usado no fim de caminhos condicionais que representam cenários de erro:

```
[If: {{ steps.api.outputs.status }} === 200]
    │ true → [Continuar testes...]
    │ false → [Log: "API retornou erro"] → [Stop and Fail]
```

```
[Switch: {{ steps.query.outputs.rowCount }}]
    │ case-0 (0) → [Log: "Nenhum dado encontrado"] → [Stop and Fail]
    │ default → [Processar dados...]
```

---

## Dicas

- Combine com um nó **Log** antes para registrar o motivo da falha
- Use após validações críticas que devem impedir a continuação do fluxo
- O status da execução inteira será marcado como **failed**
