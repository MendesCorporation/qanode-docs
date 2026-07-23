# SSO y Aprovisionamiento de Usuarios

QANode permite centralizar la autenticación en un proveedor de identidad (IdP) mediante **OpenID Connect (OIDC)** o **SAML 2.0**. El ciclo de vida de los usuarios puede gestionarse mediante **JIT** (creación en el primer inicio de sesión) o **SCIM 2.0** (creación, activación y desactivación enviadas por el IdP).

Esta guía explica qué controla el IdP, qué continúa bajo el control de QANode y cómo configurar los proveedores más comunes.

> **Disponibilidad:** la configuración es exclusiva del Super Admin y se encuentra en **Configuración → General → Inicio de sesión único**.

---

## Visión General

SSO y SCIM resuelven problemas diferentes:

| Recurso | Responsabilidad | Qué hace |
|---------|-----------------|----------|
| **OIDC o SAML** | Autenticación | Confirma la identidad y permite entrar en QANode con la cuenta corporativa |
| **JIT** | Aprovisionamiento durante el login | Crea el usuario de QANode en su primer inicio de sesión |
| **SCIM** | Ciclo de vida | Crea, activa y desactiva usuarios de QANode desde el IdP |
| **Roles de QANode** | Autorización | Define a qué proyectos, pantallas y acciones puede acceder el usuario |

En la versión actual, el IdP **no controla los roles ni los permisos**. QANode recibe la identidad y el estado del usuario, pero sigue siendo la fuente de verdad para la autorización.

```text
IdP (autenticación y ciclo de vida)
       │
       ├── OIDC o SAML ──> identidad e inicio de sesión
       └── SCIM ─────────> crear, activar y desactivar
                                  │
                                  ▼
QANode (rol, permisos, proyectos y recursos)
```

### Qué combinación elegir

| Escenario | Recomendación |
|-----------|---------------|
| El proveedor admite OIDC | Use **OIDC** para la autenticación |
| La organización exige SAML | Use **SAML 2.0** |
| El proveedor admite SCIM | Use **SCIM** para el aprovisionamiento |
| El proveedor no tiene SCIM | Use **JIT** con un intervalo corto de revalidación |
| La desactivación rápida es obligatoria | Use **SCIM**; JIT por sí solo no sincroniza la desactivación con el registro local |

---

## Antes de Comenzar

Compruebe estos puntos antes de configurar el proveedor:

1. QANode debe estar publicado en una URL **HTTPS** accesible para el IdP. No use `localhost` en producción.
2. Confirme que el Super Admin conoce la contraseña local y que su correo en QANode coincide exactamente con el devuelto por el IdP.
3. Elija un **rol por defecto** con pocos privilegios para los usuarios nuevos. Super Admin nunca está disponible como valor por defecto.
4. Decida si el aprovisionamiento usará **JIT** o **SCIM**.
5. En el IdP, conceda acceso a la aplicación solo a los usuarios o grupos que deban entrar en QANode.
6. Guarde el secreto OIDC y el token SCIM en una bóveda de secretos.
7. Realice la prueba desde una sesión de Super Admin antes de activar SSO para toda la instancia.

> QANode genera las URLs de callback, ACS y metadata a partir de su URL pública. Si el dominio, protocolo o puerto son incorrectos, el proveedor rechazará el retorno.

---

## Configuración General en QANode

En **Configuración → General → Inicio de sesión único**, complete los campos comunes:

| Campo | Descripción |
|-------|-------------|
| **Protocolo** | `OpenID Connect (OIDC)` o `SAML 2.0` |
| **Nombre del proveedor** | Nombre mostrado a los administradores, por ejemplo `Microsoft Entra` o `Google Workspace` |
| **Aprovisionamiento de usuarios** | `JIT — Crear en el primer acceso` o `SCIM — Gestionado por el IdP` |
| **Rol por defecto** | Rol asignado al usuario creado por JIT o SCIM |
| **Revalidar SSO cada** | Intervalo entre nuevas validaciones en el IdP: 30 minutos, 1, 4, 8 o 24 horas |
| **Acceso local de emergencia del Super Admin** | Permite que el Super Admin siga entrando con la contraseña local |

