# Nodo HTTP Request

El nodo **HTTP Request** permite realizar solicitudes HTTP a APIs REST, servicios SOAP o cualquier endpoint web. Admite todos los métodos HTTP comunes, múltiples tipos de autenticación e integración con credenciales guardadas.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `http-request` |
| **Categoría** | API |
| **Color** | Morado (#a855f7) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Configuración

### Modo Builder

El modo builder ofrece una interfaz visual para construir la solicitud:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Método** | `string` | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| **URL** | `string` | URL del endpoint (admite `{{ }}`) |
| **Headers** | `object` | Encabezados HTTP (clave-valor) |
| **Body** | `any` | Cuerpo de la solicitud (JSON o texto plano) |
| **Credencial** | `string` | Credencial guardada para autenticación |

### Modo Raw

El modo raw permite editar la solicitud como JSON puro, proporcionando control total.

---

## Autenticación

El nodo admite múltiples métodos de autenticación:

### Usar Credenciales Guardadas

La forma más segura y recomendada. Seleccione una credencial de tipo **HTTP/API** que contenga la URL base y el token:

1. En el campo **Credencial**, seleccione la credencial deseada
2. La URL base y los encabezados de autenticación se aplicarán automáticamente
3. En el campo **URL**, ingrese solo el path: `/api/users`

### Autenticación Manual

| Tipo | Campos | Resultado |
|------|--------|-----------|
| **Bearer Token** | Token | Header `Authorization: Bearer {token}` |
| **Basic Auth** | Usuario + Contraseña | Header `Authorization: Basic {base64}` |
| **API Key** | Nombre del Header + Token | Header personalizado con el token |

---

## Métodos HTTP

| Método | Uso Típico |
|--------|-----------|
| **GET** | Obtener datos (listar, ver detalles) |
| **POST** | Crear recursos, enviar datos |
| **PUT** | Reemplazar un recurso completo |
| **PATCH** | Actualizar parcialmente un recurso |
| **DELETE** | Eliminar un recurso |

---

## Headers

Agregue encabezados personalizados como pares clave-valor:

| Header | Ejemplo |
|--------|---------|
| `Content-Type` | `application/json` |
| `Accept` | `application/json` |
| `X-Custom-Header` | `mi-valor` |
| `Authorization` | `Bearer {{ variables.TOKEN }}` |

> Los headers admiten expresiones `{{ }}` para valores dinámicos.

---

## Body (Cuerpo de la Solicitud)

Para métodos que aceptan body (POST, PUT, PATCH), puede enviar:

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

> El body admite expresiones `{{ }}` en cualquier valor.

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `status` | `number` | Código de estado HTTP (200, 404, 500, etc.) |
| `body` | `string` | Cuerpo de la respuesta como texto plano |
| `json` | `any` | Cuerpo de la respuesta analizado como JSON |

### Acceder a los Outputs

```
// Estado de la respuesta
{{ steps["http-request"].outputs.status }}  →  200

// Cuerpo JSON
{{ steps["http-request"].outputs.json }}  →  { "id": 1, "name": "João" }

// Propiedad específica del JSON
{{ steps["http-request"].outputs.json.name }}  →  "João"

// Array en el JSON
{{ steps["http-request"].outputs.json.items[0].title }}  →  "Primeiro Item"

// Cuerpo como texto plano
{{ steps["http-request"].outputs.body }}  →  '{"id": 1, "name": "João"}'
```

---

## Ejemplos Prácticos

### GET — Obtener un usuario

```
Método: GET
URL: https://api.exemplo.com/users/1
Headers: { "Accept": "application/json" }
```

### POST — Crear un recurso

```
Método: POST
URL: https://api.exemplo.com/users
Headers: { "Content-Type": "application/json" }
Body: {
  "name": "Maria",
  "email": "maria@exemplo.com"
}
```

### PUT con autenticación Bearer

```
Método: PUT
URL: https://api.exemplo.com/users/1
Auth: Bearer Token → {{ steps.login.outputs.json.token }}
Body: {
  "name": "Maria Silva",
  "role": "admin"
}
```

### Encadenando solicitudes

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

## Manejo de Errores

| Estado | Significado | Acción Sugerida |
|--------|-------------|----------------|
| `200-299` | Éxito | Continuar normalmente |
| `400` | Solicitud inválida | Verificar body/params |
| `401` | No autorizado | Verificar token/credencial |
| `403` | Prohibido | Verificar permisos |
| `404` | No encontrado | Verificar URL |
| `500` | Error del servidor | Verificar API |

Use el nodo **If** después del HTTP Request para manejar diferentes códigos de estado:

```
[HTTP Request] → [If: status === 200]
                    │ true → [Continuar...]
                    │ false → [Log: "Error: {{ steps.api.outputs.status }}"]
```

---

## Consejos

- **Use credenciales guardadas** en lugar de colocar tokens directamente en los campos — es más seguro y facilita el mantenimiento
- **Verifique el estado** antes de usar el JSON — una respuesta de error puede no tener el formato esperado
- **Use variables** para las URLs base: `{{ variables.API_URL }}/endpoint` permite cambiar de entorno fácilmente
- El output `json` devuelve `null` si la respuesta no es JSON válido — en ese caso, use `body`
