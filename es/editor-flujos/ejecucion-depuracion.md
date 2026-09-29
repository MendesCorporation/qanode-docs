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

### Ejecución Visual de Flujos Web — Enterprise

Al iniciar una ejecución completa con el botón **Ejecutar** del canvas, desactivar **Headless** en al menos un nodo **Smart Web Flow**, **Web Flow** o **Smart Locators** activa el modo visual para esa ejecución.

El comportamiento depende de dónde se ejecuta QANode:

| Entorno | Comportamiento |
|---------|----------------|
| **Servidor con Live View** | Un único panel acompaña los nodos web de la ejecución. El nodo con **Headless** desactivado funciona como disparador; después, los demás nodos web de la misma ejecución también se muestran en el panel, aunque mantengan **Headless** activado. |
| **Desktop o entorno con ventana gráfica disponible** | La configuración continúa aplicándose por nodo. Los nodos con **Headless** desactivado pueden abrir una ventana visible; los nodos con **Headless** activado se ejecutan sin ventana. |

Live View pertenece a la ejecución iniciada desde el canvas, no a un único nodo. Por eso:

- el panel permanece abierto hasta que la ejecución finaliza con éxito, falla, se cancela o el usuario lo cierra;
- mientras el flujo ejecuta nodos que no usan navegador, como **HTTP Request**, **IF**, base de datos o **Wait**, no se produce una nueva imagen web y la última pantalla puede permanecer visible;
- si otro nodo web se ejecuta después, el mismo panel vuelve a acompañar el navegador sin cerrar y abrir una nueva ventana;
- mantener el panel abierto no significa necesariamente que el navegador anterior siga ejecutándose: el ciclo de vida del navegador todavía depende de la estrategia de sesión configurada en el nodo.

Este modo visual se activa únicamente para una ejecución interactiva iniciada por un usuario desde el canvas. Las **suites**, **programaciones**, **CI/CD**, **tokens de integración**, **API** y otros procesos externos no abren automáticamente una ventana ni Live View.

Al usar **Ejecutar** en el panel de propiedades, la visualización permanece limitada al nodo seleccionado.

### Estado de Ejecución

| Estado | Indicador | Descripción |
|--------|-----------|-------------|
| **Ejecutando** | 🔵 Azul/Pulsante | El nodo está siendo procesado |
| **Éxito** | 🟢 Verde | El nodo se ejecutó con éxito |
| **Fallo** | 🔴 Rojo | El nodo encontró un error |
| **Cancelado** | ⚪ Gris | La ejecución fue interrumpida manualmente |
| **Omitido** | ⚪ Gris | El nodo no fue ejecutado (rama inactiva o nodo marcado como omitido) |

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
| **Causa posible** | Sugerencia amigable mostrada en algunos pasos con fallo, con una corrección sugerida |
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

### Resultado mostrado en cada nodo

Al ejecutar el flujo completo desde el canvas, el panel de propiedades de cada nodo muestra el resultado de esa ejecución: éxito o fallo, logs y salidas disponibles. Antes de iniciar otra ejecución completa, QANode borra los resultados anteriores para no mezclar dos ejecuciones.

Si usa **Ejecutar nodo** después de una ejecución completa, se borran los resultados de la ejecución completa. Al probar varios nodos por separado, los resultados de esas pruebas permanecen visibles; volver a ejecutar un nodo reemplaza solo su resultado.

### Navegación en ejecuciones largas

En los detalles de una ejecución con muchos pasos, aparece un botón de acceso rápido al desplazarse por la lista: al bajar, permite ir al final; al subir, permite ir al principio. El botón permanece dentro de la ventana de detalles y desaparece al llegar a cualquiera de los extremos.

### Screenshots (Evidencias)

Para nodos web (Smart Web Flow, Web Flow y Smart Locators), las capturas de pantalla aparecen como miniaturas clicables. Haz clic para verlas a tamaño completo.

Las capturas regulares aparecen en la secuencia en que fueron realizadas, que también es el orden utilizado en el informe PDF. Las evidencias de accesibilidad, rendimiento y resumen se agrupan al final. Cuando las capturas `before` y `after` tienen el mismo instante de creación, QANode conserva primero el estado anterior y después el posterior, independientemente del nombre personalizado de la imagen.

### Archivos

Los archivos generados o capturados durante la ejecución aparecen en la sección **Archivos** del detalle de la ejecución. Al descargarlos, el navegador usa el nombre real del archivo cuando está disponible, en vez de un identificador interno de la ejecución.

Los archivos de prueba aislada creados en el editor son temporales y pueden limpiarse automáticamente cuando el usuario sale del flujo o por una rutina de limpieza.

---

## Depurando Fallos

### Identificando el Nodo con Problema

1. Busca nodos con borde **rojo** (🔴) en el canvas
2. Haz clic en el nodo con fallo
3. Verifica el **mensaje de error** y los **logs**

### Sugerencias Amigables de Fallo

Cuando un paso falla por un error conocido, el detalle de la ejecución puede mostrar una sección **Causa posible** con una sugerencia objetiva. Aparece solo en pasos con fallo y QANode muestra una sugerencia por vez para evitar mensajes contradictorios.

Las sugerencias cubren problemas comunes como sesión expirada, credenciales, expresiones rotas, archivos ausentes, objetivo ambiguo, campo oculto, deshabilitado o de solo lectura, clic bloqueado, timeout, assert que no coincide y efecto esperado que no apareció después de la acción.

Use la sugerencia como punto de partida y confirme en las evidencias, logs y capturas de pantalla antes de cambiar el flujo.

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
- Al usar **Ejecutar** en el panel de propiedades, la visualización muestra solo el nodo seleccionado
- Al ejecutar el flujo completo desde el canvas en un servidor, **Live View** acompaña toda la ejecución web en el mismo panel
- Las suites, programaciones, CI/CD y llamadas externas continúan ejecutándose sin abrir la visualización interactiva

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

Después de cada ejecución, QANode pone a disposición un **reporte PDF** con:

- Resumen de la ejecución (estado, duración, fecha)
- Detalles de cada paso (estado, logs, outputs, duración)
- Capturas de pantalla tomadas
- Errores encontrados

El reporte está disponible en la lista de ejecuciones del proyecto y puede descargarse o enviarse por correo electrónico. Según la plantilla activa, el PDF puede generarse automáticamente al finalizar la ejecución o solo cuando alguien solicita la descarga.

[Reporte de Ejecución](../../assets/images/relatorio-editor-fluxos.mp4)

---

## Próximos Pasos

- [Referencia de Nodos](../nodos/control-flujo/if.md) — Detalles completos de cada tipo de nodo
- [Expresiones](../expresiones/expresiones.md) — Domina el sistema de datos dinámicos
