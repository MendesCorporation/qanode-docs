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
| **Método** | `string` | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| **URL** | `string` | URL do endpoint (suporta `{{ }}`) |
| **Headers** | `object` | Cabeçalhos HTTP (chave-valor) |
| **Parâmetros de Query** | `array` | Pares chave-valor adicionados à URL |
| **Tipo do Corpo** | `string` | `Nenhum`, `JSON`, `Texto bruto`, `x-www-form-urlencoded`, `multipart/form-data` ou `Arquivo binário` |
| **Tipo da Resposta** | `string` | `Auto`, `Texto`, `JSON` ou `Arquivo` |
| **Credencial** | `string` | Credencial salva para autenticação |

### Modo Raw

O modo raw permite editar a requisição como JSON puro, oferecendo controle total.

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
| **POST** | Criar recursos, enviar dados |
| **PUT** | Atualizar recurso completo |
| **PATCH** | Atualizar parcialmente um recurso |
| **DELETE** | Remover um recurso |

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

Para métodos que aceitam body (POST, PUT, PATCH), escolha o **Tipo do Corpo**.

### Nenhum

Não envia corpo. É o padrão para `GET` e `DELETE`.

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