### Rol por defecto y permisos

- Un usuario nuevo recibe el **rol por defecto** seleccionado.
- El IdP no envía ni sustituye roles en la versión actual.
- Un administrador puede cambiar el rol posteriormente en **Configuración → Usuarios**.
- Una sincronización posterior no deshace ese cambio.
- El rol **Super Admin** no puede asignarse mediante JIT o SCIM.
- Aplique el mínimo privilegio: comience con `Tester` o un rol personalizado restringido y amplíe los permisos solo cuando sea necesario.

### Vínculo del Super Admin

La primera prueba vincula la cuenta local del Super Admin con la identidad externa. El correo devuelto por el IdP debe coincidir con el registrado en QANode, ignorando únicamente diferencias entre mayúsculas y minúsculas.

Después del primer vínculo, QANode también utiliza el identificador estable del proveedor (`sub` en OIDC o `NameID` en SAML). No configure un identificador temporal o que cambie en cada login.

### Acceso local de emergencia

Cuando está habilitado, solo el Super Admin puede seguir usando una contraseña local. Esta opción sirve para recuperar la administración cuando el IdP no está disponible o es necesario corregir la configuración de SSO.

Recomendaciones:

- guarde la contraseña en una bóveda corporativa;
- limite quién puede recuperarla;
- pruebe periódicamente el procedimiento de emergencia;
- deshabilite la opción si la política de la empresa prohíbe la autenticación local.

---

## Aprovisionamiento JIT

En modo **JIT**, el usuario se crea en QANode cuando completa su primer inicio de sesión SSO.

### Primer acceso

1. El usuario inicia el login desde QANode.
2. El IdP autentica a la persona y devuelve un identificador estable, correo y nombre.
3. Si ya existe un usuario con el mismo correo, QANode vincula la identidad externa a ese registro.
4. Si no existe, QANode crea un usuario activo, sin contraseña local y con el rol por defecto.
5. Los siguientes accesos utilizan el identificador inmutable del proveedor.

El nombre y el correo pueden actualizarse con los datos de SSO. El rol permanece bajo el control de QANode.

### Qué ocurre cuando el usuario se bloquea en el IdP

JIT **no envía una desactivación a la base de datos de QANode**. El registro puede continuar apareciendo como activo, pero el usuario no podrá renovar la autenticación después de que el IdP bloquee su acceso.

El tiempo máximo para que una sesión abierta vuelva al IdP depende de **Revalidar SSO cada**. Para reducir la ventana de acceso sin SCIM, use 30 o 60 minutos.

Tampoco use la desactivación manual en QANode como fuente de verdad para un usuario JIT que sigue autorizado en el IdP: un nuevo login SSO válido puede reactivar el registro. Retire primero el acceso a la aplicación en el IdP.

> Si el registro local debe desactivarse automáticamente, use SCIM. JIT crea usuarios durante el login; no es una sincronización completa del ciclo de vida.

---

## Aprovisionamiento SCIM

En modo **SCIM**, el IdP llama a la API SCIM 2.0 de QANode para crear, actualizar, activar o desactivar usuarios.

### Datos proporcionados por QANode

| Campo | Uso en el proveedor |
|-------|---------------------|
| **URL base de SCIM** | Endpoint de aprovisionamiento terminado en `/api/scim/v2` |
| **Token secreto** | Token Bearer utilizado por el proveedor para autenticar las llamadas |

El token se muestra solamente al generarlo. Cópielo directamente al IdP y guárdelo de forma segura. Al rotarlo, el valor anterior deja de funcionar inmediatamente.

### Recursos admitidos

QANode gestiona usuarios SCIM y admite:

- creación y consulta de usuarios;
- actualizaciones completas y parciales;
- activación y desactivación mediante `active`;
- eliminación SCIM tratada como **desactivación**, sin borrar físicamente al usuario;
- filtros necesarios para localizar usuarios por identidad.

La versión actual no sincroniza grupos, roles, contraseñas, fotos ni permisos mediante SCIM. Deshabilite el mapeo de grupos en el proveedor.

