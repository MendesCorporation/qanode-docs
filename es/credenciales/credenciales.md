# Credenciales

Las **Credenciales** almacenan información de conexión de forma segura y centralizada. En lugar de colocar contraseñas y tokens directamente en los nodos, usted crea una credencial y la referencia en los flujos.

---

## Gestión de Credenciales

1. En el menú lateral, haga clic en **Credenciales**
2. La lista muestra todas las credenciales registradas

[Lista de credenciales](../../assets/images/lista-credencial.mp4)
*Imagen: Pantalla de credenciales con una lista que muestra nombre, tipo y fecha de creación*

---

## Crear una Credencial

1. Haga clic en **+ Nueva Credencial**
2. Seleccione el **tipo** de credencial
3. Complete los campos específicos del tipo
4. (Opcional) Haga clic en **Probar Conexión** para verificar
5. Haga clic en **Guardar**

---

## Tipos de Credenciales

### HTTP / API

Para conexiones con APIs REST.

| Campo | Descripción |
|-------|-------------|
| **Nombre** | Nombre de la credencial |
| **URL Base** | URL base de la API (ej: `https://api.exemplo.com`) |
| **Tipo de Auth** | `None`, `Bearer Token`, `Basic Auth` |
| **Token** | Token Bearer (cuando Bearer) |
| **Usuario** | Nombre de usuario (cuando Basic) |
| **Contraseña** | Contraseña (cuando Basic) |
| **Headers** | Encabezados personalizados (clave-valor) |

**Uso en el nodo HTTP Request:** Seleccione la credencial e ingrese solo el path:
```
Credencial: "API Exemplo"
URL: /api/users  (la URL base se agrega automáticamente)
```

### Override por Pipeline — Enterprise

En la integración CI/CD de QANode Enterprise, campos específicos de una credencial pueden sobrescribirse solo para la ejecución actual.

Ejemplo:

```bash
npx @qanode/cli run scenario \
  --scenario-id SCENARIO_ID \
  --credential api-main.token=$API_TOKEN \
  --wait
```

Este patrón es útil para:

- tokens efímeros de pipeline
- secretos temporales de homologación
- endpoints de entorno transitorio

El valor guardado en la credencial permanece intacto al finalizar la ejecución.

> Para más detalles, vea [Overrides por Ejecución](../ci-cd/overrides.md).

---

### PostgreSQL

| Campo | Descripción |
|-------|-------------|
| **Nombre** | Nombre de la credencial |
| **Host** | Dirección del servidor |
| **Puerto** | Puerto (predeterminado: 5432) |
| **Base de Datos** | Nombre de la base de datos |
| **Usuario** | Usuario de conexión |
| **Contraseña** | Contraseña de conexión |
| **SSL** | Usar conexión SSL |

---

### MySQL

| Campo | Descripción |
|-------|-------------|
| **Nombre** | Nombre de la credencial |
| **Host** | Dirección del servidor |
| **Puerto** | Puerto (predeterminado: 3306) |
| **Base de Datos** | Nombre de la base de datos |
| **Usuario** | Usuario de conexión |
| **Contraseña** | Contraseña de conexión |
| **SSL** | Usar conexión SSL |

---

### MariaDB

Mismos campos que MySQL.

---

### Oracle

| Campo | Descripción |
|-------|-------------|
| **Nombre** | Nombre de la credencial |
| **Host** | Dirección del servidor |
| **Puerto** | Puerto (predeterminado: 1521) |
| **Service Name** | Nombre del servicio Oracle |
| **Usuario** | Usuario de conexión |
| **Contraseña** | Contraseña de conexión |

---

### MongoDB

| Campo | Descripción |
|-------|-------------|
| **Nombre** | Nombre de la credencial |
| **URI** | Cadena de conexión de MongoDB |
| **Base de Datos** | Nombre de la base de datos (cuando no está incluido en la URI) |

**URIs de ejemplo:**
```
mongodb://usuario:senha@host:27017/banco?authSource=admin
mongodb+srv://usuario:senha@cluster.mongodb.net/banco
```

---

### SSH

