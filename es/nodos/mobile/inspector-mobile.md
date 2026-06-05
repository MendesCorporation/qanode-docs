# Inspector Mobile

El **Inspector Mobile** es una herramienta visual integrada en QANode para grabar y construir pasos de automatización mobile de forma interactiva. En lugar de escribir XPath y selectores manualmente, interactúas directamente con la pantalla del dispositivo y los pasos se generan automáticamente.

---

## Cómo Acceder

El Inspector Mobile se abre desde el editor de flujos:

1. Agrega un nodo **Mobile Flow** al flujo
2. Configura la credencial mobile (dispositivo, plataforma, app)
3. Haz clic en el botón con el **icono de lupa** en el panel de propiedades del nodo

> En la **edición Desktop**, Appium se inicia automáticamente al conectar. En la edición web/servidor, asegúrate de que Appium esté en ejecución antes de abrir el inspector.

[Como Acessar](../../assets/images/mobile-como-acessar.mp4)

---

## Interfaz

El Inspector Mobile está dividido en tres áreas principales:

```
┌──────────────────────────────────────────────────────────────┐
│  [Conectar]  Modo: [Toque | Inspeccionar]  [Cerrar Sesión]   │
├─────────────────────────┬────────────────────────────────────┤
│                         │  Pasos Grabados                    │
│   Pantalla del          │  ─────────────────────────────     │
│   Dispositivo           │  1. tap — Botón Login              │
│   (captura en vivo)     │  2. type — email@...               │
│                         │  3. tap — Iniciar sesión           │
│                         │  ─────────────────────────────     │
│                         │  [Limpiar]       [Guardar Pasos]   │
├─────────────────────────┴────────────────────────────────────┤
│  Elemento Seleccionado: id=com.app:id/login_btn              │
└──────────────────────────────────────────────────────────────┘
```

### Área de Pantalla del Dispositivo

[Área de Pantalla del Dispositivo](../../assets/images/mobile-area-dispositivo.mp4)

Muestra la captura de pantalla actualizada del dispositivo en tiempo real. Puedes:
- **Tocar** elementos para generar pasos `tap`
- **Arrastrar** para generar pasos `swipe`
- **Clic derecho** para inspeccionar un elemento sin tocarlo (funciona en cualquier modo)

### Área de Pasos Grabados

Lista todos los pasos registrados durante la sesión. Cada acción muestra:
- Ícono y color de la acción, siguiendo el patrón visual de QANode
- Tipo de acción (tap, type, swipe, wait, extract, assert, etc.)
- Descripción del objetivo o texto
- Acciones para copiar o eliminar el paso individualmente

También es posible reorganizar los pasos arrastrándolos en la lista antes de guardarlos en el nodo.

### Panel del Elemento Seleccionado

Muestra la información del elemento tocado en modo **Inspeccionar**:
- resumen del objetivo seleccionado;
- texto, label, clase nativa y bounds;
- atributos relevantes del elemento;
- selectores detectados, con opción de marcar o desmarcar cuáles serán usados;
- cantidad de matches cuando esta información esté disponible.

La sección de selectores comienza colapsada para mantener la pantalla limpia. Ábrala cuando necesite revisar o ajustar la estrategia antes de agregar una acción.

---

## Modos de Operación

### Modo Toque (predeterminado)

[Modo Toque](../../assets/images/mobile-modo-toque.mp4)

En el modo **Toque**, cada clic en la pantalla del dispositivo:
1. Envía el toque al dispositivo via Appium
2. Graba un paso `tap` o `tap-coords` con los selectores del elemento tocado
3. Actualiza la captura de pantalla automáticamente tras la acción

Para **deslizar** (swipe): haz clic y arrastra en la pantalla del dispositivo — el gesto se ejecuta en el dispositivo y se graba como paso `swipe`.

Cuando un campo de texto está seleccionado, el inspector también permite grabar llenado con un diálogo propio, sin cambiar de modo. La opción **Limpiar primero** viene marcada por defecto.

### Modo Inspeccionar