### Mapeo recomendado

Mantenga únicamente los atributos necesarios:

| Atributo SCIM en QANode | Obligatorio | Origen recomendado en el IdP | Observación |
|-------------------------|-------------|--------------------------------|------------|
| `userName` | Sí | Correo corporativo real | Debe ser un correo válido y único; úselo como atributo de correspondencia con precedencia 1 |
| `active` | Recomendado | Estado activo del usuario | Controla activación y desactivación |
| `displayName` | Recomendado | Nombre para mostrar | Nombre mostrado en QANode |
| `name.givenName` | No | Nombre | Opcional |
| `name.familyName` | No | Apellido | Opcional |
| `emails[type eq "work"].value` | No | El mismo correo de `userName` | Ayuda a proveedores que envían una colección de correos |
| `externalId` | Recomendado | ID inmutable del objeto en el IdP | No use un alias que pueda cambiar |

QANode exige que `userName` o el correo principal recibido sea una dirección válida. No debe usarse un UPN técnico como `GUID@tenant.onmicrosoft.com` cuando no representa el correo real de login.

### Desactivación y reactivación

- Cuando el IdP envía `active: false`, QANode desactiva al usuario e invalida su acceso.
- Cuando envía `active: true`, el usuario vuelve a estar activo, respetando el límite de usuarios de la licencia.
- Una operación SCIM `DELETE` también desactiva al usuario; no elimina definitivamente sus datos.
- Los usuarios controlados por SCIM deben activarse o desactivarse en el IdP, no manualmente en QANode.
- El Super Admin nunca se desactiva ni se gestiona mediante SCIM.

### Ventanas de sincronización

El tiempo de sincronización depende del proveedor. Algunos trabajan por ciclos y un cambio puede tardar varios minutos. Use la función de aprovisionamiento bajo demanda durante la implantación y espere el siguiente ciclo para validar el funcionamiento normal.

---

## Configuración de OIDC

OIDC es la opción recomendada cuando el proveedor admite el protocolo. QANode usa el flujo **Authorization Code con PKCE** y solicita los scopes `openid`, `profile` y `email`.

### Campos de QANode

| Campo en QANode | Dónde encontrarlo en el IdP |
|-----------------|------------------------------|
| **URL del emisor** | Valor `issuer` de `/.well-known/openid-configuration` |
| **Client ID** | ID público de la aplicación OIDC |
| **Client secret** | Valor del secreto de la aplicación; no use el ID ni el nombre del secreto |
| **Autenticación del endpoint de token** | `client_secret_post` o `client_secret_basic`, según lo admitido por el proveedor |
| **URI de redireccionamiento** | Cópiela desde QANode a la lista de callbacks/redirect URIs del IdP |

> La URL del emisor debe coincidir exactamente con el valor `issuer` publicado por el proveedor. No agregue ni elimine manualmente barras, paths o versiones.

El IdP debe devolver:

- un `sub` estable y no vacío;
- un correo en el claim `email` o, como alternativa, en `preferred_username`;
- un nombre en `name`, cuando esté disponible;
- un valor de `email_verified` distinto de `false`, cuando ese claim esté presente.

### Microsoft Entra ID y External ID

1. Abra **Registros de aplicaciones** y seleccione la aplicación de QANode.
2. En **Autenticación**, añada la **URI de redireccionamiento** de QANode a la plataforma **Web**.
3. En **Certificados y secretos**, cree un secreto y copie su **Valor**. El valor solo se muestra una vez; el **ID del secreto** no funciona como Client Secret.
4. Copie el **ID de la aplicación (cliente)** en Client ID.
5. Abra **Puntos de conexión** y acceda al **Documento de metadatos de OpenID Connect**.
6. Copie el valor exacto de `issuer` del JSON en **URL del emisor**.

En tenants Microsoft Entra de fuerza de trabajo, el emisor suele tener este formato:

```text
https://login.microsoftonline.com/{tenant-id}/v2.0
```

En Microsoft Entra External ID, normalmente utiliza el dominio `ciamlogin.com`:

```text
https://{tenant}.ciamlogin.com/{tenant-id}/v2.0
```

