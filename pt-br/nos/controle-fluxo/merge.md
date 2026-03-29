# Nó Merge

O nó **Merge** permite juntar múltiplos caminhos de execução em um único ponto. É útil para reconectar caminhos que foram divididos por um nó If ou Switch.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `merge` |
| **Categoria** | Controle de Fluxo |
| **Cor** | 🟡 Amarelo (#f59e0b) |
| **Entradas** | `in1`, `in2` |
| **Saída** | `out` |

---

## Configuração

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Modo** | `string` | Estratégia de merge |

---

## Comportamento

O nó Merge aguarda que pelo menos um dos caminhos de entrada seja ativado antes de prosseguir. Quando um caminho condicional (If/Switch) ativa apenas um ramo, o Merge reconhece que o outro ramo foi pulado e não fica aguardando indefinidamente.

---

## Exemplo Prático

### Reconectando após If

```
                   ┌── true ──→ [Processar A] ──┐
[If: condição] ────┤                            ├──→ [Merge] → [Log: "Continuando"]
                   └── false ─→ [Processar B] ──┘
```

Neste exemplo:
1. O nó If avalia a condição
2. Apenas um dos caminhos (A ou B) será executado
3. O Merge recebe o resultado do caminho ativado
4. O fluxo continua normalmente após o Merge

### Reconectando após Switch

```
[Switch] ──case-0──→ [Ação 1] ──┐
         ──case-1──→ [Ação 2] ──┼──→ [Merge] → [Próximo]
         ──default─→ [Ação 3] ──┘
```

---

## Dicas

- Use Merge sempre que precisar reconectar caminhos divididos
- O Merge é especialmente útil para evitar duplicação de nós que devem executar independentemente do caminho tomado
- Nomeie o Merge de forma descritiva: "Merge após validação", "Juntar caminhos"
