# Administración

La sección de administración permite gestionar usuarios, roles, permisos, configuración SMTP, alarmas, webhooks, auditoría y licenciamiento.

Acceda a través del ícono de **engranaje** (⚙️) en el menú lateral → **Configuración**.

---

## Usuarios

### Invitando Usuarios

1. Vaya a **Configuración** → **Usuarios**
2. Haga clic en **Invitar Usuario**
3. Complete:
   - **Correo**: Correo electrónico del nuevo usuario
   - **Rol**: Rol asignado (define los permisos)
4. Se enviará un correo de invitación con un enlace de activación

> **Requisito:** El SMTP debe estar configurado para el envío de invitaciones.

### Importación en Lote

Para registrar varios usuarios a la vez:

1. Vaya a **Configuración** → **Usuarios**
2. Haga clic en el ícono de **importación** (junto al botón de invitar)
3. Descargue la **plantilla Excel** haciendo clic en **Download Template**
4. Complete la hoja de cálculo con los datos de los usuarios:

| Columna | Descripción |
|---------|-------------|
| **name** | Nombre completo del usuario |
| **email** | Correo electrónico del usuario |
| **role** | Nombre del rol (ej.: Admin, Architect, Tester) |

5. Cargue el archivo Excel (.xlsx o .xls) arrastrándolo al área indicada o haciendo clic para seleccionar
6. El sistema procesará cada fila y enviará invitaciones por correo electrónico

Después del procesamiento, se muestra un resumen con:
- Cantidad de usuarios importados con éxito
- Lista de fallos con el número de fila, correo y motivo del error (ej.: correo ya existe, rol no encontrado, campos obligatorios faltantes)

> **Nota:** La importación respeta el límite de usuarios de la licencia. Si el archivo contiene más usuarios de los que permite el plan, el sistema informará y ofrecerá la opción de importar solo hasta el límite disponible.

### Gestionando Usuarios

En la lista de usuarios puede:

| Acción | Descripción |
|--------|-------------|
| **Activar/Desactivar** | Activa o desactiva el acceso del usuario |
| **Cambiar Rol** | Cambia el rol (y permisos) del usuario |
| **Eliminar** | Elimina el usuario permanentemente |

### Estado de Usuarios

| Estado | Descripción |
|--------|-------------|
| **Activo** | Usuario con acceso normal |
| **Inactivo** | Acceso bloqueado |
| **Pendiente** | Invitación enviada, esperando activación |

---

## Roles y Permisos

### Roles del Sistema

QANode incluye roles predefinidos que no pueden eliminarse:

| Rol | Descripción |
|-----|-------------|
| **Super Admin** | Acceso total a todas las funcionalidades |
| **Admin** | Acceso administrativo |
| **Architect** | Acceso de administración de proyectos |
| **Tester** | Acceso de escenarios y ejecuciones |

### Roles Personalizados

Cree roles con permisos específicos:

1. Vaya a **Configuración** → **Roles y Permisos**
2. Haga clic en **+ Nuevo Rol**
3. Defina el nombre, la vista de tablas y seleccione los permisos

### Vista de Tablas (Table View)

Al crear o editar un rol, el campo **Vista de Tablas** define qué tablas de la base de datos están accesibles en el **query builder** de los dashboards personalizados. Esto controla lo que el usuario puede consultar al crear widgets con consultas SQL o explorar datos en los dashboards.

| Perfil | Tablas accesibles |
|--------|-------------------|
| **admin** | Todas las tablas: Projects, Flows, Suites, Runs, Run Steps, Variables, Credentials, Users, Roles, Audit Log |
| **architect** | Projects, Flows, Suites, Runs, Run Steps, Variables |
| **tester** | Projects, Flows, Suites, Runs, Run Steps |
| **none** | Ninguna tabla — query builder deshabilitado |

> **Nota:** Las columnas sensibles (contraseñas, tokens, datos cifrados) se ocultan automáticamente, independientemente del perfil de vista.

#### Perfiles predeterminados de los roles del sistema

| Rol del Sistema | Vista de Tablas |
|-----------------|-----------------|
| **Super Admin** | admin |
| **Admin** | admin |
| **Architect** | architect |
| **Tester** | tester |

