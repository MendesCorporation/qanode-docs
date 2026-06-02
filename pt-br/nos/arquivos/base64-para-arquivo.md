# Nó Base64 para Arquivo

O nó **Base64 para Arquivo** recebe conteúdo Base64 ou Data URI e cria um arquivo dentro do QANode, retornando um `fileRef` para uso nos próximos nós.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `file-from-base64` |
| **Categoria** | Arquivos |
| **Cor** | 🟤 Dourado (#C89F65) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Quando Usar

Use este nó quando:

- uma API retorna arquivo em Base64;
- um sistema envia Data URI, como `data:image/png;base64,...`;
- você precisa converter Base64 em arquivo para extrair conteúdo;
- outro nó espera `fileRef`, mas a origem está em Base64.

---

## Configuração

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Nome do arquivo** | texto | Nome final do arquivo, incluindo extensão |
| **Base64** | texto | Conteúdo Base64 puro ou Data URI |

O usuário não precisa informar MIME type. O QANode tenta inferir automaticamente pelo Data URI, assinatura do conteúdo ou nome do arquivo.

---

## Base64 Puro vs Data URI

### Base64 puro

```
iVBORw0KGgoAAAANSUhEUg...
```

### Data URI

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUg...
```

Quando o valor é Data URI, o MIME type costuma ser detectado com mais precisão.

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `fileRef` | `fileRef` | Referência do arquivo criado |
| `name` | `string` | Nome do arquivo |
| `mimeType` | `string` | MIME type detectado |
| `sizeBytes` | `number` | Tamanho em bytes |

### Acessando os outputs

```
{{ steps["file-from-base64"].outputs.fileRef }}
{{ steps["file-from-base64"].outputs.mimeType }}
```

---

## Exemplo

```
Nome do arquivo: resposta.pdf
Base64: {{ steps["http-request"].outputs.body.arquivoBase64 }}
```

Depois:

```
[Base64 para Arquivo] → [Extrair Arquivo]
```

---

## Dicas

- Sempre informe um nome com extensão quando possível.
- Use Data URI quando a origem já trouxer MIME type.
- Para validar o conteúdo, encadeie com **Extrair Arquivo**.
- Para enviar o arquivo em upload, use o output `fileRef`.