Use siempre el valor publicado por el documento de su tenant. Comience con `client_secret_post`; cambie a `client_secret_basic` únicamente si el documento del proveedor indica soporte y el cambio es necesario.

### Google

1. Configure la pantalla de consentimiento OAuth en Google Cloud Console.
2. Cree un **OAuth Client ID** de tipo **Web application**.
3. Añada la URI de QANode en **Authorized redirect URIs**.
4. Copie el Client ID y el Client Secret.
5. Use este emisor:

```text
https://accounts.google.com
```

Google acepta `client_secret_post` y `client_secret_basic`. La URI de redireccionamiento debe coincidir exactamente, incluyendo protocolo, dominio, puerto, path y barra final.

### Okta

1. Cree una integración **OIDC — Web Application**.
2. Añada la URI de QANode en **Sign-in redirect URIs**.
3. Asigne usuarios o grupos a la aplicación.
4. Copie Client ID y Client Secret.
5. Use el **Issuer URI** del Authorization Server elegido. Para el servidor por defecto, normalmente es:

```text
https://{su-dominio-okta}/oauth2/default
```

### Keycloak

1. Cree un client de tipo **OpenID Connect**.
2. Habilite **Standard flow** y la autenticación del client.
3. Añada la URI de QANode en **Valid redirect URIs**.
4. Copie el Client ID y el secreto de la pestaña de credenciales.
5. Use este emisor:

```text
https://{host-keycloak}/realms/{realm}
```

### Auth0

1. Cree una **Regular Web Application**.
2. Añada la URI de QANode en **Allowed Callback URLs**.
3. Copie el Client ID y el Client Secret.
4. Use el valor `issuer` del discovery del tenant o del dominio personalizado:

```text
https://{su-dominio-auth0}/
```

### Zoho

Cree un cliente OIDC para una aplicación web, registre la URI de redireccionamiento y copie el Client ID y el Client Secret. Use el valor `issuer` del documento de discovery del **data center de su organización**. No suponga que todas las cuentas usan el dominio `.com`; las organizaciones europeas, indias y de otras regiones pueden publicar otro dominio.

---

## Configuración de SAML 2.0

En SAML, QANode es el **Service Provider (SP)** y la plataforma corporativa es el **Identity Provider (IdP)**.

### Datos de QANode para registrar en el IdP

| Campo en QANode | Nombre habitual en el proveedor |
|-----------------|----------------------------------|
| **URL de login** | Sign-on URL, Start URL o Login URL |
| **Emisor / Entity ID** | Identifier, Audience URI, SP Entity ID o Client ID |
| **URL del servicio consumidor de aserciones (ACS)** | Reply URL, ACS URL, Single sign-on URL o Recipient URL |
| **URL de metadatos** | SP Metadata URL, cuando el proveedor permite importar metadata |
| **URL de logout** | Retorno local a la página de login; no es un endpoint SAML SLO |

> En la versión actual, el login SAML debe comenzar en QANode. Abrir la aplicación directamente desde el catálogo del IdP (IdP-initiated) o usar el botón de prueba del proveedor puede fallar porque no existe el estado de login creado por QANode.

### Datos del IdP para registrar en QANode

| Campo en QANode | Nombre habitual en el proveedor |
|-----------------|----------------------------------|
| **URL de SSO del IdP** | Login URL, SSO URL o SAML 2.0 Endpoint |
| **Emisor del IdP** | IdP Entity ID, Issuer o Identifier |
| **Certificado de firma X.509** | Certificate, Signing Certificate o Certificate Base64/PEM |
| **Atributo de correo** | Nombre exacto del atributo que contiene el correo; por defecto `email` |
| **Atributo de nombre** | Nombre exacto del atributo que contiene el nombre; por defecto `displayName` |

Cuando el proveedor entregue PEM, incluya `-----BEGIN CERTIFICATE-----` y `-----END CERTIFICATE-----` al pegar el certificado.

### Requisitos de compatibilidad SAML

Configure el proveedor con estas reglas:

