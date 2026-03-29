# Nodo Email Inbox

El nodo **Email Inbox** se conecta a un buzón de correo vía IMAP y espera o busca mensajes que cumplan los criterios definidos. Ideal para validar la recepción de correos, extraer OTPs y capturar enlaces de confirmación en flujos de prueba.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `email-inbox` |
| **Categoría** | Utilidades |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Prerequisito: Credencial de Correo

Antes de usar este nodo, crea una **Credencial de Correo** en la sección [Credenciales](../../credenciales/credenciales.md#email). Almacena la conexión IMAP y la autenticación de forma segura.

---

## Campos de Configuración

### General

| Campo | Tipo | Predeterminado | Descripción |
|-------|------|----------------|-------------|
| **Credencial** | selección | — | Credencial de correo a usar |
| **Operación** | selección | `Extract Email` | Qué hacer al encontrar el correo |
| **Carpeta** | texto | `INBOX` | Carpeta IMAP a monitorear |
| **Solo No Leídos** | boolean | `true` | Busca solo mensajes no leídos |
| **Marcar Como Leído** | boolean | `false` | Marca los mensajes encontrados como leídos |

### Operaciones Disponibles

| Valor | Descripción |
|-------|-------------|
| `Extract Email` | Espera y retorna el correo completo |
| `Extract OTP` | Extrae un código numérico del cuerpo del correo |
| `Extract Link` | Extrae una URL del cuerpo del correo |

### Filtros de Búsqueda

Todos los filtros son opcionales. Cuando se completan, el valor debe estar **contenido** en el correo para que sea considerado.

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| **From Contains** | Filtra por remitente | `noreply@gmail.com` |
| **To Contains** | Filtra por destinatario | `micorreo@` |
| **Subject Contains** | Filtra por asunto | `Código de verificación` |
| **Body Contains** | Filtra por texto en el cuerpo | `tu código es` |

### Filtro de Fecha

| Campo | Opciones | Descripción |
|-------|----------|-------------|
| **Received After Mode** | `Step Start` / `Absolute Date/Time` / `No Date Filter` | Modo del filtro de fecha |
| **Received After** | ISO 8601 | Se usa solo cuando el modo = `Absolute Date/Time` |

> **Step Start (recomendado):** el nodo usa el UID del siguiente correo en el momento en que comienza la ejecución, ignorando mensajes anteriores de forma confiable, independientemente de la zona horaria.

### Polling y Límites

| Campo | Tipo | Predeterminado | Descripción |
|-------|------|----------------|-------------|
| **Timeout (ms)** | number | `120000` | Tiempo máximo de espera (0 = sin espera, verifica una vez) |
| **Poll Interval (ms)** | number | `3000` | Intervalo entre verificaciones |
| **Max Scan** | number | `30` | Máximo de correos a analizar por ciclo |
| **Max Matches** | number | `1` | Máximo de correos a retornar |

### Extracción de OTP (cuando operación = `Extract OTP`)

| Campo | Tipo | Predeterminado | Descripción |
|-------|------|----------------|-------------|
| **OTP Regex (opcional)** | regex | automático | Expresión personalizada para capturar el código |
| **OTP Min Length** | number | `6` | Longitud mínima del código |
| **OTP Max Length** | number | `6` | Longitud máxima del código |

Cuando no se ingresa ningún regex, el nodo busca automáticamente secuencias numéricas con la longitud configurada.

### Extracción de Enlace (cuando operación = `Extract Link`)

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Link Domain Contains** | texto | Filtra enlaces por dominio (ej: `accounts.google.com`) |
| **Link Regex (opcional)** | regex | Expresión personalizada para capturar la URL |

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `found` | `boolean` | Si se encontró algún correo |
| `matchCount` | `number` | Cantidad de correos que coincidieron |
| `attempts` | `number` | Número de verificaciones realizadas |
| `email` | `object` | Primer correo encontrado (objeto completo) |
| `emails` | `array` | Todos los correos encontrados (hasta `maxMatches`) |
| `links` | `array` | URLs encontradas en el primer correo |
| `otp` | `string` | Código extraído (solo en `extractOtp`) |
| `link` | `string` | URL extraída (solo en `extractLink`) |

### Estructura del objeto `email`

```json
{
  "uid": 12345,
  "messageId": "<abc@gmail.com>",
  "subject": "Tu código de verificación",
  "from": "Empresa <noreply@empresa.com>",
  "to": "usuario@gmail.com",
  "date": "2026-03-08T12:00:00.000Z",
  "text": "Tu código es 482910. Válido por 10 minutos.",
  "html": "<p>Tu código es <strong>482910</strong>...</p>",
  "links": ["https://empresa.com/confirmar?token=xyz"]
}
```

---

## Ejemplos Prácticos

### Esperar correo de confirmación de registro

```
[Clic en "Crear cuenta" en el navegador]
    │
    ▼
[Email Inbox]
  Operación: Extract Email
  Subject Contains: "Confirma tu correo"
  From Contains: noreply@servicio.com
  Timeout: 60000ms
    │
    ▼
[HTTP Request: GET {{ steps.emailInbox.outputs.links[0] }}]
```

### Extraer OTP de 6 dígitos

```
[Clic en "Enviar código" en la app]
    │
    ▼
[Email Inbox]
  Operación: Extract OTP
  Subject Contains: "código de verificación"
  OTP Min Length: 6
  OTP Max Length: 6
  Timeout: 30000ms
    │
    ▼
[Escribir en campo OTP: {{ steps.emailInbox.outputs.otp }}]
```

### Extraer enlace de restablecimiento de contraseña

```
[Clic en "Olvidé mi contraseña"]
    │
    ▼
[Email Inbox]
  Operación: Extract Link
  Subject Contains: "restablecer contraseña"
  Link Domain Contains: accounts.empresa.com
  Timeout: 30000ms
    │
    ▼
[Navigate: {{ steps.emailInbox.outputs.link }}]
```

---

## Configuración por Proveedor

### Gmail

| Modo | Configuración necesaria |
|------|------------------------|
| **Contraseña / App Password** | Activa "Acceso a aplicaciones menos seguras" o genera una [App Password](https://myaccount.google.com/apppasswords) |
| **OAuth2** | Crea una app en Google Cloud Console con la URI de callback de la credencial |

> Para Gmail con OAuth2, haz clic en **Connect OAuth (Browser)** en la pantalla de credencial. Google requiere que la URI de callback registrada en la Consola sea idéntica a la que se muestra en el campo **OAuth Callback URI**.

### Outlook / Microsoft 365

| Modo | Configuración necesaria |
|------|------------------------|
| **Contraseña / App Password** | Funciona con cuentas sin MFA o con contraseñas de aplicación |
| **OAuth2** | Registra la app en Azure Active Directory con permiso `IMAP.AccessAsUser.All` |

### IMAP Personalizado

Configura manualmente host, puerto y modo de seguridad (SSL/TLS).

---

## Consejos

- **Siempre usa Received After Mode = Step Start** — evita capturar correos anteriores a la prueba
- **Combina filtros** — cuanto más específico (`from` + `subject`), más confiable la detección
- **Marcar como leído** — útil para evitar que el mismo correo sea capturado en ejecuciones sucesivas
- **Timeout 0** — ejecuta solo una verificación sin esperar; útil para aserciones sobre correos ya recibidos
- **OTP Regex personalizado** — úsalo cuando el código tenga formato no numérico o esté en un contexto ambiguo (ej: `código: ([A-Z0-9]{8})`)