### Lista Completa de Permisos

#### Escenarios (Flows)
| Permiso | Descripción |
|---------|-------------|
| `flow.view` | Ver escenarios — permite ver la lista y detalles de los escenarios |
| `flow.create` | Crear escenarios |
| `flow.edit.own` | Editar escenarios propios (creados por el usuario) |
| `flow.edit.all` | Editar cualquier escenario (incluyendo los de otros usuarios) |
| `flow.delete.own` | Eliminar escenarios propios |
| `flow.delete.all` | Eliminar cualquier escenario |
| `flow.run` | Ejecutar escenarios |

#### Proyectos
| Permiso | Descripción |
|---------|-------------|
| `project.view` | Ver proyectos — permite ver la lista y detalles de los proyectos |
| `project.create` | Crear proyectos |
| `project.edit.own` | Editar proyectos propios (creados por el usuario) |
| `project.edit.all` | Editar cualquier proyecto |
| `project.archive` | Archivar proyectos |
| `project.delete` | Eliminar proyectos permanentemente |

#### Suites
| Permiso | Descripción |
|---------|-------------|
| `suite.view` | Ver suites — permite ver la lista y detalles de las suites |
| `suite.create` | Crear suites |
| `suite.edit` | Editar suites |
| `suite.delete` | Eliminar suites |
| `suite.run` | Ejecutar suites |

#### Ejecuciones (Runs)
| Permiso | Descripción |
|---------|-------------|
| `run.view` | Ver ejecuciones propias — permite ver el historial de ejecuciones realizadas por el usuario |
| `run.view.all` | Ver todas las ejecuciones — permite ver ejecuciones de todos los usuarios |
| `run.cancel` | Cancelar ejecuciones en curso |
| `run.delete` | Eliminar registros de ejecución |

#### Variables
| Permiso | Descripción |
|---------|-------------|
| `variable.view` | Ver variables — permite ver la lista de variables |
| `variable.create` | Crear variables |
| `variable.edit` | Editar variables |
| `variable.delete` | Eliminar variables |
| `variable.view.secret` | Ver valores secretos — permite revelar el valor de variables marcadas como secretas |

#### Credenciales
| Permiso | Descripción |
|---------|-------------|
| `credential.view` | Ver credenciales — permite ver la lista de credenciales registradas |
| `credential.create` | Crear credenciales |
| `credential.edit` | Editar credenciales |
| `credential.delete` | Eliminar credenciales |

#### Proveedores
| Permiso | Descripción |
|---------|-------------|
| `provider.view` | Ver proveedores |
| `provider.create` | Registrar proveedores |
| `provider.edit` | Editar proveedores |
| `provider.delete` | Eliminar proveedores |

#### Informes (Reports)
| Permiso | Descripción |
|---------|-------------|
| `report.view` | Ver informes — permite acceder a los informes PDF generados en las ejecuciones |
| `report.export` | Exportar informes — permite descargar informes en PDF |

#### Dashboards
| Permiso | Descripción |
|---------|-------------|
| `dashboard.view` | Ver dashboards — permite acceder y ver dashboards existentes |
| `dashboard.create` | Crear dashboards personalizados |
| `dashboard.edit` | Editar dashboards |
| `dashboard.delete` | Eliminar dashboards |
| `dashboard.share` | Compartir dashboards — permite hacer dashboards públicos o compartirlos con roles específicos |
| `dashboard.sql` | Ejecutar SQL — permite usar consultas SQL directas en el query builder de widgets |

#### Webhooks
| Permiso | Descripción |
|---------|-------------|
| `webhook.view` | Ver webhooks |
| `webhook.create` | Crear webhooks |
| `webhook.edit` | Editar webhooks |
| `webhook.delete` | Eliminar webhooks |

#### Usuarios
| Permiso | Descripción |
|---------|-------------|
| `user.view` | Ver lista de usuarios |
| `user.invite` | Invitar nuevos usuarios — envía correo de invitación con enlace de activación |
| `user.edit` | Editar datos de usuarios (cambiar rol, información) |
| `user.delete` | Eliminar usuarios permanentemente |
| `user.deactivate` | Desactivar/activar usuarios — bloquea o restaura el acceso sin eliminar |

