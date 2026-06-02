# Nó Arquivo para Base64

O nó **Arquivo para Base64** recebe um `fileRef` e devolve o conteúdo do arquivo em Base64 e Data URI.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `file-to-base64` |
| **Categoria** | Arquivos |
| **Cor** | 🟤 Dourado (#C89F65) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Quando Usar

Use este nó quando:

- uma API exige arquivo em Base64;
- você precisa montar payload JSON com arquivo embutido;
- um sistema externo espera Data URI;
- você precisa registrar ou comparar o conteúdo codificado.

Quando a API aceita upload multipart ou binário, normalmente é melhor enviar o `fileRef` diretamente pelo nó [HTTP Request](../api/http-request.md), sem converter para Base64.

---

## Configuração

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Arquivo** | `fileRef` | Arquivo de entrada |

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `base64` | `string` | Conteúdo Base64 puro |
| `dataUri` | `string` | Conteúdo no formato `data:mime/type;base64,...` |
| `name` | `string` | Nome do arquivo |
| `mimeType` | `string` | MIME type |
| `sizeBytes` | `number` | Tamanho em bytes |
| `fileRef` | `fileRef` | Referência original do arquivo |

### Acessando os outputs

```
{{ steps["file-to-base64"].outputs.base64 }}
{{ steps["file-to-base64"].outputs.dataUri }}
```

---

## Exemplo: Enviar Base64 em JSON

```
[Gerar Arquivo] → [Arquivo para Base64] → [HTTP Request]
```

No HTTP Request:

```json
{
  "fileName": "{{ steps["file-to-base64"].outputs.name }}",
  "mimeType": "{{ steps["file-to-base64"].outputs.mimeType }}",
  "content": "{{ steps["file-to-base64"].outputs.base64 }}"
}
```

---

## Dicas

- Use `dataUri` quando o destino espera o prefixo `data:mime/type;base64,`.
- Use `base64` quando o destino espera apenas o conteúdo codificado.
- Evite converter arquivos grandes para Base64 sem necessidade, pois o texto fica maior que o arquivo original.
- Para uploads comuns, prefira `fileRef` direto no nó que aceita arquivo.