| Configuración | Valor esperado |
|---------------|----------------|
| Binding de ACS | HTTP POST |
| Aserción SAML | **Firmada** |
| Respuesta SAML | Puede estar firmada o no |
| Cifrado de la aserción | Deshabilitado |
| AuthnRequest de QANode | Sin firma; el IdP no debe exigirla |
| NameID | Estable y no vacío; formato persistente o correo |
| Correo | Enviado en el atributo configurado o en un claim de correo conocido |

QANode busca el correo en el atributo configurado y después en `email`, `mail`, el OID LDAP de correo o el NameID cuando tiene formato de correo. Aun así, configure explícitamente el atributo para hacer la integración más predecible.

### Microsoft Entra ID — SAML

En **Aplicaciones empresariales → Inicio de sesión único → SAML**:

| Campo en Microsoft Entra | Valor de QANode |
|--------------------------|-----------------|
| **Identifier (Entity ID)** | Emisor / Entity ID |
| **Reply URL (ACS)** | URL ACS |
| **Sign-on URL** | URL de login |
| **Logout URL** | Déjelo en blanco |

Configure estos atributos:

| Claim | Origen recomendado |
|-------|--------------------|
| `email` | `user.mail` u otro atributo que contenga el correo real |
| `displayName` | `user.displayname` |

Después copie en QANode:

- **Login URL** → URL de SSO del IdP;
- **Microsoft Entra Identifier** → Emisor del IdP;
- certificado de firma → Certificado X.509.

Elija una opción que firme la **aserción**. No habilite el cifrado ni exija una AuthnRequest firmada.

### Google Workspace — SAML

En **Apps → Web and mobile apps → Add app → Add custom SAML app**:

1. Copie **SSO URL**, **Entity ID** y el certificado de Google en los campos del IdP en QANode.
2. Configure **ACS URL** con la URL ACS de QANode.
3. Configure **Entity ID** con el Emisor / Entity ID de QANode.
4. Use el correo principal como NameID con formato `EMAIL`.
5. **Signed response** puede permanecer desmarcado: Google sigue firmando la aserción, que es lo que QANode exige.
6. Mapee el correo principal a `email` y el nombre para mostrar a `displayName`, cuando esté disponible.
7. Active el servicio para las unidades organizativas o grupos requeridos.

Inicie la prueba desde QANode, no desde el botón **Test SAML login** de Google.

### Okta — SAML

| Campo en Okta | Valor |
|----------------|-------|
| **Single sign-on URL** | URL ACS de QANode |
| **Use this for Recipient URL and Destination URL** | Habilitado |
| **Audience URI (SP Entity ID)** | Emisor / Entity ID de QANode |
| **Default RelayState** | En blanco |
| **Name ID format** | EmailAddress |
| **Application username** | Email |
| **Assertion Signature** | Signed |
| **Assertion Encryption** | Unencrypted |
| **Signed Requests** | Deshabilitado |

Añada `email = user.email` y `displayName = user.displayName`. Después copie Identity Provider Single Sign-On URL, Issuer y el certificado X.509 en QANode.

### OneLogin — SAML

Configure:

- **ACS (Consumer) URL** y **Recipient** con la URL ACS;
- **Audience** con el Entity ID de QANode;
- RelayState en blanco;
- correo como NameID;
- aserción firmada y cifrado deshabilitado.

Copie **SAML 2.0 Endpoint (HTTP)**, **Issuer URL** y el certificado X.509 en QANode.

### Zoho — SAML

Cree una aplicación SAML personalizada y use:

- **ACS URL** = URL ACS de QANode;
- **Entity ID / Audience** = Emisor de QANode;
- **Start URL** = URL de login de QANode, cuando el campo esté disponible;
- **NameID** = correo corporativo;
- atributos `email` y `displayName`.

Copie en QANode la URL de login SAML, el emisor de Zoho y el certificado proporcionado por la plataforma.

### JumpCloud — SAML

Configure SP Entity ID, ACS URL y Login URL con los valores de QANode. Use el correo como NameID y seleccione una opción que firme la **assertion** o la response y la assertion. El valor por defecto que firma únicamente la response no es suficiente. Mantenga el cifrado deshabilitado y no exija solicitudes firmadas.

