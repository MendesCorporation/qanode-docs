# Variables

**Las variables** son valores reutilizables en escenarios de prueba. Permiten centralizar configuraciones como URLs, credenciales de prueba, indicadores y datos compartidos.

QANode tiene dos alcances principales:

- **Globales** — disponibles para cualquier escenario.
- **De proyecto** — disponibles para escenarios vinculados a un proyecto específico.

---

## Gestionar Variables

1. En el menú lateral, haga clic en **Variables**
2. La lista muestra las variables globales registradas

[Lista de variables](../../assets/images/variavel.mp4)
*Imagen: Pantalla de variables mostrando lista con nombre, tipo, valor y última actualización*

---

## Variables Globales y de Proyecto

### Variables Globales

Las variables globales deben usarse para valores compartidos por varios proyectos, como:

- URL base común;
- indicadores de ejecución;
- datos de prueba reutilizables;
- configuraciones generales.

Se crean desde el menú lateral **Variables**.

### Variables de Proyecto

Las variables de proyecto están dentro de la pestaña **Variables** de la página del proyecto. Use este alcance cuando el valor pertenece a un proyecto específico, como:

- URL de homologación de ese sistema;
- usuario de prueba de ese proyecto;
- identificadores o datos de prueba propios de ese equipo;
- configuraciones que no deben aparecer en la lista global.

Cuando un escenario pertenece a un proyecto, las variables del proyecto están disponibles con la misma sintaxis que las variables globales:

```
{{ variables.NOMBRE_DE_VARIABLE }}
```

Si una variable global y una variable de proyecto tienen el mismo nombre, evite depender de esa superposición. Prefiera nombres claros para cada alcance.

---

## Crear una Variable

1. Para una variable global, acceda a **Variables** en el menú lateral. Para una variable de proyecto, abra el proyecto y acceda a la pestaña **Variables**
2. Haga clic en **+ Nueva Variable**
3. Complete:

| Campo | Requerido | Descripción |
|-------|-----------|-------------|
| **Nombre** | Sí | Identificador de la variable (sin espacios) |
| **Tipo** | Sí | Tipo del valor |
| **Valor** | Sí | Valor de la variable |
| **Secreta** | No | Si está activado, el valor será cifrado |

4. Haga clic en **Guardar**

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

En el editor de flujo, el botón **Variables** del panel de propiedades muestra secciones contraídas para variables locales, globales y, cuando el escenario está dentro de un proyecto, variables de proyecto.

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

### Override por Pipeline — Enterprise

En la integración CI/CD de QANode Enterprise, una variable también puede sobrescribirse directamente desde la CLI o la API para una ejecución específica.

Ejemplo:

```bash
npx @qanode/cli run suite \
  --suite-id SUITE_ID \
  --var BASE_URL=https://preview.app \
  --wait
```

Este override:

- vale solo para la ejecución actual
- no cambia el valor persistido de la variable
- es útil para entornos temporales de build, homologación y preview

> Para más detalles, vea [Overrides por Ejecución](../ci-cd/overrides.md).

---

## Buenas Prácticas

### Nomenclatura

Use **UPPER_SNAKE_CASE** para variables globales y de proyecto:

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
- Use variables **globales** para valores compartidos por toda la instancia
- Use variables **de proyecto** para datos específicos de un proyecto
- El tipo **JSON** es útil para agrupar configuraciones relacionadas
