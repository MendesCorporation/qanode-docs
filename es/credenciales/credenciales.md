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