### Keycloak — SAML

Cree un client SAML con:

- **Client ID** = Entity ID de QANode;
- **Master SAML Processing URL** y **Valid Redirect URIs** = URL ACS;
- **Client signature required** = deshabilitado;
- firma de assertions = habilitada;
- cifrado de assertions = deshabilitado;
- mappers para `email` y `displayName`.

Use la URL SAML del realm como URL de SSO del IdP, el emisor del realm y el certificado público de firma en QANode.

### PingOne, Duo y Auth0 — SAML

Estos proveedores utilizan los mismos campos del estándar SAML:

- ACS/Recipient/Single Sign-on URL = URL ACS de QANode;
- Audience/Entity ID = Entity ID de QANode;
- NameID = correo estable;
- aserción firmada;
- cifrado deshabilitado;
- solicitud firmada no obligatoria.

Copie la SSO URL, el Issuer/Entity ID del IdP y el certificado X.509 presentados por el proveedor.

---

## SCIM en Microsoft Entra ID y External ID

Microsoft Entra configura SCIM en una **Aplicación empresarial**. Esta aplicación de aprovisionamiento puede estar separada del registro de aplicación OIDC.

### Crear la aplicación SCIM

1. Abra **Aplicaciones empresariales → Nueva aplicación**.
2. Elija **Crear su propia aplicación** y la opción para integrar otra aplicación que no se encuentra en la galería.
3. Abra **Aprovisionamiento** y elija el modo automático.
4. Introduzca la URL base SCIM de QANode en **Tenant URL**.
5. Introduzca el token generado por QANode en **Secret Token**.
6. Seleccione **Test Connection**.
7. En **Mappings**, mantenga el aprovisionamiento de usuarios y deshabilite el de grupos.
8. Asigne los usuarios o grupos que deben sincronizarse.
9. Use **Provision on demand** para probar un usuario.
10. Después de validar, seleccione **Start provisioning**.

### Mapeo mínimo recomendado en Entra

| Destino en QANode | Origen en Entra | Configuración |
|--------------------|-----------------|---------------|
| `userName` | Atributo que contiene el correo real | Match objects = `Yes`, precedence = `1`, apply = `Always` |
| `active` | `Not([IsSoftDeleted])` | Apply = `Always` |
| `displayName` | `displayName` | Apply = `Always` |
| `emails[type eq "work"].value` | El mismo atributo de correo | Opcional |
| `externalId` | `objectId` u otro ID inmutable expuesto | Recomendado |

El atributo calculado `IsSoftDeleted` pasa a verdadero cuando el usuario se deshabilita, se elimina de la aplicación, se borra o queda fuera del ámbito. Mantenga `active ← Not([IsSoftDeleted])` para que estas acciones desactiven la cuenta en QANode.

### Atención al correo en External ID

En tenants External ID:

- `mail` puede estar vacío;
- `userPrincipalName` puede ser un identificador técnico como `GUID@tenant.onmicrosoft.com`;
- el correo real puede existir únicamente en la colección **Identidades** del usuario.

En esa situación, `userName ← mail` produce un valor vacío y QANode responde:

```text
A valid userName/email is required
```

No lo sustituya por un UPN técnico solo para que la prueba pase. El valor final enviado a `userName` debe ser el correo real del usuario.

Procedimiento recomendado:

1. Abra **Provision on demand** y seleccione al usuario.
2. Inspeccione **Modified attributes / Data flow**.
3. Confirme que el valor de destino de `userName` es un correo real.
4. Si `mail` está vacío, copie o sincronice el correo de login en un atributo de origen expuesto por el aprovisionamiento, como `mail` o una extensión apropiada.
5. Mapee ese atributo a `userName` y, opcionalmente, a `emails[type eq "work"].value`.
6. Ejecute de nuevo el aprovisionamiento bajo demanda antes de iniciar el ciclo automático.

Si la colección **Identidades** no está disponible como origen en la interfaz de mapeo, esa configuración de aprovisionamiento no puede utilizarla directamente. Materialice el correo en otra propiedad aprovisionable o sincronícelo mediante un proceso administrativo del tenant.

