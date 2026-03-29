# Tokens de Integración

> **Disponible en:** QANode Enterprise

Los tokens de integración son credenciales dedicadas para automatización. Permiten que pipelines y scripts llamen a la CLI o a la API de QANode sin depender de una sesión de navegador.

---

## Dónde acceder

Vaya a:

**Configuración → Access Tokens**

Esta pantalla solo aparece para usuarios con los permisos requeridos.

---

## Permisos

| Permiso | Lo que permite |
|---------|----------------|
| `settings.integration_token` | Crear y revocar sus propios tokens |
| `settings.integration_token_all` | Ver todos los tokens, revocar tokens de otras personas y controlar la política global de expiración |

En la configuración predeterminada del producto:

- **Super Admin** y **Admin** suelen tener acceso completo
- **Architect** puede tener acceso para crear sus propios tokens según el rol configurado

> **Importante:** Un token hereda la capacidad operativa del usuario que lo utiliza. En otras palabras, tener un token no concede acceso adicional; solo provee autenticación no interactiva.

---

## Crear un token

1. Vaya a **Configuración → Access Tokens**
2. Haga clic en **Generar Token**
3. Defina:
   - el **nombre** del token
   - el **plazo de expiración**, cuando la política global permita elección individual
4. Copie el valor generado
5. Guarde ese valor en un **secret** del pipeline

El token completo se muestra solo en el momento de la creación. Después de eso, la interfaz conserva solo información de referencia y auditoría.

---

## Prefijo del token

Los tokens de integración de QANode usan el prefijo:

```bash
qnt_
```

Esto facilita su identificación en logs, variables de entorno y almacenes de secretos.

---

## Política global de expiración

Los usuarios con `settings.integration_token_all` pueden configurar la política global en:

**Configuración → Access Tokens → Política Global de Expiración de Tokens**

Las opciones disponibles son:

- **Cada usuario elige** — cada persona decide si el token expira y en cuánto tiempo
- **Expiración fija** — todo token nuevo expira después del mismo plazo definido por administración

Cuando la política global es fija, el usuario deja de elegir una expiración individual en el modal de creación.

---

## Revocación

Revocar un token impide de inmediato su uso en nuevos comandos CLI o llamadas API.

Puede revocar:

- sus propios tokens con `settings.integration_token`
- cualquier token de la instancia con `settings.integration_token_all`

---

## Buenas prácticas de seguridad

- Guarde `QANODE_TOKEN` en los **Secrets** de su proveedor de CI/CD
- Nunca escriba tokens en archivos versionados
- Prefiera un token por pipeline o por sistema integrado
- Revoque tokens que ya no se usen
- Use expiración cuando su empresa requiera rotación periódica

---

## Ejemplo con secret

```bash
export QANODE_URL=https://qanode.empresa.com
export QANODE_TOKEN=qnt_xxxxx
```

En GitHub Actions o Azure DevOps, este valor normalmente se guarda como secret del job y no como texto plano en el repositorio.

---

## Auditoría

QANode registra eventos relacionados con tokens de integración, como:

- creación
- revocación
- actualización de la política global de expiración

Esto ayuda a rastrear quién creó o revocó cada clave operativa.

---

## Próximos Pasos

- [CLI y API del CI/CD](./cli-api.md) — Vea cómo autenticar, disparar ejecuciones y consultar resultados