| Campo | Descripción |
|-------|-------------|
| **Nombre** | Nombre de la credencial |
| **Host** | Dirección del servidor |
| **Puerto** | Puerto SSH (predeterminado: 22) |
| **Usuario** | Nombre de usuario |
| **Contraseña** | Contraseña (autenticación por contraseña) |
| **Clave Privada** | Contenido de la clave privada (autenticación por clave) |
| **Passphrase** | Frase de contraseña de la clave (si está protegida) |

---

### Correo Electrónico

Credencial para conectarse a buzones de correo vía IMAP. Usada por el nodo [Email Inbox](../nodos/utilidades/email-inbox.md).

| Campo | Descripción |
|-------|-------------|
| **Nombre** | Nombre de la credencial |
| **Proveedor** | `Gmail`, `Outlook` o `Custom IMAP` |
| **Auth Mode** | `Password / App Password` u `OAuth2` |
| **Correo** | Dirección de correo de la cuenta |
| **IMAP Host** | Servidor IMAP (completado automáticamente para Gmail y Outlook) |
| **IMAP Port** | Puerto IMAP (predeterminado: 993) |
| **Secure** | Usar TLS/SSL (predeterminado: true) |

**Modo Contraseña:**

| Campo | Descripción |
|-------|-------------|
| **Password / App Password** | Contraseña de la cuenta o App Password generada por el proveedor |

> Para Gmail con contraseña, es obligatorio usar una **App Password** — no la contraseña normal de la cuenta. [Generar App Password](https://myaccount.google.com/apppasswords)

**Modo OAuth2:**

| Campo | Descripción |
|-------|-------------|
| **Client ID** | ID de cliente OAuth registrado en el proveedor |
| **Client Secret** | Secreto de cliente OAuth |
| **Tenant ID** | Solo para Outlook — ID del tenant de Azure (predeterminado: `common`) |
| **Scopes** | Ámbitos OAuth (opcional; valores predeterminados completados automáticamente) |
| **OAuth Callback URI** | URI que debe registrarse en la consola del proveedor (solo lectura) |

Después de completar Client ID y Secret, haz clic en **Connect OAuth (Browser)** para autorizar. Se abrirá una ventana para iniciar sesión en el proveedor y, al completarse, el Access Token y Refresh Token se completan automáticamente.

> **Gmail:** registra la **OAuth Callback URI** mostrada en el campo como "URI de redireccionamiento autorizado" en la [Google Cloud Console](https://console.cloud.google.com/) → Credenciales → OAuth 2.0.

> **Outlook:** registra la URI como "URI de redireccionamiento" en el [Portal de Azure](https://portal.azure.com/) → Registros de aplicaciones → Autenticación.

---

## Probar la Conexión

Antes de usar una credencial, pruebe la conexión:

1. En la pantalla de edición de la credencial, haga clic en **Probar Conexión**
2. QANode intentará conectarse usando los datos proporcionados
3. El resultado se mostrará: éxito o error con un mensaje detallado

[Prueba de conexión](../../assets/images/teste-credencial.mp4)
*Imagen: Credencial con botón de prueba de conexión y mensaje de éxito*

---

## Usar Credenciales en los Flujos

### En Nodos de Base de Datos

En el nodo PostgreSQL, MySQL, MariaDB, Oracle o MongoDB:

1. En el campo **Credencial**, seleccione la credencial
2. Los campos de conexión se completan automáticamente
3. Configure solo la consulta/operación

### En Nodos HTTP Request

1. En el campo **Credencial**, seleccione la credencial HTTP
2. La URL base y la autenticación se aplican automáticamente
3. En el campo **URL**, ingrese solo el path

### En Nodos SSH

1. En el campo **Credencial**, seleccione la credencial SSH
2. Los datos de conexión se aplican automáticamente
3. Configure solo los comandos

---

## Seguridad

- Las contraseñas y los tokens están **cifrados** en la base de datos
- Los valores sensibles están **enmascarados** en la interfaz
- Las credenciales son accesibles solo para usuarios con el permiso adecuado
- El propietario de la credencial puede editarla; otros requieren permiso de administrador

---

## Consejos

- **Cree credenciales por entorno** — Producción, Staging, Dev
- **Siempre pruebe** la conexión antes de usarla en un flujo
- **Use nombres descriptivos**: "PostgreSQL Producción" es mejor que "pg1"
- **Prefiera credenciales** en lugar de colocar contraseñas directamente en los nodos
- Al **duplicar entornos**, cree nuevas credenciales para cada uno
