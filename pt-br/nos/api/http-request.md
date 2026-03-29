# Nó HTTP Request

O nó **HTTP Request** permite fazer requisições HTTP para APIs REST, SOAP ou qualquer endpoint web. Suporta todos os métodos HTTP comuns, múltiplos tipos de autenticação e integração com credenciais salvas.

---

## Visão Geral

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
| **Body** | `any` | Corpo da requisição (JSON ou texto) |
| **Credencial** | `string` | Credencial salva para autenticação |

### Modo Raw

O modo raw permite editar a requisição como JSON puro, oferecendo controle total.

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

Para métodos que aceitam body (POST, PUT, PATCH), você pode enviar:

### JSON

```json
{
  "name": "{{ variables.USER_NAME }}",
  "email": "{{ steps["web-flow"].outputs.extracts.email }}",
  "active": true
}
```

### Form Data

```json
{
  "username": "admin",
  "password": "{{ variables.ADMIN_PASS }}"
}
```

> O body suporta expressões `{{ }}` em qualquer valor.

---

## Outputs

| Output | Tipo | Descrição |
|--------|------|-----------|
| `status` | `number` | Código de status HTTP (200, 404, 500, etc.) |
| `body` | `string` | Corpo da resposta como texto |
| `json` | `any` | Corpo da resposta parseado como JSON |

### Acessando os Outputs

```
// Status da resposta
{{ steps["http-request"].outputs.status }}  →  200

// Corpo JSON
{{ steps["http-request"].outputs.json }}  →  { "id": 1, "name": "João" }

// Propriedade específica do JSON
{{ steps["http-request"].outputs.json.name }}  →  "João"

// Array no JSON
{{ steps["http-request"].outputs.json.items[0].title }}  →  "Primeiro Item"

// Corpo como texto
{{ steps["http-request"].outputs.body }}  →  '{"id": 1, "name": "João"}'
```

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
Headers: { "Content-Type": "application/json" }
Body: {
  "name": "Maria",
  "email": "maria@exemplo.com"
}
```

### PUT com autenticação Bearer

```
Método: PUT
URL: https://api.exemplo.com/users/1
Auth: Bearer Token → {{ steps.login.outputs.json.token }}
Body: {
  "name": "Maria Silva",
  "role": "admin"
}
```

### Encadeando requisições

```
[HTTP Request: POST /login]
    │ outputs.json.token = "abc123"
    ▼
[HTTP Request: GET /api/profile]
    │ Header: Authorization = Bearer {{ steps.login.outputs.json.token }}
    ▼
[If: {{ steps.profile.outputs.status }} === 200]
    │ true → [Log: "Perfil: {{ steps.profile.outputs.json.name }}"]
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
- **Verifique o status** da resposta antes de usar o JSON — uma resposta de erro pode não ter o formato esperado
- **Use variáveis** para URLs base: `{{ variables.API_URL }}/endpoint` permite trocar ambientes facilmente
- O output `json` retorna `null` se a resposta não for JSON válido — nesse caso, use `body`