#### Roles
| Permiso | Descripción |
|---------|-------------|
| `role.view` | Ver roles |
| `role.create` | Crear roles |
| `role.edit` | Editar roles |
| `role.delete` | Eliminar roles |

#### Configuración
| Permiso | Descripción |
|---------|-------------|
| `settings.view` | Ver configuración — acceso básico a la página de configuración |
| `settings.smtp` | Configurar SMTP — permite cambiar la configuración de correo electrónico |
| `settings.mfa` | Configurar MFA — permite gestionar la autenticación de dos factores |
| `settings.audit` | Ver auditoría — acceso al registro de auditoría |
| `settings.report_template` | Gestionar plantillas de informe — permite crear y editar plantillas PDF |

---

## Configuración SMTP

Para el envío de correos electrónicos (invitaciones, informes, alarmas):

1. Vaya a **Configuración** → **SMTP**
2. Complete:

| Campo | Descripción |
|-------|-------------|
| **Host** | Servidor SMTP (ej.: `smtp.gmail.com`) |
| **Puerto** | Puerto del servidor |
| **Seguridad** | STARTTLS (587), TLS/SSL (465) o Ninguna |
| **Usuario** | Correo/usuario de autenticación |
| **Contraseña** | Contraseña del correo |
| **Remitente** | Dirección de envío (from) |

3. Haga clic en **Guardar** — el sistema prueba la conexión automáticamente

---

## Alarmas

Configure notificaciones automáticas para fallos en suites:

1. Vaya a **Configuración** → **Alarmas**
2. Configure:
   - **Suites monitoreadas** — qué suites disparan una alarma
   - **Destinatarios** — quién recibe la notificación
   - **Condición** — cuándo notificar (fallo, siempre, etc.)

> **Requisito:** El SMTP debe estar configurado para el envío de alarmas por correo electrónico.

---

## Webhooks

Los webhooks permiten enviar automáticamente notificaciones HTTP POST a servicios externos cuando ocurren eventos de ejecución. Cada usuario gestiona sus propios webhooks de forma independiente.

### Configurando Webhooks

1. Vaya a **Configuración** → **Webhooks**
2. Haga clic en **+ Nuevo Webhook**
3. Complete:

| Campo | Descripción |
|-------|-------------|
| **Nombre** | Nombre identificador del webhook |
| **URL** | Dirección HTTP/HTTPS que recibirá el POST |
| **Secret** | (Opcional) Clave HMAC para firma del payload |
| **Eventos** | Qué eventos disparan el webhook |
| **Alcance** | Filtro por proyecto, suite o escenario específico |

### Eventos Disponibles

| Evento | Descripción |
|--------|-------------|
| `run.completed` | Ejecución de escenario finalizada (éxito o fallo) |
| `run.success` | Ejecución de escenario finalizada con éxito |
| `run.failed` | Ejecución de escenario finalizada con fallo |
| `suite.completed` | Ejecución de suite finalizada (éxito o fallo) |
| `suite.success` | Ejecución de suite finalizada con éxito |
| `suite.failed` | Ejecución de suite finalizada con fallo |

### Alcance (Filtros)

Puede filtrar los webhooks para que se disparen solo en contextos específicos:

- **Proyecto** — Solo ejecuciones de un proyecto específico
- **Suite** — Solo ejecuciones de una suite específica
- **Escenario** — Solo ejecuciones de un escenario específico

> Al seleccionar un proyecto, las suites y escenarios se filtran para mostrar solo los de ese proyecto. Al seleccionar una suite, el filtro de escenario se oculta.

### Payload del Webhook

El cuerpo del POST enviado contiene:

```json
{
  "event": "run.completed",
  "timestamp": "2026-02-13T10:30:00.000Z",
  "data": {
    "runId": "uuid",
    "flowId": "uuid",
    "flowName": "Nome do Cenário",
    "suiteId": "uuid",
    "suiteName": "Nome da Suíte",
    "projectId": "uuid",
    "projectName": "Nome do Projeto",
    "status": "success",
    "durationMs": 1234,
    "executor": "Nome do Usuário",
    "errorSummary": null
  }
}
```

### Firma HMAC

