# Nó Gerar Arquivo

O nó **Gerar Arquivo** cria arquivos dentro da execução do QANode. Ele pode gerar TXT, JSON, CSV, Excel ou PDF a partir de conteúdo informado diretamente no painel ou vindo de expressões de nós anteriores.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `file-generate` |
| **Categoria** | Arquivos |
| **Cor** | 🟤 Dourado (#C89F65) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Quando Usar

Use este nó quando o fluxo precisa:

- criar uma massa de dados em CSV ou Excel;
- gerar um JSON para enviar a uma API;
- montar um arquivo TXT de apoio;
- criar um PDF simples para anexar, baixar ou validar;
- produzir um arquivo que será usado por HTTP Request, Smart Web Flow, Mobile Flow, SSH, componente ou Custom JavaScript.

---

## Configuração

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Tipo de arquivo** | seleção | TXT, JSON, CSV, Excel ou PDF |
| **Nome do arquivo** | texto | Nome final do arquivo, incluindo extensão |

Os demais campos mudam conforme o tipo escolhido.

### TXT

| Campo | Descrição |
|-------|-----------|
| **Conteúdo** | Texto do arquivo. Aceita expressões `{{ }}` |

### JSON

| Campo | Descrição |
|-------|-----------|
| **JSON** | Objeto ou array JSON válido |

### CSV

| Campo | Descrição |
|-------|-----------|
| **Modo** | Linhas manuais ou a partir de lista |
| **Colunas** | Nomes das colunas |
| **Linhas** | Valores de cada linha, quando o modo é manual |
| **Lista** | Array vindo de outro nó, quando o modo é a partir de lista |
| **Mapeamento** | Campo do item usado em cada coluna |

### Excel

| Campo | Descrição |
|-------|-----------|
| **Nome da aba** | Nome da planilha dentro do arquivo |
| **Modo** | Linhas manuais ou a partir de lista |
| **Colunas / Linhas / Lista** | Mesma lógica do CSV |

### PDF

| Campo | Descrição |
|-------|-----------|
| **Título** | Título opcional exibido no PDF |
| **Conteúdo** | Texto ou JSON a renderizar no PDF |

---

## Tabela Manual e a Partir de Lista

Para CSV e Excel, existem dois jeitos principais de montar a tabela.

### Linhas manuais

Use quando os dados são pequenos ou fixos.

```
Colunas: id, nome, email
Linha 1: 1, Maria, maria@empresa.com
Linha 2: 2, João, joao@empresa.com
```

### A partir de lista

Use quando os dados já vêm de uma API, banco, componente ou nó anterior.

```
Lista: {{ steps["consulta"].outputs.rows }}
Colunas: id, nome, email
Mapeamentos: id, nome, email
```

Se a lista contém objetos, o QANode pode inferir colunas comuns, mas em fluxos compartilhados é melhor definir as colunas explicitamente.

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `fileRef` | `fileRef` | Referência do arquivo gerado |
| `name` | `string` | Nome do arquivo |
| `mimeType` | `string` | MIME type do arquivo |
| `sizeBytes` | `number` | Tamanho em bytes |

### Acessando os outputs

```
{{ steps["file-generate"].outputs.fileRef }}
{{ steps["file-generate"].outputs.name }}
{{ steps["file-generate"].outputs.mimeType }}
{{ steps["file-generate"].outputs.sizeBytes }}
```

Use `fileRef` para passar o arquivo para outros nós. Os demais campos servem para logs, validações e mensagens.

---

## Exemplo: Gerar CSV de Massa

```
Tipo de arquivo: CSV
Nome do arquivo: massas.csv
Modo: A partir de lista
Lista: {{ steps["consulta-clientes"].outputs.rows }}
Colunas:
  - id
  - nome
  - email
```

Depois, o arquivo pode ser usado em:

```
{{ steps["file-generate"].outputs.fileRef }}
```

---

## Dicas

- Inclua a extensão no nome do arquivo: `.txt`, `.json`, `.csv`, `.xlsx` ou `.pdf`.
- Para CSV e Excel, prefira **A partir de lista** quando a origem dos dados já é um array.
- Use nomes claros, como `massa-clientes.csv` ou `relatorio-pedido.pdf`.
- Para enviar arquivo para API ou upload web, passe sempre o `fileRef`, não o conteúdo em texto.

