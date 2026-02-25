# Variables

**Las variables** son valores globales reutilizables en cualquier escenario de prueba. Permiten centralizar configuraciones como URLs, credenciales de prueba, indicadores y datos compartidos.

---

## Gestionar Variables

1. En el menú lateral, haga clic en **Variables**
2. La lista muestra todas las variables registradas

[Lista de variables](../../assets/images/variavel.mp4)
*Imagen: Pantalla de variables mostrando lista con nombre, tipo, valor y última actualización*

---

## Crear una Variable

1. Haga clic en **+ Nueva Variable**
2. Complete:

| Campo | Requerido | Descripción |
|-------|-----------|-------------|
| **Nombre** | Sí | Identificador de la variable (sin espacios) |
| **Tipo** | Sí | Tipo del valor |
| **Valor** | Sí | Valor de la variable |
| **Secreta** | No | Si está activado, el valor será cifrado |

3. Haga clic en **Guardar**

---

## Tipos de Variables

| Tipo | Descripción | Ejemplo |
|------|-------------|---------|
| **String** | Texto | `https://api.exemplo.com` |
| **Number** | Número | `30000` |
| **Boolean** | Verdadero/Falso | `true` |
| **JSON** | Objeto o arreglo | `{ "timeout": 5000, "retries": 3 }` |

---

## Variables Secretas

Las variables marcadas como **secretas** se cifran en la base de datos. Son ideales para:

- Contraseñas de prueba
- Tokens de API
- Claves de acceso

Características:
- El valor está **enmascarado** en la interfaz (mostrado como `••••••`)
- El valor está **cifrado** en la base de datos
- El valor real es **accesible** durante la ejecución de los flujos

> **Nota:** Para credenciales de conexión (base de datos, SSH, API), prefiera usar [Credenciales](../credenciales/credenciales.md) en lugar de variables secretas.

---

## Usar Variables en los Flujos

Acceda a las variables usando la sintaxis de expresiones:

```
{{ variables.NOMBRE_DE_VARIABLE }}
```

### Ejemplos de Uso

**URL de API:**
```
URL: {{ variables.API_URL }}/api/users
```

**Timeout:**
```
Timeout: {{ variables.DEFAULT_TIMEOUT }}
```

**Credenciales de prueba:**
```
Email: {{ variables.TEST_EMAIL }}
Contraseña: {{ variables.TEST_PASSWORD }}
```

**Configuración JSON:**
```
Retries: {{ variables.CONFIG.retries }}
Timeout: {{ variables.CONFIG.timeout }}
```

---

## Sobrescritura en Tiempo de Ejecución

El nodo **Set Variable** puede sobrescribir variables durante la ejecución:

```
[Set Variable: API_URL = https://staging.exemplo.com]
    │
    ▼
[HTTP Request: GET {{ variables.API_URL }}/api/health]
```

La sobrescritura es temporal — solo afecta la ejecución actual. El valor original en la base de datos no se modifica.

---

## Buenas Prácticas

### Nomenclatura

Use **UPPER_SNAKE_CASE** para variables globales:

| Correcto | Incorrecto |
|----------|------------|
| `API_URL` | `apiUrl` |
| `TEST_EMAIL` | `email` |
| `DEFAULT_TIMEOUT` | `timeout` |
| `MAX_RETRIES` | `retries` |

### Organización por Ambiente

Cree variables con prefijo de ambiente para facilitar el cambio:

```
PROD_API_URL = https://api.exemplo.com
STAGING_API_URL = https://staging.exemplo.com
DEV_API_URL = http://localhost:3000
```

### Cuándo Usar Variables vs. Credenciales

| Escenario | Use |
|-----------|-----|
| URL base de API | Variable |
| Token de autenticación | Credencial HTTP |
| Datos de conexión a base de datos | Credencial de base de datos |
| Contraseña de prueba genérica | Variable secreta |
| Indicador de configuración | Variable |
| Datos de conexión SSH | Credencial SSH |

---

## Consejos

- **Centralice las URLs** en variables para facilitar el cambio de ambiente
- **Use variables secretas** para datos sensibles
- **Prefiera credenciales** para datos de conexión estructurados
- Las variables son **globales** — accesibles en cualquier flujo
- El tipo **JSON** es útil para agrupar configuraciones relacionadas