[Modo Inspeccionar](../../assets/images/mobile-modo-inspecionar.mp4)

En el modo **Inspeccionar**, hacer clic en un elemento **no envía el toque al dispositivo** — solo muestra la información del elemento en el panel lateral. Usa este modo para:
- Explorar la jerarquía de elementos sin interactuar con la app
- Copiar selectores para uso manual
- Identificar el mejor selector antes de crear un paso

> **Consejo:** También puedes hacer **clic derecho** en cualquier lugar de la pantalla para inspeccionar un elemento sin cambiar al modo Inspeccionar.

---

## Grabando Pasos

### Paso de Toque (tap)

[Paso de Toque](../../assets/images/mobile-passo-toque.mp4)

1. Asegúrate de estar en el **Modo Toque**
2. Haz clic en el elemento deseado en la pantalla del dispositivo
3. El inspector detecta el elemento, extrae sus selectores y graba un paso `tap`
4. La pantalla se actualiza automáticamente

**Atajo de doble toque:** mantén `Shift` presionado y haz clic en el elemento para ejecutar y grabar `double-tap`.

### Paso de Escritura (type)

[Paso de Escritura](../../assets/images/mobile-passo-digitacao.mp4)

Después de tocar un campo de texto:
1. El inspector detecta que el campo está activo
2. Escribe el texto en tu teclado físico — el inspector captura las pulsaciones en tiempo real
3. El texto escrito se agrupa y se graba como paso `type`
4. Presionar **Backspace** borra caracteres y actualiza el paso

### Paso de Swipe

[Paso de Swipe](../../assets/images/mobile-passo-swipe.mp4)

1. En el **Modo Toque**, haz clic y **arrastra** en la pantalla del dispositivo
2. El gesto se ejecuta en el dispositivo
3. Se graba un paso `swipe` con las coordenadas de inicio y fin

---

## Retroalimentación Visual

Durante el uso del inspector, se muestran indicadores visuales sobre la pantalla del dispositivo:

| Indicador | Significado |
|-----------|-------------|
| Círculo verde pulsante | Punto de toque registrado |
| Línea verde con círculos | Gesto de swipe registrado |

Estos indicadores desaparecen automáticamente después de unos segundos.

---

## Acciones Avanzadas

Además de grabar toques y swipes, el Inspector Mobile ofrece botones para agregar acciones avanzadas directamente al flujo sin necesidad de configurar los campos manualmente.

### Acciones disponibles en el panel

**Con elemento seleccionado (modo Inspeccionar):**

| Acción | Descripción |
|--------|-------------|
| **Toque** | Crea un paso `tap` en el elemento seleccionado |
| **Rellenar** | Abre un diálogo para informar el texto y crea un paso `type` |
| **Doble Toque** | Crea un paso de doble toque en el elemento seleccionado |
| **Presionar y Mantener** | Crea un paso `long-press` en el elemento seleccionado |
| **Pinza para Dentro** | Crea un gesto de pinza sobre el elemento |
| **Zoom para Fuera** | Crea un gesto de apertura sobre el elemento |
| **Esperar Visible** | Crea una espera hasta que el elemento aparezca |
| **Esperar Habilitado** | Crea una espera hasta que el elemento esté habilitado |
| **Extracción** | Crea un paso `extract` usando selectores estables |
| **Validación** | Crea un paso de assert para el elemento seleccionado |

**Acciones de sistema (sin elemento seleccionado):**

| Acción | Descripción |
|--------|-------------|
| **Enviar archivo** | Envía un archivo al dispositivo y crea un paso `push-file` |
| **Descargar archivo** | Captura un archivo del dispositivo y crea un paso `pull-file` |
| **Accept Alert** | Agrega un paso `permission` que acepta la alerta del sistema actual |
| **Dismiss Alert** | Agrega un paso `permission` que descarta la alerta del sistema actual |
| **Grant Permission** | Solicita el nombre del permiso Android y agrega un paso `permission` para concederlo |
| **Revoke Permission** | Solicita el nombre del permiso Android y agrega un paso `permission` para revocarlo |

