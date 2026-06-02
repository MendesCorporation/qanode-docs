# Nó Extrair Arquivo

O nó **Extrair Arquivo** lê um arquivo recebido como `fileRef` e devolve texto ou dados estruturados conforme o tipo escolhido.

---

## Visão Geral

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `file-extract` |
| **Categoria** | Arquivos |
| **Cor** | 🟤 Dourado (#C89F65) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Quando Usar

Use este nó quando o fluxo precisa:

- ler TXT ou Markdown como texto;
- validar ou reaproveitar conteúdo JSON;
- transformar CSV em linhas estruturadas;
- ler planilhas Excel;
- extrair texto de PDF;
- usar OCR como fallback quando o PDF é imagem.

---

## Configuração

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Arquivo** | `fileRef` | Arquivo de entrada |
| **Tipo de extração** | seleção | TXT, JSON, CSV, Excel ou PDF |

Os campos seguintes mudam conforme o tipo de extração.

### TXT

| Campo | Descrição |
|-------|-----------|
| **Codificação** | Codificação usada para ler o texto, como `utf8` |

### JSON

| Campo | Descrição |
|-------|-----------|
| **Codificação** | Codificação usada para ler o arquivo |

O conteúdo precisa ser JSON válido.

### CSV

| Campo | Descrição |
|-------|-----------|
| **Delimitador** | Separador das colunas, como `,` ou `;` |
| **Tem cabeçalho** | Usa a primeira linha como nomes de colunas |
| **Codificação** | Codificação do arquivo |

### Excel

| Campo | Descrição |
|-------|-----------|
| **Nome da aba** | Aba a ler. Se vazio, usa a primeira aba |
| **Linha do cabeçalho** | Linha usada como cabeçalho |

### PDF

| Campo | Descrição |
|-------|-----------|
| **Intervalo de páginas** | Páginas a extrair, como `1-3,5` |
| **OCR** | Ativa OCR como fallback para PDFs em imagem |
| **Idiomas OCR** | Idiomas do OCR, como `por+eng+spa` |
| **Escala OCR** | Escala usada na renderização antes do OCR |

---

## OCR em PDF

Quando o PDF possui texto real, o QANode tenta extrair diretamente. Se o PDF parece ser imagem, o OCR pode ser usado automaticamente como fallback.

Use OCR para:

- PDFs digitalizados;
- demonstrativos e comprovantes em imagem;
- documentos escaneados;
- arquivos sem camada de texto.

Idiomas comuns:

| Valor | Idiomas |
|-------|---------|
| `por` | Português |
| `eng` | Inglês |
| `spa` | Espanhol |
| `por+eng+spa` | Português, inglês e espanhol |

---

## Outputs

Os outputs dependem do tipo escolhido.

### TXT

| Output | Tipo | Descrição |
|--------|------|-----------|
| `text` | `string` | Texto extraído |

### JSON

| Output | Tipo | Descrição |
|--------|------|-----------|
| `json` | `any` | JSON parseado |
| `text` | `string` | Texto original |

### CSV e Excel

| Output | Tipo | Descrição |
|--------|------|-----------|
| `rows` | `array` | Linhas extraídas |
| `columns` | `array` | Colunas detectadas |
| `rowCount` | `number` | Quantidade de linhas |
| `sheets` | `array` | Abas encontradas, no caso de Excel |

### PDF

| Output | Tipo | Descrição |
|--------|------|-----------|
| `text` | `string` | Texto extraído |
| `pages` | `array` | Texto por página |
| `pageCount` | `number` | Quantidade de páginas processadas |

---

## Exemplos

### Extrair CSV

```
Arquivo: {{ steps["file-generate"].outputs.fileRef }}
Tipo de extração: CSV
Delimitador: ,
Tem cabeçalho: true
```

Uso posterior:

```
{{ steps["file-extract"].outputs.rows[0].email }}
{{ steps["file-extract"].outputs.rowCount }}
```

### Extrair texto de PDF

```
Arquivo: {{ steps["http-request"].outputs.fileRef }}
Tipo de extração: PDF
Intervalo de páginas: 1-2
OCR: true
Idiomas OCR: por+eng+spa
```

---

## Validação de Tipo

O nó valida se o arquivo é compatível com o tipo de extração. Se o arquivo não for suportado, o campo fica em vermelho no painel e a execução falha com mensagem clara, por exemplo:

```
Arquivo não suportado: planilha.xlsx
```

---

## Dicas

- Escolha o tipo de extração de acordo com o conteúdo real do arquivo.
- Para PDF escaneado, habilite OCR; para PDF com texto real, deixe o OCR como fallback.
- Para CSV com `;`, altere o delimitador antes de executar.
- Use `rows` para alimentar loops, componentes, geração de arquivos ou validações.