Si el campo **Secret** está completo, QANode agrega el encabezado `X-QANode-Signature` con la firma HMAC-SHA256 del payload. Esto permite que el servicio receptor valide la autenticidad de la solicitud.

Ejemplo de encabezado:
```
X-QANode-Signature: sha256=abc123...
```

### Reintentos

En caso de fallo en la entrega (error de red o código HTTP diferente de 2xx), QANode intenta reenviar automáticamente hasta **3 veces** con intervalos crecientes:

| Intento | Intervalo |
|---------|-----------|
| 1° | Inmediato |
| 2° | 1 segundo |
| 3° | 5 segundos |

### Registro de Entregas

Para cada webhook, es posible ver el historial de las últimas 50 entregas:

1. Haga clic en el ícono de **lista** junto al webhook
2. Vea el estado de cada entrega (OK/Falló), código HTTP, intentos y errores

### Probando Webhooks

Haga clic en el ícono de **play** junto al webhook para enviar un payload de prueba. Esto permite verificar si la URL es accesible y responde correctamente.

### Permisos

Los siguientes roles tienen acceso a webhooks:

| Rol | Acceso |
|-----|--------|
| **Super Admin** | Total |
| **Admin** | Total |
| **Architect** | Total |
| **Tester** | Sin acceso |

> Cada usuario visualiza y gestiona solo sus propios webhooks.

---

## Auditoría

El registro de auditoría registra todas las acciones importantes en el sistema:

### Acciones Rastreadas

| Acción | Descripción |
|--------|-------------|
| **create** | Creación de recurso |
| **update** | Actualización de recurso |
| **delete** | Eliminación de recurso |
| **execute** | Ejecución de escenario/suite |
| **login** | Inicio de sesión de usuario |
| **logout** | Cierre de sesión de usuario |
| **import** | Importación de datos |
| **export** | Exportación de datos |
| **test** | Prueba de conexión |
| **invite** | Invitación de usuario |
| **setup** | Configuración inicial |

### Entidades Rastreadas

Proyectos, Flujos, Suites, Ejecuciones, Variables, Credenciales, Proveedores, Usuarios, Roles, Configuración, Sistema.

### Información del Registro

| Campo | Descripción |
|-------|-------------|
| **Acción** | Tipo de acción realizada |
| **Entidad** | Tipo y nombre del recurso afectado |
| **Usuario** | Quién realizó la acción |
| **IP** | Dirección IP de origen |
| **Fecha/Hora** | Cuándo ocurrió la acción |

### Visualizando

1. Vaya a **Configuración** → **Auditoría**
2. Navegue por el historial usando paginación
3. Use filtros para buscar acciones específicas


---

## Cambio de Contraseña

Cualquier usuario puede cambiar su propia contraseña:

1. Vaya a **Configuración** → **Cambiar Contraseña**
2. Ingrese la contraseña actual
3. Ingrese y confirme la nueva contraseña
4. Haga clic en **Guardar**

---

## MFA (Autenticación de Dos Factores)

Para mayor seguridad, active la autenticación de dos factores:

1. Vaya a **Configuración** → **MFA**
2. Escanee el código QR con una app de autenticación (Google Authenticator, Authy, etc.)
3. Ingrese el código generado para confirmar
4. El MFA estará activo — se solicitarán códigos en cada inicio de sesión

---

## Licencia

Gestione la licencia de QANode:

1. Vaya a **Configuración** → **Licencia**
2. Visualice:
   - **Estado** de la licencia (válida/expirada)
   - **Fecha de expiración**
   - **Días restantes**
   - **Límite de usuarios**
   - **Usuarios activos** vs límite

---

## Consejos

- **Configure el SMTP primero** — muchas funcionalidades dependen del correo electrónico
- Use **roles personalizados** para un control de acceso detallado
- **Monitoree la auditoría** regularmente para detectar acciones inesperadas
- **Active MFA** para cuentas de administrador
- **Desactive usuarios** en lugar de eliminarlos — preserva el historial de auditoría
- **Use webhooks** para integrar QANode con Slack, Teams, Discord o cualquier servicio que acepte HTTP POST
- **Defina el secret** en los webhooks para garantizar la autenticidad de las solicitudes
