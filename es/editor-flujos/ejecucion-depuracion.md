# Ejecución y Depuración

Aprende cómo ejecutar tus flujos, interpretar resultados y diagnosticar fallos.

---
 
## Ejecutando un Flujo

### Ejecución Completa

Para ejecutar todo el flujo:

1. Asegúrate de que el flujo esté guardado
2. Haz clic en el botón **Ejecutar** (▶️) en la barra superior
3. Espera la finalización — los nodos mostrarán indicadores de estado en tiempo real

[Flujo en ejecución](../../assets/images/execucao-completa-fluxos.mp4)
*Imagen: Canvas mostrando nodos con indicadores de estado durante la ejecución*

### Estado de Ejecución

| Estado | Indicador | Descripción |
|--------|-----------|-------------|
| **Ejecutando** | 🔵 Azul/Pulsante | El nodo está siendo procesado |
| **Éxito** | 🟢 Verde | El nodo se ejecutó con éxito |
| **Fallo** | 🔴 Rojo | El nodo encontró un error |
| **Cancelado** | ⚪ Gris | La ejecución fue interrumpida manualmente |
| **Omitido** | ⚪ Gris | El nodo no fue ejecutado (rama inactiva) |

---

## Cancelando Ejecuciones

Las ejecuciones en curso pueden ser canceladas por usuarios con el permiso `run.cancel`.

Dónde cancelar:

- **Editor del escenario**: durante la ejecución, el botón de ejecutar cambia a un botón de parada.
- **Lista de ejecuciones**: usa el menú de acciones de la ejecución en curso.
- **Ejecución de suite**: cancelar la ejecución principal también cancela los escenarios pendientes o en ejecución de esa suite.

Al cancelar, el estado final queda como **Cancelado**. Las evidencias y logs ya generados hasta ese punto permanecen disponibles para consulta.

La cancelación ocurre en puntos seguros de la ejecución. Los nodos web, Smart Web, mobile, API, base de datos, SSH, carga y JavaScript personalizado verifican la solicitud de cancelación durante el procesamiento. En nodos con navegador o sesión mobile, QANode también intenta cerrar recursos abiertos para evitar sesiones atascadas.

> Una acción externa muy larga puede terminar el tramo actual antes de reconocer la cancelación. En esos casos, QANode cancela tan pronto como la ejecución vuelve a un punto seguro.

---

## Analizando Resultados

Después de la ejecución, haz clic en cualquier nodo para ver sus resultados en el panel de propiedades:

### Pestaña de Resultados

| Sección | Descripción |
|---------|-------------|
| **Estado** | Éxito o fallo, con duración |
| **Logs** | Mensajes registrados durante la ejecución del nodo |
| **Outputs** | Datos producidos por el nodo (JSON navegable) |
| **Error** | Mensaje de error detallado (cuando corresponda) |
| **Archivos y Screenshots** | Artefactos generados, downloads, uploads capturados y evidencias |

[Pestaña de Resultados](../../assets/images/analise-resultados-editor-fluxos.mp4)

### Outputs

Los outputs de cada nodo quedan accesibles para nodos siguientes mediante expresiones. Por ejemplo, después de ejecutar un **HTTP Request**, los outputs serán:

```json
{
  "status": 200,
  "body": {
    "id": 1,
    "name": "João",
    "email": "joao@exemplo.com"
  }
}
```

Luego puedes acceder a estos datos en nodos siguientes:
```
{{ steps["http-request"].outputs.body.name }}  →  "João"
{{ steps["http-request"].outputs.status }}     →  200
```

Cuando el output es un archivo, el valor principal es un `fileRef`:

```
{{ steps["http-request"].outputs.fileRef }}
{{ steps["file-generate"].outputs.fileRef }}
```

En el panel de variables, los archivos muestran nombre, MIME type y tamaño en los detalles. Los campos internos como el camino en el storage quedan ocultos en la visualización principal.

### Screenshots (Evidencias)

Para nodos web (Smart Web Flow, Web Flow y Smart Locators), las capturas de pantalla aparecen como miniaturas clicables. Haz clic para verlas a tamaño completo.

### Archivos

Los archivos generados o capturados durante la ejecución aparecen en la sección **Archivos** del detalle de la ejecución. Al descargarlos, el navegador usa el nombre real del archivo cuando está disponible, en vez de un identificador interno de la ejecución.

Los archivos de prueba aislada creados en el editor son temporales y pueden limpiarse automáticamente cuando el usuario sale del flujo o por una rutina de limpieza.

---

## Depurando Fallos

### Identificando el Nodo con Problema

1. Busca nodos con borde **rojo** (🔴) en el canvas
2. Haz clic en el nodo con fallo
3. Verifica el **mensaje de error** y los **logs**

### Mensajes de Error Comunes

#### Web Flow / Smart Locators

