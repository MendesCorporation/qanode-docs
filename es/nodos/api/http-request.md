# Nodo HTTP Request

El nodo **HTTP Request** permite realizar solicitudes HTTP a APIs REST, servicios SOAP o cualquier endpoint web. Admite métodos HTTP comunes, varios tipos de autenticación, credenciales guardadas, envío de archivos y respuestas en archivo.

---

## Visión General

[Visión General](../../assets/images/nos-http-request-visao-geral.mp4)

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `http-request` |
| **Categoría** | API |
| **Color** | 🟣 Morado (#a855f7) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Configuración

### Modo Builder

El modo builder ofrece una interfaz visual para construir la solicitud:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Método** | `string` | `GET`, `QUERY`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS` |
| **URL** | `string` | URL del endpoint (admite `{{ }}`) |
| **Headers** | `object` | Encabezados HTTP como pares clave-valor |
| **Parámetros de query** | `array` | Pares clave-valor agregados a la URL |
| **Tipo del cuerpo** | `string` | `Ninguno`, `JSON`, `Texto bruto`, `x-www-form-urlencoded`, `multipart/form-data` o `Archivo binario` |
| **Tipo de respuesta** | `string` | `Auto`, `Texto`, `JSON` o `Archivo` |
| **Credencial** | `string` | Credencial guardada para autenticación |
| **Timeout (ms)** | `number` o expresión | Tiempo máximo de esta solicitud; valor predeterminado de 30.000 ms |

### Modo Raw

El modo raw permite editar la solicitud como JSON, ofreciendo control total.

### Tiempo límite de la solicitud

El timeout se aplica a **cada llamada HTTP**. El valor predeterminado es **30.000 ms (30 segundos)** y el intervalo configurable va de **1 ms a 86.400.000 ms (24 horas)**. Puede introducir un número fijo o una expresión, por ejemplo `{{ variables.API_TIMEOUT_MS }}`. La expresión debe dar como resultado un número dentro de ese intervalo.

Cuando la llamada supera el límite, la ejecución informa explícitamente del timeout de HTTP Request e indica la duración configurada. En los detalles de la ejecución, la posible causa y la sugerencia se refieren a la solicitud, no a la carga de la página. Si la API suele responder más lentamente, aumente este campo según corresponda y compruebe también la disponibilidad del servidor.

### Importar cURL

**Importar cURL** convierte un comando `curl` en la configuración visual de la solicitud. QANode intenta detectar:

- método HTTP;
- URL;
- headers;
- parámetros de query;
- body JSON o texto;
- `x-www-form-urlencoded`;
- `multipart/form-data`;
- archivo binario enviado con `--data-binary @archivo`.

Cuando el cURL referencia un archivo local, QANode muestra el nombre/camino importado y el usuario debe adjuntar o informar el `fileRef` correspondiente en el campo de archivo.

---

## Autenticación

El nodo admite varios métodos de autenticación.

### Usar Credenciales Guardadas

La forma más segura y recomendada. Selecciona una credencial **HTTP/API** que contenga la URL base y el token:

1. Selecciona la credencial en **Credencial**
2. La URL base y los headers de autenticación se aplican automáticamente
3. En **URL**, informa solo el path, como `/api/users`

### Autenticación Manual

| Tipo | Campos | Resultado |
|------|--------|-----------|
| **Bearer Token** | Token | Header `Authorization: Bearer {token}` |
| **Basic Auth** | Usuario + contraseña | Header `Authorization: Basic {base64}` |
| **API Key** | Nombre del header + token | Header personalizado con el token |

---

## Métodos HTTP

| Método | Uso habitual |
|--------|--------------|
| **GET** | Obtener datos o detalles de un recurso |
| **QUERY** | Consultar datos enviando filtros o criterios estructurados en el cuerpo |
| **POST** | Crear recursos o enviar datos |
| **PUT** | Sustituir un recurso completo |
| **PATCH** | Actualizar parcialmente un recurso |
| **DELETE** | Eliminar un recurso |
| **HEAD** | Obtener únicamente estado y headers, sin cuerpo de respuesta |
| **OPTIONS** | Consultar métodos y opciones admitidos por el endpoint |

### Método QUERY

`QUERY`, definido por la [RFC 10008](https://www.rfc-editor.org/rfc/rfc10008.html), es un método HTTP seguro e idempotente para consultas que necesitan transportar criterios estructurados en el cuerpo. No es lo mismo que los **Parámetros de query** de la URL.

Compare:

```text
GET /orders?status=pending&from=2026-01-01
```

con:

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

Use `QUERY` cuando la documentación de la API indique explícitamente que admite el método, especialmente para filtros grandes, objetos anidados o consultas que no encajan bien en una URL. Para una consulta normal, continúe usando `GET`.

QANode admite en `QUERY` los mismos tipos de cuerpo disponibles para `POST`, `PUT` y `PATCH`:

- JSON;
- texto bruto;
- `x-www-form-urlencoded`;
- `multipart/form-data`;
- archivo binario.

Cuando haya contenido, envíe un `Content-Type` coherente. QANode lo define automáticamente en los modos estructurados, salvo que se haya informado manualmente.

Ejemplo con formulario URL encoded:

```text
Método: QUERY
URL: https://api.ejemplo.com/search
Tipo del cuerpo: x-www-form-urlencoded
Campos:
  term = notebook
  limit = 50
  includeArchived = false
```

> El servidor, proxy, gateway y WAF deben aceptar `QUERY`. Si alguna capa no reconoce el método, la respuesta puede ser `405 Method Not Allowed` o `501 Not Implemented`. Habilite el método en la infraestructura o use el contrato alternativo definido por la API.

---

## Cuerpo

Para métodos que aceptan body (`QUERY`, `POST`, `PUT`, `PATCH`, `DELETE` y `OPTIONS`), elija **Tipo del cuerpo**. QANode no envía body con `GET` o `HEAD`.

### Ninguno

No envía cuerpo. Es la opción utilizada para `GET` y `HEAD`.

### JSON

```json
{
  "name": "{{ variables.USER_NAME }}",
  "email": "{{ steps["web-flow"].outputs.extracts.email }}",
  "active": true
}
```

El editor JSON resalta la sintaxis y acepta expresiones `{{ }}` dentro de los valores.

### Texto bruto

Usa este modo para payloads textuales, XML, SOAP, scripts o cualquier contenido que no deba tratarse como JSON.

```xml
<login>
  <user>{{ variables.USUARIO }}</user>
</login>
```

### x-www-form-urlencoded

Envía campos como `application/x-www-form-urlencoded`.

| Campo | Valor |
|-------|-------|
| `username` | `admin` |
| `password` | `{{ variables.ADMIN_PASS }}` |

### multipart/form-data

Envía campos de formulario en formato multipart, incluyendo texto y archivos.

| Tipo de campo | Descripción |
|---------------|-------------|
| **Texto** | Campo textual común |
| **Archivo** | Campo que recibe un `fileRef` |

Ejemplo:

```
Método: POST
URL: https://httpbin.org/post
Tipo del cuerpo: multipart/form-data
Campo texto:
  description = prueba de upload
Campo archivo:
  file = {{ steps["file-generate"].outputs.fileRef }}
```

### Archivo binario

Envía el contenido de un único archivo como cuerpo de la solicitud. Úsalo cuando la API espera el archivo directamente en el body, no dentro de un formulario multipart.

```
Tipo del cuerpo: Archivo binario
Archivo: {{ steps["file-generate"].outputs.fileRef }}
```

> QANode usa el MIME type del archivo cuando está disponible. Si es necesario, agrega manualmente un header `Content-Type`.

---

## Respuestas en Archivo

El campo **Tipo de respuesta** controla cómo se trata el retorno de la API:

| Tipo | Comportamiento |
|------|----------------|
| **Auto** | Detecta JSON/texto o archivo por contenido y headers |
| **JSON** | Intenta interpretar la respuesta como JSON |
| **Texto** | Devuelve el cuerpo como texto |
| **Archivo** | Guarda la respuesta como artefacto y expone `fileRef` |

Usa **Archivo** cuando la API devuelve PDF, imagen, CSV, Excel, ZIP u otro contenido binario.

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `status` | `number` | Código de estado HTTP (200, 404, 500, etc.) |
| `body` | `any` | Cuerpo de la respuesta como texto o JSON, según la respuesta |
| `fileRef` | `fileRef` | Archivo devuelto por la API, cuando la respuesta se trata como archivo |

### Accediendo a los Outputs

```
// Estado de la respuesta
{{ steps["http-request"].outputs.status }}  →  200

// Cuerpo JSON o texto
{{ steps["http-request"].outputs.body }}  →  { "id": 1, "name": "Juan" }

// Array en el JSON
{{ steps["http-request"].outputs.body.items[0].title }}  →  "Primer item"

// Archivo devuelto por la API
{{ steps["http-request"].outputs.fileRef }}
```

> Campos antiguos como `json`, `headers`, `text`, `rawBody` y `statusCode` pueden seguir accesibles en expresiones manuales durante la ventana de compatibilidad, pero el panel de variables sugiere solo la superficie nueva: `status`, `body` o `fileRef`.

---

## Ejemplos Prácticos

### GET — Obtener usuario

```
Método: GET
URL: https://api.example.com/users/1
Headers: { "Accept": "application/json" }
```

### POST — Crear recurso

```
Método: POST
URL: https://api.example.com/users
Tipo del cuerpo: JSON
JSON: {
  "name": "Maria",
  "email": "maria@example.com"
}
```

### QUERY — Búsqueda estructurada

```text
Método: QUERY
URL: https://api.ejemplo.com/orders/search
Tipo del cuerpo: JSON
JSON: {
  "customerIds": ["1001", "1002"],
  "status": ["pending", "review"],
  "createdAfter": "2026-01-01T00:00:00Z"
}
```

Las expresiones pueden utilizarse normalmente dentro de los criterios:

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
Tipo del cuerpo: multipart/form-data
Campo texto: description = prueba de upload
Campo archivo: file = {{ steps["file-generate"].outputs.fileRef }}
```

### GET — Descargar archivo

```
Método: GET
URL: https://httpbin.org/image/png
Tipo de respuesta: Archivo
```

---

## Manejo de Errores

| Estado | Significado | Acción sugerida |
|--------|-------------|-----------------|
| `200-299` | Éxito | Continuar normalmente |
| `400` | Solicitud inválida | Revisar body/params |
| `401` | No autorizado | Revisar token/credencial |
| `403` | Prohibido | Revisar permisos |
| `404` | No encontrado | Revisar URL |
| `500` | Error del servidor | Revisar API |

Usa un nodo **If** después de HTTP Request para tratar diferentes códigos de estado:

```
[HTTP Request] → [If: status === 200]
                    │ true → [Continuar...]
                    │ false → [Log: "Error: {{ steps.api.outputs.status }}"]
```

---

## Consejos

- Usa credenciales guardadas en lugar de colocar tokens directamente en los campos.
- Verifica `status` antes de usar `body`.
- Usa variables para URLs base: `{{ variables.API_URL }}/endpoint`.
- Usa `multipart/form-data` cuando la API espera upload de formulario.
- Usa **Tipo de respuesta = Archivo** para respuestas binarias.