### Relación entre las aplicaciones OIDC y SCIM

El portal no vincula automáticamente las dos aplicaciones. Trabajan juntas porque utilizan:

- el mismo tenant;
- los mismos usuarios o grupos asignados;
- el mismo correo normalizado;
- QANode como destino común.

Asigne los mismos usuarios o grupos tanto a la aplicación OIDC de login como a la aplicación empresarial SCIM. SCIM crea al usuario y el primer login OIDC vincula la identidad externa mediante el correo.

---

## Guardar, Probar y Activar

Seleccionar SSO no activa la integración inmediatamente. Use esta secuencia segura:

1. Complete la configuración.
2. Seleccione **Guardar borrador**.
3. Seleccione **Probar configuración**.
4. Entre en el IdP con la misma cuenta que el Super Admin actual.
5. Vuelva a QANode y confirme el estado **Verificado**.
6. Seleccione **Activar SSO**.

QANode solo permite activar la configuración exacta probada por el mismo Super Admin. Cambiar la URL, emisor, client, secreto, certificado u otro dato del proveedor invalida la verificación y exige una nueva prueba.

### Efectos de la activación

Al activar SSO:

- el MFA local se deshabilita globalmente;
- se eliminan los secretos MFA locales;
- los usuarios no pueden habilitar MFA mientras SSO esté activo;
- se invalidan los tokens de invitaciones pendientes;
- se bloquean las invitaciones, su reenvío, la aceptación y la importación de usuarios;
- se invalidan las sesiones locales existentes;
- los usuarios pasan a entrar mediante OIDC o SAML;
- solo el Super Admin puede conservar el login local cuando está habilitado el acceso de emergencia.

Exija MFA en el propio IdP mediante Acceso Condicional o la política equivalente del proveedor.

### Modificar una configuración activa

Cambiar datos específicos del proveedor desactiva SSO automáticamente para evitar que una configuración no verificada bloquee a toda la organización. Después del cambio:

1. guarde el borrador;
2. vuelva a probar;
3. active de nuevo SSO.

Cambiar el rol por defecto o el intervalo de revalidación no modifica los roles ya asignados.

---

## Operación y Mantenimiento

### Entrada de usuarios

El usuario debe seleccionar **Entrar con SSO** en la página de login de QANode. Para SAML, no use el tile de la aplicación en el portal del IdP como flujo principal.

### Rotación del certificado SAML

QANode mantiene un certificado de firma del IdP por configuración. Antes de que expire el certificado actual:

1. genere o active el nuevo certificado en el IdP;
2. copie el nuevo certificado en QANode;
3. guarde el borrador — SSO se desactiva por seguridad;
4. pruebe la integración;
5. vuelva a activarla;
6. revoque el certificado antiguo en el IdP después de validar.

Planifique una ventana corta de mantenimiento, ya que la versión actual no superpone automáticamente dos certificados.

### Rotación del token SCIM

1. Genere un nuevo token en QANode.
2. Actualice inmediatamente **Secret Token** en el IdP.
3. Pruebe la conexión.
4. Ejecute un aprovisionamiento bajo demanda.

El token anterior deja de funcionar después de la rotación.

### Desactivar SSO

Al desactivar SSO, la autenticación local vuelve a estar disponible. Los usuarios creados únicamente mediante JIT o SCIM no tienen contraseña local y deben completar un proceso de recuperación/definición de contraseña antes de entrar localmente.

El token SCIM no se elimina automáticamente al desactivar SSO. Revóquelo por separado si el aprovisionamiento también debe detenerse.

---

## Solución de Problemas