> Las acciones Grant/Revoke Permission son exclusivas para sesiones Android y abren un prompt solicitando el nombre del permiso, por ejemplo: `android.permission.CAMERA`.

### Enviar archivo durante la grabación

[Enviar archivo](../../assets/images/mobile-enviar-arquivo.mp4)

Use **Enviar archivo** cuando la app necesita recibir un archivo antes de abrir un selector nativo o continuar el flujo. El inspector envía el archivo al dispositivo durante la grabación y agrega el paso correspondiente al Mobile Flow.

En Android, el destino predeterminado suele ser la carpeta de downloads. En iOS, el inspector usa el container de la aplicación cuando está disponible y aplica un fallback compatible con Appium.

### Descargar archivo durante la grabación

[Descargar archivo](../../assets/images/mobile-baixar-arquivo.mp4)

Use **Descargar archivo** cuando la app generó un archivo y la prueba necesita capturarlo como `fileRef`.

El diálogo permite informar:

- ruta exacta en el dispositivo;
- carpeta y patrón de nombre para buscar el archivo más reciente;
- nombre de variable de salida;
- nombre del archivo guardado en QANode.

Use ruta exacta cuando la app muestra o define el nombre del archivo. Use carpeta/patrón cuando el nombre es dinámico.

---

## Guardando los Pasos

[Guardando los Pasos](../../assets/images/mobile-salvar-passos.mp4)

Al hacer clic en **"Guardar Pasos"**:
1. Todos los pasos grabados en la sesión se transfieren al nodo Mobile Flow
2. La sesión Appium permanece abierta (puedes continuar grabando si es necesario)
3. El inspector se cierra y los pasos quedan visibles en el panel del nodo

> Los pasos generados por el inspector pueden editarse manualmente en el nodo Mobile Flow después de guardar — agregar evidencias, ajustar selectores, configurar intentos, etc.

---

## Gestión de la Sesión

| Acción | Descripción |
|--------|-------------|
| **Conectar** | Inicia una nueva sesión Appium con la configuración de la credencial |
| **Cerrar Sesión** | Termina la sesión Appium en el dispositivo |
| **Limpiar Pasos** | Elimina todos los pasos grabados (la sesión continúa activa) |

> Cerrar el inspector sin hacer clic en "Guardar Pasos" descarta los pasos grabados en la sesión actual. Los pasos ya guardados anteriormente en el nodo no se ven afectados.

---

## Selectores Generados

El inspector genera automáticamente selectores para cada elemento tocado, priorizando en este orden:

1. **`accessibility_id`**, `id`, `resource-id`, `name` o `content-desc` — señales más fuertes
2. **`hint`**, `label` y `text` cuando pertenecen al propio elemento
3. **Contexto cercano** — label/texto asociado al campo, útil para inputs sin identificador
4. **`xpath`** con `@bounds` — selector basado en las coordenadas del elemento
5. **Coordenadas de fallback** — X/Y absolutos usados si todos los selectores fallan
6. **Clase genérica** — usada solo como último recurso

> Los selectores con `@bounds` y las coordenadas son fallback visual. En flujos automatizados, prefiera selectores por `accessibility_id`, `id`, `resource-id`, `label` o contexto cercano.

---

## Consejos

- Usa el **Modo Inspeccionar** (o clic derecho) para identificar el mejor selector antes de grabar, especialmente en pantallas con muchos elementos similares
- **Graba un flujo completo** de una vez — conectar, ejecutar el caso de uso en la app, guardar — para obtener pasos con contexto real
- Tras guardar, revisa los pasos en el nodo y agrega **capturas de evidencia** en los pasos críticos
- Para apps con animaciones largas, agrega pasos `wait` (`elementVisible`) después de los toques para asegurarte de que la pantalla cargó antes de la siguiente acción
- En la **edición Desktop**, la conexión con el dispositivo local es más rápida — úsala para desarrollo. Para pruebas en cloud (BrowserStack, etc.), configura la credencial SaaS antes de abrir el inspector
