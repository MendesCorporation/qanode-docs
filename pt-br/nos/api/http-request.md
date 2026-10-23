# Nó HTTP Request

O nó **HTTP Request** permite fazer requisições HTTP para APIs REST, SOAP ou qualquer endpoint web. Suporta todos os métodos HTTP comuns, múltiplos tipos de autenticação e integração com credenciais salvas.

---

## Visão Geral

[Visão geral](../../assets/images/nos-http-request-visao-geral.mp4)

| Propriedade | Valor |
|-------------|-------|
| **Tipo** | `http-request` |
| **Categoria** | API |
| **Cor** | 🟣 Roxo (#a855f7) |
| **Entrada** | `in` |
| **Saída** | `out` |

---

## Configuração

### Modo Builder

O modo builder oferece uma interface visual para construir a requisição:

| Campo | Tipo | Descrição |
|-------|------|-----------|
| **Método** | `string` | `GET`, `QUERY`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS` |
| **URL** | `string` | URL do endpoint (suporta `{{ }}`) |
| **Headers** | `object` | Cabeçalhos HTTP (chave-valor) |
| **Parâmetros de Query** | `array` | Pares chave-valor adicionados à URL |
| **Tipo do Corpo** | `string` | `Nenhum`, `JSON`, `Texto bruto`, `x-www-form-urlencoded`, `multipart/form-data` ou `Arquivo binário` |
| **Tipo da Resposta** | `string` | `Auto`, `Texto`, `JSON` ou `Arquivo` |
| **Credencial** | `string` | Credencial salva para autenticação |
| **Timeout (ms)** | `number` ou expressão | Tempo máximo desta requisição; padrão de 30.000 ms |

### Modo Raw

O modo raw permite editar a requisição como JSON puro, oferecendo controle total.

### Tempo limite da requisição

O timeout é aplicado a **cada chamada HTTP**. O padrão é **30.000 ms (30 segundos)** e o valor configurável vai de **1 ms a 86.400.000 ms (24 horas)**. Você pode informar um número fixo ou uma expressão, por exemplo `{{ variables.API_TIMEOUT_MS }}`. A expressão precisa resultar em um número dentro desse intervalo.

Quando a chamada ultrapassa o limite, a execução informa explicitamente o timeout do HTTP Request, com a duração configurada. Nos detalhes da execução, a possível causa e a sugestão são específicas da requisição, não de carregamento de página. Se a API costuma responder mais devagar, aumente esse campo para o tempo apropriado e confira também a disponibilidade do servidor.

### Importar cURL

O modo **Importar cURL** converte um comando `curl` para a configuração visual do nó. O QANode tenta identificar automaticamente:

- método HTTP;
- URL;
- headers;
- parâmetros de query;
- body JSON ou texto;
- `x-www-form-urlencoded`;
- `multipart/form-data`;
- arquivo binário enviado por `--data-binary @arquivo`.

Quando o cURL referencia um arquivo local, o QANode mostra o nome/caminho importado e o usuário deve anexar ou informar o `fileRef` correspondente no campo de arquivo.

---

## Autenticação

O nó suporta múltiplos métodos de autenticação:

### Usando Credenciais Salvas

A forma mais segura e recomendada. Selecione uma credencial do tipo **HTTP/API** que contenha a URL base e o token:

1. No campo **Credencial**, selecione a credencial desejada
2. A URL base e os headers de autenticação serão aplicados automaticamente
3. No campo **URL**, informe apenas o path: `/api/users`

### Autenticação Manual

| Tipo | Campos | Resultado |
|------|--------|-----------|
| **Bearer Token** | Token | Header `Authorization: Bearer {token}` |
| **Basic Auth** | Usuário + Senha | Header `Authorization: Basic {base64}` |
| **API Key** | Header Name + Token | Header customizado com o token |

---

## Métodos HTTP

| Método | Uso Típico |
|--------|-----------|
| **GET** | Buscar dados (listar, obter detalhes) |
| **QUERY** | Consultar dados enviando filtros ou critérios estruturados no corpo |
| **POST** | Criar recursos, enviar dados |
| **PUT** | Atualizar recurso completo |
| **PATCH** | Atualizar parcialmente um recurso |
| **DELETE** | Remover um recurso |
| **HEAD** | Obter somente status e headers, sem corpo de resposta |
| **OPTIONS** | Consultar métodos e opções aceitos pelo endpoint |

### Método QUERY

`QUERY`, definido pela [RFC 10008](https://www.rfc-editor.org/rfc/rfc10008.html), é um método HTTP seguro e idempotente para consultas que precisam carregar critérios estruturados no corpo da requisição. Ele não é a mesma coisa que os **Parâmetros de Query** da URL.

Compare:

```text
GET /orders?status=pending&from=2026-01-01
```

com:

```text
QUERY /orders
Content-Type: application/json

{
  "status": ["pending", "review"],
  "period": {
    "from": "2026-01-01",
    "to": "2026-01-31"
  },
  "sort": ["createdAt:desc"]
}
```

Use `QUERY` quando a documentação da API declarar suporte ao método, especialmente para filtros grandes, objetos aninhados ou consultas que não cabem bem na URL. Para uma busca comum, continue usando `GET`.

O QANode permite usar em `QUERY` os mesmos tipos de corpo disponíveis em `POST`, `PUT` e `PATCH`:

- JSON;
- texto bruto;
- `x-www-form-urlencoded`;
- `multipart/form-data`;
- arquivo binário.

Quando houver conteúdo, envie um `Content-Type` coerente. O QANode define esse header automaticamente nos modos estruturados, salvo quando ele tiver sido informado manualmente.

Exemplo com formulário URL encoded:

```text
Método: QUERY
URL: https://api.exemplo.com/search
Tipo do Corpo: x-www-form-urlencoded
Campos:
  term = notebook
  limit = 50
  includeArchived = false
```

> O servidor, proxy, gateway e WAF precisam aceitar `QUERY`. Se algum deles não reconhecer o método, a resposta pode ser `405 Method Not Allowed` ou `501 Not Implemented`. Isso não pode ser corrigido apenas no QANode; habilite o método na infraestrutura ou use o contrato alternativo definido pela API.

---

## Headers

Adicione headers customizados como pares chave-valor:

| Header | Exemplo |
|--------|---------|
| `Content-Type` | `application/json` |
| `Accept` | `application/json` |
| `X-Custom-Header` | `meu-valor` |
| `Authorization` | `Bearer {{ variables.TOKEN }}` |

> Headers suportam expressões `{{ }}` para valores dinâmicos.

---

## Body (Corpo da Requisição)

Para métodos que aceitam body (`QUERY`, `POST`, `PUT`, `PATCH`, `DELETE` e `OPTIONS`), escolha o **Tipo do Corpo**. O QANode não envia body em `GET` ou `HEAD`.

### Nenhum

Não envia corpo. É a opção usada em `GET` e `HEAD`.

### JSON

```json
{
  "name": "{{ variables.USER_NAME }}",
  "email": "{{ steps["web-flow"].outputs.extracts.email }}",
  "active": true
}
```

O editor de JSON mostra realce de sintaxe e continua aceitando expressões `{{ }}` dentro de valores.

### Texto bruto

Use para enviar payloads textuais, XML, SOAP, scripts ou qualquer conteúdo que não deve ser tratado como JSON.

```xml
<login>
  <user>{{ variables.USUARIO }}</user>
</login>
```

### x-www-form-urlencoded

Envia campos no formato `application/x-www-form-urlencoded`.

| Campo | Valor |
|-------|-------|
| `username` | `admin` |
| `password` | `{{ variables.ADMIN_PASS }}` |

### multipart/form-data

Envia campos de formulário no formato multipart, incluindo texto e arquivos.

| Tipo do campo | Descrição |
|---------------|-----------|
| **Texto** | Campo textual normal |
| **Arquivo** | Campo que recebe um `fileRef` |

Exemplo:

```
Método: POST
URL: https://httpbin.org/post
Tipo do Corpo: multipart/form-data
Campo texto:
  description = teste de upload
Campo arquivo:
  file = {{ steps["file-generate"].outputs.fileRef }}
```

### Arquivo binário

Envia o conteúdo de um único arquivo como corpo da requisição. Use quando a API espera o arquivo diretamente no body, e não dentro de um formulário multipart.

```
Tipo do Corpo: Arquivo binário
Arquivo: {{ steps["file-generate"].outputs.fileRef }}
```

> O QANode usa o MIME type do arquivo quando disponível. Se necessário, adicione manualmente um header `Content-Type`.

---

## Respostas em Arquivo

O campo **Tipo da Resposta** controla como o retorno da API será tratado:

| Tipo | Comportamento |
|------|---------------|
| **Auto** | Detecta JSON/texto ou arquivo pelo conteúdo e headers |
| **JSON** | Tenta interpretar a resposta como JSON |
| **Texto** | Retorna o corpo como texto |
| **Arquivo** | Salva a resposta como artefato e expõe `fileRef` |

Use **Arquivo** quando a API retorna PDF, imagem, CSV, Excel, ZIP ou outro conteúdo binário.

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `status` | `number` | Código de status HTTP (200, 404, 500, etc.) |
| `body` | `any` | Corpo da resposta como texto ou JSON, conforme o retorno |
| `fileRef` | `fileRef` | Arquivo retornado pela API, quando a resposta for tratada como arquivo |

### Acessando os Outputs

```
// Status da resposta
{{ steps["http-request"].outputs.status }}  →  200

// Corpo JSON ou texto
{{ steps["http-request"].outputs.body }}  →  { "id": 1, "name": "João" }

// Array no JSON
{{ steps["http-request"].outputs.body.items[0].title }}  →  "Primeiro Item"

// Arquivo retornado pela API
{{ steps["http-request"].outputs.fileRef }}
```

> Campos antigos como `json`, `headers`, `text`, `rawBody` e `statusCode` podem continuar acessíveis em expressões manuais durante a janela de compatibilidade, mas o painel de variáveis sugere apenas a superfície nova: `status`, `body` ou `fileRef`.

---

## Exemplos Práticos

### GET — Buscar usuário

```
Método: GET
URL: https://api.exemplo.com/users/1
Headers: { "Accept": "application/json" }
```

### POST — Criar recurso

```
Método: POST
URL: https://api.exemplo.com/users
Tipo do Corpo: JSON
JSON: {
  "name": "Maria",
  "email": "maria@exemplo.com"
}
```

### QUERY — Busca estruturada

```text
Método: QUERY
URL: https://api.exemplo.com/orders/search
Tipo do Corpo: JSON
JSON: {
  "customerIds": ["1001", "1002"],
  "status": ["pending", "review"],
  "createdAfter": "2026-01-01T00:00:00Z"
}
```

Use expressões normalmente dentro dos critérios:

```json
{
  "customerId": "{{ variables.CUSTOMER_ID }}",
  "cursor": "{{ steps[\"previous-page\"].outputs.body.nextCursor }}"
}
```

### POST — Upload multipart

```
Método: POST
URL: https://httpbin.org/post
Tipo do Corpo: multipart/form-data
Campo texto: description = teste de upload
Campo arquivo: file = {{ steps["file-generate"].outputs.fileRef }}
```

### GET — Baixar arquivo

```
Método: GET
URL: https://httpbin.org/image/png
Tipo da Resposta: Arquivo
```

O output `fileRef` pode ser usado em nós de arquivo, componentes, SSH, Mobile, Smart Web ou Custom JavaScript.

### PUT com autenticação Bearer

```
Método: PUT
URL: https://api.exemplo.com/users/1
Auth: Bearer Token → {{ steps.login.outputs.body.token }}
Tipo do Corpo: JSON
JSON: {
  "name": "Maria Silva",
  "role": "admin"
}
```

### Encadeando requisições

```
[HTTP Request: POST /login]
    │ outputs.body.token = "abc123"
    ▼
[HTTP Request: GET /api/profile]
    │ Header: Authorization = Bearer {{ steps.login.outputs.body.token }}
    ▼
[If: {{ steps.profile.outputs.status }} === 200]
    │ true → [Log: "Perfil: {{ steps.profile.outputs.body.name }}"]
```

---

## Tratamento de Erros

| Status | Significado | Ação Sugerida |
|--------|-------------|--------------|
| `200-299` | Sucesso | Prosseguir normalmente |
| `400` | Requisição inválida | Verificar body/params |
| `401` | Não autorizado | Verificar token/credencial |
| `403` | Proibido | Verificar permissões |
| `404` | Não encontrado | Verificar URL |
| `500` | Erro do servidor | Verificar API |

Use o nó **If** após o HTTP Request para tratar diferentes status:

```
[HTTP Request] → [If: status === 200]
                    │ true → [Continuar...]
                    │ false → [Log: "Erro: {{ steps.api.outputs.status }}"]
```

---

## Dicas

- **Use credenciais salvas** em vez de colocar tokens diretamente nos campos — é mais seguro e facilita a manutenção
- **Verifique o status** da resposta antes de usar o body — uma resposta de erro pode não ter o formato esperado
- **Use variáveis** para URLs base: `{{ variables.API_URL }}/endpoint` permite trocar ambientes facilmente
- Use **Tipo da Resposta = Arquivo** quando a API retornar conteúdo binário
- Para uploads, prefira passar `fileRef` de nós anteriores em vez de Base64 em texto