| Error | Causa | Solución |
|-------|-------|----------|
| `Timeout waiting for selector` | Elemento no encontrado en la página | Verifica el selector/localizador; aumenta el timeout |
| `Element not visible` | El elemento existe pero no está visible | Agrega un paso `wait` o `scroll` antes |
| `Navigation timeout` | La página no cargó a tiempo | Verifica la URL y la conectividad |
| `Element is not attached to DOM` | Elemento eliminado antes de la acción | Agrega un `wait` para estabilidad |

#### HTTP Request

| Error | Causa | Solución |
|-------|-------|----------|
| `ECONNREFUSED` | El servidor no está accesible | Verifica la URL y si el servidor está corriendo |
| `401 Unauthorized` | Autenticación inválida | Verifica el token/credenciales |
| `ETIMEOUT` | La solicitud superó el tiempo límite | Aumenta el timeout o verifica el servidor |

#### Base de Datos

| Error | Causa | Solución |
|-------|-------|----------|
| `Connection refused` | Base de datos no accesible | Verifica host, puerto y firewall |
| `Authentication failed` | Credenciales inválidas | Verifica usuario y contraseña |
| `Relation does not exist` | Tabla no encontrada | Verifica el nombre de la tabla y la base de datos |

#### SSH

| Error | Causa | Solución |
|-------|-------|----------|
| `Authentication failed` | Credenciales SSH inválidas | Verifica usuario, contraseña o clave privada |
| `Connection timeout` | Host no accesible | Verifica host, puerto y red |

### Usando Logs para Diagnóstico

Los **logs** de cada nodo proporcionan detalles sobre cada paso ejecutado. Para nodos web, los logs muestran:

```
Navigated to: https://exemplo.com/login
Filled: "usuario@exemplo.com" on [getByLabel("E-mail")]
Clicked: [getByRole("button", { name: "Entrar" })]
Assert passed: textContains "Bem-vindo" — true
```

Si un paso falló, el log mostrará exactamente cuál paso y por qué:

```
Navigated to: https://exemplo.com/login
Filled: "usuario@exemplo.com" on [getByLabel("E-mail")]
ERROR: Click failed after 3 attempts: [getByRole("button", { name: "Login" })] — element not found
```

---

## Consejos de Depuración

### 1. Desactiva el Modo Headless

Para pruebas web, desactiva el **modo headless** en el nodo para ver el navegador en acción:
- En el nodo Smart Web Flow, Web Flow o Smart Locators, desmarca **Headless**
- Ejecuta el flujo y observa el navegador

### 2. Agrega Pasos de Wait

Si los elementos aparecen con retraso, agrega pasos de **wait** antes de interactuar:
- `wait` con modo `visible` espera que el elemento aparezca
- `wait` con modo `networkIdle` espera que todas las solicitudes de red terminen

### 3. Captura Screenshots para Diagnóstico

Activa las capturas de pantalla en modo **antes** para ver el estado de la página antes de cada acción. Esto ayuda a identificar si la página estaba en el estado esperado.

### 4. Verifica las Expresiones

Si un nodo falla con un valor inesperado, agrega un nodo **Log** antes de él para inspeccionar los valores:

```
Valor del token: {{ steps.login.outputs.body.token }}
Estado: {{ steps["http-request"].outputs.status }}
```

### 5. Usa el Toggle "Continuar en Fallo"

Para diagnosticar múltiples fallos a la vez, activa **Continuar en Fallo** en los nodos de verificación. Esto permite que el flujo continúe y puedas ver todos los fallos en una única ejecución.

---

## Abrir un Defecto desde una Ejecución Fallida — Enterprise

Cuando una ejecución termina con **fallo**, el botón **Abrir Defecto** queda disponible en el detalle de la ejecución. Al hacer clic, se abre un modal para completar la información del defecto — título, descripción, severidad, prioridad y demás campos definidos por el flujo de trabajo.

El defecto creado queda automáticamente vinculado a la ejecución, al escenario y al paso que originó el fallo.

> **Permiso requerido:** `bug.create`

[Abrir un Defecto](../../assets/images/abrir-defeito-editor-fluxos.mp4)

### Defectos relacionados

El detalle de la ejecución también muestra la sección **Defectos Relacionados** con la lista de todos los defectos vinculados a esa ejecución — bug key, título, estado y severidad — con enlace directo a cada defecto.

---

## Reporte de Ejecución

Después de cada ejecución, QANode genera automáticamente un **reporte PDF** con:

- Resumen de la ejecución (estado, duración, fecha)
- Detalles de cada paso (estado, logs, outputs, duración)
- Capturas de pantalla tomadas
- Errores encontrados

El reporte está disponible en la lista de ejecuciones del proyecto y puede descargarse o enviarse por correo electrónico.

[Reporte de Ejecución](../../assets/images/relatorio-editor-fluxos.mp4)

---

## Próximos Pasos

- [Referencia de Nodos](../nodos/control-flujo/if.md) — Detalles completos de cada tipo de nodo
- [Expresiones](../expresiones/expresiones.md) — Domina el sistema de datos dinámicos