| Síntoma | Causa probable | Cómo corregirlo |
|---------|----------------|-----------------|
| `redirect_uri_mismatch` o `AADSTS50011` | La URI del IdP difiere de la enviada por QANode | Copie de nuevo la URI de QANode y regístrela como callback **Web** sin cambiar ningún carácter |
| La prueba OIDC falla antes de la pantalla de login | Emisor incorrecto o discovery inaccesible | Abra `issuer/.well-known/openid-configuration` y confirme que su `issuer` coincide exactamente con el valor configurado |
| `invalid_client` | Client ID o secreto incorrecto | Use el **valor** del secreto, compruebe su validez y confirme `client_secret_post/basic` |
| El correo del Super Admin no coincide | El IdP devolvió otro claim o alias | Ajuste el usuario/claim para devolver el correo registrado en QANode |
| SAML falla desde el botón de prueba del proveedor | Flujo iniciado por el IdP, sin estado de QANode | Inicie desde **Probar configuración** en QANode |
| SAML informa firma ausente o inválida | Solo se firmó la respuesta | Configure el IdP para firmar la aserción |
| No se puede procesar la respuesta SAML | Aserción cifrada o AuthnRequest firmada obligatoria | Deshabilite el cifrado y la exigencia de solicitud firmada |
| SCIM devuelve `A valid userName/email is required` | `userName` vacío o UPN técnico | Mapee un origen que contenga el correo real e inspeccione el valor de destino en el aprovisionamiento bajo demanda |
| Entra muestra `Active in the source system = False` | Usuario deshabilitado, eliminado o fuera del ámbito | Compruebe asignación, estado de la cuenta, filtros e `IsSoftDeleted` |
| Entra muestra `UnprocessableEntry` en la correspondencia | El atributo de origen está vacío | Corrija `userName`, mantenga precedencia 1 y vuelva a ejecutar bajo demanda |
| El usuario se eliminó del IdP pero sigue activo en QANode con JIT | JIT no sincroniza la desactivación | Use SCIM o espere la revalidación; migre a SCIM para controlar todo el ciclo de vida |
| No se creó el usuario SCIM | No está asignado, está fuera del ámbito o supera la licencia | Compruebe asignación, filtros, estado, logs de aprovisionamiento y plazas disponibles |
| El rol no cambió después de actualizar el IdP | Los roles no se sincronizan | Cambie el rol en QANode |
| SSO se desactivó después de guardar | Cambiaron datos críticos del proveedor | Pruebe de nuevo y reactive SSO |

---

## Checklist de Homologación

- [ ] El Super Admin puede probar y entrar mediante SSO.
- [ ] El acceso local de emergencia se probó o se deshabilitó conscientemente.
- [ ] Un usuario común nuevo recibe el rol por defecto esperado.
- [ ] Un usuario existente con el mismo correo se vincula sin duplicación.
- [ ] Cambiar el rol en QANode no es deshecho por el IdP.
- [ ] Un usuario sin asignación a la aplicación no puede autenticarse.
- [ ] La creación SCIM genera el registro correcto en QANode.
- [ ] Con SCIM, `active: false` desactiva al usuario.
- [ ] Reactivar al usuario en el IdP lo reactiva en QANode.
- [ ] El mapeo envía un correo real en `userName`.
- [ ] La política de MFA del IdP está aplicada.
- [ ] El intervalo de revalidación cumple la política de bajas de la empresa.
- [ ] Se definieron responsables y fechas de rotación para secretos, token SCIM y certificado SAML.

---

## Referencias de los Proveedores

- [Microsoft Entra — protocolo OIDC](https://learn.microsoft.com/entra/identity-platform/v2-protocols-oidc)
- [Microsoft Entra — mapeos de atributos SCIM](https://learn.microsoft.com/entra/identity/app-provisioning/customize-application-attributes)
- [Microsoft Entra — aprovisionamiento SCIM](https://learn.microsoft.com/entra/identity/app-provisioning/use-scim-to-provision-users-and-groups)
- [Google — OpenID Connect](https://developers.google.com/identity/openid-connect/openid-connect)
- [Google Workspace — aplicación SAML personalizada](https://support.google.com/a/answer/6087519)
- [Okta — referencia de campos SAML](https://help.okta.com/en-us/content/topics/apps/aiw-saml-reference.htm)
- [Keycloak — guía de administración](https://www.keycloak.org/docs/latest/server_admin/)
- [JumpCloud — campos del conector SAML](https://jumpcloud.com/support/sso-application-connector-fields)
- [Duo — SSO SAML genérico](https://duo.com/docs/sso-generic)
