# Nó Log

O nó **Log** registra uma mensagem no log de execução. Útil para depuração, acompanhamento do progresso e documentação de valores durante a execução.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `log` |
| **Categoria** | Utilitários |
| **Cor** | ⚪ Cinza (#6b7280) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Configuração

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Mensagem** | `string` | Texto a ser registrado (suporta `{{ }}`) |

---

## Exemplos

### Log simples

```
Mensagem: Teste iniciado com sucesso
```

### Log com dados dinâmicos

```
Mensagem: Usuário encontrado: {{ steps.query.outputs.rows[0].name }} ({{ steps.query.outputs.rows[0].email }})
```

### Log para depuração

```
Mensagem: Status da API: {{ steps['HTTP Request'].outputs.status }}, Body: {{ JSON.stringify(steps['HTTP Request'].outputs.json) }}
```

---

## Dicas

- Use logs estrategicamente para **documentar valores** em pontos críticos do fluxo
- As mensagens aparecem nos **resultados da execução** e no **relatório PDF**
- Combine com **If/Switch** para logs condicionais:
  ```
  [If: status === 200]
      │ true → [Log: "Sucesso"]
      │ false → [Log: "Falha: status {{ steps.api.outputs.status }}"]
  ```
