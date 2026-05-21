# Nodo Web Flow

El nodo **Web Flow** permite automatizar interacciones con páginas web usando **selectores CSS, XPath, data-testid y texto**. Es ideal cuando necesitas selectores tradicionales y control detallado sobre la localización de elementos.

> **Consejo:** Para flujos web nuevos grabados por la extensión, prefiera [Smart Web Flow](smart-web-flow.md). Si prefiere localizadores semánticos basados en accesibilidad (getByRole, getByLabel, etc.) en flujos legados, use el nodo [Smart Locators](smart-locators.md).

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `web-flow` |
| **Categoría** | Web |
| **Color** | Azul (#3b82f6) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Configuración General

| Campo | Tipo | Predeterminado | Descripción |
|-------|------|----------------|-------------|
| **Modo de Sesión** | `new` / `reuse` | `new` | Nueva sesión o reutilizar una existente |
| **Session ID** | `string` | — | ID de la sesión a reutilizar (cuando `reuse`) |
| **Headless** | `boolean` | `true` | Ejecutar sin interfaz gráfica visible |
| **Estrategia de Storage** | `inMemory` / `persisted` | `inMemory` | Persistir cookies/localStorage entre ejecuciones |
| **Clave de Storage** | `string` | — | Identificador del storage persistido |
| **Navegador** | `chromium` / `firefox` / `webkit` | `chromium` | Navegador a usar en la ejecución |
| **Dispositivo** | `desktop` / `mobile` | `desktop` | Modo de emulación |
| **Viewport** | preset o `custom` | `1920x1080` | Tamaño de ventana (modo Desktop) |
| **Ancho / Alto** | `number` | — | Dimensiones personalizadas (cuando `custom`) |
| **Modelo de Dispositivo** | string | `iPhone 14` | Dispositivo a emular (modo Mobile) |
| **Escaneo de accesibilidad** | `boolean` | `false` | Ejecuta diagnóstico integrado de accesibilidad |
| **Auditoría de rendimiento** | `boolean` | `false` | Recoge métricas de carga y red durante la ejecución |
| **Tiempo máximo de carga (ms)** | `number` | `4000` | Threshold para fallar cuando la carga de la página supera el límite |
| **Tiempo máximo de respuesta de API (ms)** | `number` | `1500` | Threshold para APIs lentas capturadas en la sesión |
| **Fallar en errores de solicitud** | `boolean` | `true` | Falla el nodo cuando las APIs monitoreadas devuelven errores HTTP/de red |
| **Self Healing** | `boolean` | `false` | Intenta recuperar pasos cuando el elemento original cambia y los selectores ya no encuentran el objetivo |

### Navegador

| Valor | Navegador |
|-------|-----------|
| `chromium` | Chrome / Chromium (predeterminado) |
| `firefox` | Firefox |
| `webkit` | Safari / WebKit |

> Firefox y WebKit se descargan automáticamente en el primer uso si no están instalados.

### Dispositivo — Desktop

Presets de viewport disponibles:

| Preset | Resolución |
|--------|------------|
| Full HD (predeterminado) | 1920 × 1080 |
| 2K | 2560 × 1440 |
| 4K | 3840 × 2160 |
| Laptop | 1440 × 900 |
| HD | 1366 × 768 |
| WXGA | 1280 × 720 |
| XGA | 1024 × 768 |
| Personalizado | ancho y alto libres |

### Dispositivo — Mobile

Usa perfiles reales de Playwright (viewport, user agent, escala de píxeles, touch). Modelos disponibles:

**iOS:** iPhone SE, iPhone 12, iPhone 13, iPhone 14, iPhone 14 Pro, iPhone 15, iPhone 15 Pro, iPad Mini, iPad (gen 9), iPad Pro 11

**Android:** Pixel 5, Pixel 7, Galaxy S8, Galaxy S9+, Galaxy Tab S4

> En modo Mobile con Firefox, las opciones `isMobile` y `hasTouch` se eliminan automáticamente (no son compatibles con Firefox en Playwright). El viewport y el user agent del dispositivo se aplican igualmente.

### Modo de Sesión

- **Nueva Sesión (`new`)**: Abre un nuevo navegador para cada ejecución. Ideal para pruebas aisladas.
- **Reutilizar Sesión (`reuse`)**: Usa una sesión de navegador ya abierta por otro nodo web compatible. Útil para dividir pruebas largas en múltiples nodos manteniendo el mismo navegador y cookies.

### Estrategia de Storage

- **En Memoria (`inMemory`)**: Las cookies y el localStorage se descartan al final de la ejecución.
- **Persistido (`persisted`)**: Las cookies y el localStorage se guardan con una clave y pueden reutilizarse en ejecuciones futuras. Ideal para mantener sesiones de inicio de sesión entre ejecuciones.

### Auditoría de Rendimiento

Cuando **Auditoría de rendimiento** está habilitada, Web Flow monitorea la sesión activa de Playwright y genera evidencias visuales de rendimiento por pantalla.

La función recoge:

- **Page Load**
- **FCP** (*First Contentful Paint*)
- **LCP** (*Largest Contentful Paint*)
- cantidad total de solicitudes
- cantidad de solicitudes de API
- errores de solicitud/API
- APIs lentas por pantalla

Además del resumen global, QANode genera **checkpoints por pantalla** con:

- nombre de la pantalla
- URL
- paso que originó la navegación
- gráfico de las APIs más lentas
- gráfico separado de **errores de API** cuando existan en esa pantalla

### Outputs de Rendimiento

Cuando la auditoría está activa, el nodo expone:

- `performancePassed`
- `performanceRequestCount`
- `performanceApiRequestCount`
- `performanceErrorCount`
- `performanceSlowRequestCount`
- `performance`

El objeto `performance` contiene un resumen estructurado de la ejecución, incluidos los checkpoints por pantalla.

### Self Healing

Cuando **Self Healing** está habilitado, Web Flow puede recuperar automáticamente un paso cuando el objetivo esperado y sus `selectorStrategies` dejan de funcionar.

El mecanismo considera señales como:

- texto visible y nombre accesible
- `label`, `placeholder`, `title`, `alt`
- `data-testid`, `id` y `name`
- `role` del elemento
- contexto estructural de la página, como contenedor, formulario, heading y región visual

Esto ayuda en cambios comunes de interfaz, por ejemplo:

- diferencias entre mayúsculas y minúsculas
- texto con o sin acentos
- pequeños cambios de etiqueta
- cambios simples de sinónimos, como `Siguiente` → `Continuar`

Self healing es conservador y solo se aplica cuando existe un candidato fuerte y claramente mejor que las alternativas, reduciendo falsos positivos en pantallas con varios botones parecidos.

Cuando ocurre una recuperación, los detalles de la ejecución muestran:

- el paso afectado
- la confianza de la recuperación
- el selector utilizado
- una acción **Aplicar al Flujo** para promover el selector recuperado al flujo original

---

## Pasos (Web Steps)

El nodo Web Flow ejecuta una secuencia de **pasos** configurables. Cada paso representa una acción en el navegador.

### Acciones Disponibles

| Acción | Color | Descripción |
|--------|-------|-------------|
| [navigate](#navigate) | Azul | Navegar a una URL |
| [switchPage](#switchpage) | Morado | Cambiar a otra página o pestaña |
| [click](#click) | Amarillo | Hacer clic en un elemento |
| [type](#type) | Verde | Escribir texto en un campo |
| [drag](#drag) | Rosa | Arrastrar y soltar |
| [wait](#wait) | Morado | Esperar una condición o tiempo |
| [assert](#assert) | Rojo | Verificar una condición en la página |
| [extract](#extract) | Cian | Extraer datos de un elemento |
| [extractList](#extractlist) | Esmeralda | Extraer lista de elementos repetidos |
| [hover](#hover) | Cian | Pasar el mouse sobre un elemento |
| [scroll](#scroll) | Amarillo | Desplazar la página |
| [refresh](#refresh) | Naranja | Recargar la página |
| [select](#select) | Morado | Seleccionar una opción de un dropdown |
| [press-key](#press-key) | Verde | Presionar una tecla |
| [clock](#clock) | Verde azulado | Controlar el reloj del navegador |
| [frame](#frame) | Gris | Alternar entre frames/iframes |

---

## Estrategias de Selector

Web Flow usa **estrategias de selector** para localizar elementos en la página. Cada paso puede tener múltiples estrategias ordenadas por prioridad — si la primera no encuentra el elemento, se intenta con la siguiente.

| Estrategia | Descripción | Ejemplo |
|------------|-------------|---------|
| `css` | Selector CSS | `#login-button`, `.submit-btn`, `[data-testid="login"]` |
| `dataTestId` | Atributo data-testid | `login-button` |
| `role` | ARIA role | `button` |
| `text` | Texto visible | `Ingresar` |
| `xpath` | Expresión XPath | `//button[@type="submit"]` |

**Prioridad recomendada:**
1. `dataTestId` — Más estable, no cambia con el diseño
2. `css` con ID — Generalmente único en la página
3. `role` — Basado en accesibilidad
4. `text` — Puede cambiar con traducciones
5. `xpath` — Último recurso, más frágil

---

## Detalles de las Acciones

### navigate

Navega a una URL específica.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **URL** | `string` | URL de destino (soporta expresiones `{{ }}`) |

**Ejemplo:**
```
URL: https://misitio.com/login
URL: {{ variables.BASE_URL }}/dashboard
```

---

### switchPage

Cambia la página activa cuando la automatización necesita continuar en otra pestaña o ventana abierta por el navegador.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Modo** | `string` | Cómo elegir la pestaña: última abierta, siguiente, anterior o por coincidencia |
| **URL contiene** | `string` | Fragmento opcional de la URL para localizar la pestaña |
| **Título contiene** | `string` | Fragmento opcional del título de la página |
| **Esperar Hasta** | `string` | Evento de carga esperado después del cambio |
| **Timeout (ms)** | `number` | Tiempo máximo para encontrar/cargar la pestaña |

**Modos:**

| Modo | Descripción |
|------|-------------|
| `last` | Usa la última pestaña abierta en el contexto del navegador |
| `next` | Usa la siguiente pestaña en relación con la actual |
| `previous` | Usa la pestaña anterior en relación con la actual |
| `match` | Busca una pestaña por fragmento de URL y/o título |

Cuando la extensión detecta un clic que abre nueva pestaña, puede grabar este paso automáticamente justo después del click. Esto evita que los próximos pasos intenten ejecutarse en la pestaña anterior.

---

### click

Hace clic en un elemento de la página.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Selectores** | `array` | Estrategias de selector |
| **Intentos** | `number` | Número de intentos (predeterminado: 3) |

El click intenta localizar el elemento usando cada estrategia en orden. Si falla, espera y vuelve a intentar (hasta el número de intentos configurado).

---

### type

Escribe texto en un campo de entrada.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Selectores** | `array` | Estrategias de selector |
| **Texto** | `string` | Texto a escribir (soporta `{{ }}`) |
| **Limpiar Primero** | `boolean` | Limpia el campo antes de escribir |

> **Nota:** La acción `type` usa `fill()` de Playwright, que reemplaza el valor del campo de forma instantánea. Para simular la escritura carácter por carácter, usa el nodo [Smart Locators](smart-locators.md) con la acción `type` (pressSequentially).

---

### drag

Arrastra un elemento de origen y lo suelta sobre otro elemento o en coordenadas específicas de la página.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Selectores de Origen** | `array` | Estrategias CSS/XPath del elemento que será arrastrado |
| **Selectores de Destino** | `array` | Estrategias CSS/XPath del elemento donde será soltado |
| **Destino X / Y** | `number` | Coordenadas absolutas que se muestran cuando no hay selector de destino |
| **Esperar Después (ms)** | `number` | Tiempo de espera después de soltar |

Web Flow intenta ejecutar el drag con el método nativo de Playwright (`dragTo`) cuando origen y destino son identificables. Si el destino es un área libre, como un canvas o editor visual, usa movimiento real del mouse hasta `Destino X / Y`.

Cuando el paso viene de la extensión, el JSON puede incluir metadatos internos del gesto. Ayudan al ejecutor a reproducir el movimiento con precisión, pero no aparecen como campos principales en el panel.

**Ejemplos de uso:**

- Mover cards en un Kanban
- Arrastrar ítems a una lista
- Soltar un nodo en un área libre del canvas
- Probar ordenación o upload cuando la UI depende de drag/drop

**Buenas prácticas:**

- Prefiere `data-testid`, `data-qa` o `id` estables para origen y destino
- Usa coordenadas solo cuando el destino no tenga un elemento identificable
- Captura screenshots `before` y `after` para facilitar la revisión visual
- Si la página tiene elementos repetidos, prefiere selectores más específicos para evitar ambigüedad

---

### wait

Espera una condición antes de continuar.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Modo** | `string` | Tipo de espera |
| **Timeout (ms)** | `number` | Tiempo máximo de espera |
| **Selectores** | `array` | Selectores (modo `selectorVisible`) |

**Modos:**

| Modo | Descripción |
|------|-------------|
| `networkIdle` | Espera a que terminen todas las solicitudes de red |
| `selectorVisible` | Espera a que un elemento sea visible |
| `timeout` | Espera un tiempo fijo (ms) |

---

### assert

Verifica una condición en la página. Si la condición falla, el paso se marca como fallido.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Nombre** | `string` | Identificador de la aserción |
| **Modo** | `string` | Tipo de verificación |
| **Selectores** | `array` | Selectores del elemento |
| **Texto Esperado** | `string` | Valor esperado |
| **Case Sensitive** | `boolean` | Sensible a mayúsculas |
| **Continuar en Fallo** | `boolean` | No fallar el nodo completo |

**Modos:**

| Modo | Descripción |
|------|-------------|
| `elementExists` | Verifica si el elemento existe en la página |
| `textContains` | Verifica si el texto del elemento contiene el valor esperado |
| `textEquals` | Verifica si el texto del elemento es exactamente el valor esperado |

Los resultados de las aserciones están disponibles en los outputs:
```
{{ steps["web-flow"].outputs.asserts.nombreAsercion }}  →  true o false
```

---

### extract

Extrae datos de un elemento de la página.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Nombre** | `string` | Nombre de la extracción (clave en el output) |
| **Selectores** | `array` | Selectores del elemento |
| **Atributo** | `string` | Qué extraer |

**Atributos:**

| Atributo | Descripción |
|----------|-------------|
| `text` | Texto visible del elemento |
| `innerHTML` | HTML interno |
| `href` | URL de enlaces |
| `src` | URL de imágenes |
| `value` | Valor de campos de formulario |

Los datos extraídos están disponibles en los outputs:
```
{{ steps["web-flow"].outputs.extracts.nombreExtraccion }}
```

---

### extractList

Itera sobre una lista de elementos repetidos (filas de tabla, cards, ítems de lista) y extrae campos nombrados de cada uno, produciendo un array de objetos como output.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Nombre** | `string` | Nombre de la extracción (clave en el array de output) |
| **Selector de Contenedor** | `array` | Estrategias de selector CSS/XPath para encontrar el contenedor de la lista |
| **Selector de Ítem** | `string` | Selector CSS relativo al contenedor que selecciona cada ítem repetido |
| **Campos** | `array` | Lista de campos a extraer de cada ítem |

**Configuración de Campos:**

| Sub-campo | Tipo | Descripción |
|-----------|------|-------------|
| **Nombre del Campo** | `string` | Clave en el objeto de output |
| **Selector** | `string` | Selector CSS relativo al ítem |
| **Atributo** | `string` | Qué extraer: `text` (predeterminado), `innerHTML`, `href`, `src`, `value` |

**Ejemplo de output:**
```json
{
  "extracts": {
    "productos": [
      { "nombre": "Ítem A", "precio": "$10.00" },
      { "nombre": "Ítem B", "precio": "$25.00" }
    ]
  }
}
```

> **Grabación con la extensión:** Usa **Ctrl+Shift+E** en el Chrome Recorder para grabar una extracción de lista en dos pasos: primero haz clic en el ítem repetido, luego haz clic en los campos a extraer dentro de él.

---

### hover

Pasa el mouse sobre un elemento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Selectores** | `array` | Selectores del elemento |
| **Intentos** | `number` | Número de intentos |
| **Espera Después (ms)** | `number` | Tiempo de espera tras el hover |

---

### scroll

Desplaza la página o hasta un elemento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Modo** | `string` | Tipo de desplazamiento |
| **Píxeles** | `number` | Cantidad de píxeles (modo `by`) |
| **Espera Después (ms)** | `number` | Tiempo de espera tras desplazar |
| **Selectores** | `array` | Selectores (modo `toSelector`) |

**Modos:**

| Modo | Descripción |
|------|-------------|
| `toSelector` | Desplaza hasta que el elemento sea visible |
| `by` | Desplaza una cantidad fija de píxeles |
| `toBottom` | Desplaza hasta el final de la página |

---

### refresh

Recarga la página actual.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Esperar Hasta** | `string` | Evento de carga |
| **Espera Después (ms)** | `number` | Tiempo adicional tras recargar |

**Eventos:**

| Evento | Descripción |
|--------|-------------|
| `load` | Página completamente cargada |
| `domcontentloaded` | DOM listo (más rápido) |
| `networkidle` | Sin solicitudes de red pendientes |

---

### select

Selecciona una opción de un elemento `<select>` (dropdown).

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Selectores** | `array` | Selectores del dropdown |
| **Seleccionar Por** | `string` | Criterio de selección |
| **Valor** | `string` | Valor correspondiente al criterio |

**Criterios:**

| Criterio | Descripción |
|----------|-------------|
| `value` | Por el atributo `value` de la opción |
| `label` | Por el texto visible de la opción |
| `index` | Por el índice (posición) de la opción |

---

### press-key

Presiona una tecla del teclado.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Tecla** | `string` | Nombre de la tecla (ej: `Enter`, `Tab`, `Escape`) |
| **Selectores** | `array` | Selectores del elemento enfocado (opcional) |
| **Espera Después (ms)** | `number` | Tiempo de espera tras presionar |

---

### clock

Controla el reloj del navegador usando la API `page.clock` de Playwright. Es útil para probar reglas dependientes de fecha/hora, expiración de sesión, promociones temporales, bloqueos por horario, vencimientos, temporizadores y pantallas que cambian con el paso del tiempo.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Operación** | `setFixedTime` / `fastForward` / `pauseAt` / `resume` | Operación del reloj a ejecutar |
| **Fecha / Hora** | `string` | Fecha/hora usada por `setFixedTime` y `pauseAt` |
| **Duración** | `string` / `number` | Tiempo virtual avanzado por `fastForward` (`1000`, `05:00`, `02:34:10`) |
| **Recargar página después de aplicar** | `boolean` | Recarga la página después de aplicar el clock cuando está activado |
| **Esperar recarga hasta** | `load` / `domcontentloaded` / `networkidle` | Evento de carga usado cuando la recarga está activada |

**Operaciones:**

| Operación | Comportamiento |
|-----------|----------------|
| `setFixedTime` | Fija el reloj del navegador en una fecha/hora específica |
| `fastForward` | Avanza el tiempo virtual sin esperar tiempo real |
| `pauseAt` | Mueve a una fecha/hora específica y mantiene el reloj pausado |
| `resume` | Reanuda el reloj controlado desde el tiempo emulado actual |

Los nuevos pasos de clock comienzan con **Recargar página después de aplicar** desactivado por defecto. Esto conserva el estado actual de la pantalla y funciona mejor para páginas con temporizadores activos. Activa la recarga cuando la aplicación solo recalcula fecha/hora durante la carga de la página.

---

### frame

Cambia el contexto a un iframe.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Modo** | `string` | Método de selección del frame |
| **Selectores** | `array` | Selectores (modo `selector`) |
| **Nombre del Frame** | `string` | Nombre (modo `name`) |
| **Timeout (ms)** | `number` | Tiempo máximo de espera |

**Modos:**

| Modo | Descripción |
|------|-------------|
| `selector` | Localiza el iframe por selector CSS |
| `name` | Localiza el iframe por el atributo name |
| `main` | Vuelve al contexto principal (fuera del iframe) |

---

## Evidencias (Screenshots)

Cada paso puede tener configuración de evidencia individual:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Capturar Screenshot** | `boolean` | Activar captura |
| **Modo** | `before` / `after` / `both` | Cuándo capturar |
| **Plantilla de Nombre** | `string` | Nombre personalizado del archivo |
| **Esperar Carga** | `string` | Evento antes de capturar |
| **Delay (ms)** | `number` | Tiempo adicional antes de la captura |

---

## Prueba de Accesibilidad

Web Flow soporta escaneo automático de accesibilidad con **axe-core** integrado. Cuando está habilitado, el nodo audita la página después de cada paso que captura un screenshot, identificando violaciones WCAG/ARIA y marcando visualmente los elementos afectados.

### Configuración

| Campo | Tipo | Predeterminado | Descripción |
|-------|------|----------------|-------------|
| **Accessibility Scan** | `boolean` | `false` | Activa el escaneo axe-core |
| **Fail When Severity >=** | `none` / `minor` / `moderate` / `serious` / `critical` | `serious` | Nivel mínimo de severidad que reprueba el nodo |

### Cómo Funciona

Después de cada paso que toma un screenshot, axe-core es inyectado en la página y analiza el documento en busca de violaciones. Las violaciones se dibujan sobre el screenshot con cajas de colores alrededor de los elementos afectados y un panel de conteo en la esquina:

| Severidad | Color | Descripción |
|-----------|-------|-------------|
| **Critical** | 🔴 Rojo | Bloquea el acceso a usuarios con discapacidad |
| **Serious** | 🟠 Naranja | Causa dificultad significativa |
| **Moderate** | 🟡 Amarillo | Causa alguna dificultad |
| **Minor** | 🔵 Azul | Mejora recomendada |

Al final de la ejecución se generan automáticamente:

- **Screenshots por paso** — overlay con cajas de colores + panel `Critical N / Serious N / Moderate N / Minor N` + top 2 reglas
- **Gráfico de severidad** — `{id}-accessibility-severity-chart.png` — distribución por nivel
- **Gráfico de reglas** — `{id}-accessibility-rule-chart.png` — top 8 reglas más violadas
- **Reporte JSON** — `{id}-accessibility-report.json` — datos completos de todas las violaciones

### Criterio de Aprobación

El nodo falla si hay cualquier violación con severidad **igual o superior** al umbral configurado. Usa `none` para nunca reprobar (recolectar métricas sin bloquear el flujo).

### Ejemplo: Auditoría de inicio de sesión

```
Configuración del nodo:
  Accessibility Scan: true
  Fail When Severity >= : serious

Pasos:
1. navigate → https://misitio.com/login    (sin screenshot = sin escaneo)
2. type → #email → "{{ variables.EMAIL }}" → screenshot after → escaneo ejecutado
3. click → button[type="submit"]            → screenshot after → escaneo ejecutado
4. wait → networkIdle
5. assert → loginOk → hasURL → /dashboard  → screenshot after → escaneo ejecutado
```

Si cualquier paso tiene una violación `serious` o `critical`, el nodo falla con:
> *"Accessibility findings at or above "serious" were detected."*

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `sessionId` | `string` | ID de la sesión del navegador |
| `extracts` | `object` | Datos extraídos (clave → valor) |
| `asserts` | `object` | Resultados de las aserciones (clave → boolean) |
| `accessibilityPassed` | `boolean` | Si pasó el criterio de severidad (cuando está habilitado) |
| `accessibilityViolationCount` | `number` | Total de instancias de violación encontradas |
| `accessibility` | `object` | Reporte completo de accesibilidad |
| `accessibility.threshold` | `string` | Umbral configurado |
| `accessibility.scanCount` | `number` | Número de checkpoints escaneados |
| `accessibility.counts` | `object` | `{ minor, moderate, serious, critical }` — totales agregados |
| `accessibility.steps` | `array` | Detalles por checkpoint (url, counts, topRules) |
| `accessibility.rules` | `array` | Top 10 reglas violadas en todo el flujo |

---

## Ejemplo Completo

### Inicio de sesión en un sitio

Pasos configurados en el nodo:

1. **navigate** → `https://misitio.com/login`
2. **type** → Selector: `#email`, Texto: `{{ variables.USER_EMAIL }}`
3. **type** → Selector: `#password`, Texto: `{{ variables.USER_PASSWORD }}`
4. **click** → Selector: `button[type="submit"]`
5. **wait** → Modo: `networkIdle`
6. **assert** → Nombre: `loginOk`, Modo: `textContains`, Selector: `.welcome`, Texto: `Bienvenido`
7. **extract** → Nombre: `userName`, Selector: `.user-name`, Atributo: `text`

**Resultado:** Tras la ejecución, los outputs serán:
```json
{
  "sessionId": "abc-123",
  "extracts": { "userName": "João Silva" },
  "asserts": { "loginOk": true }
}
```

---

## Consejos

- Usa **data-testid** como estrategia principal de selector — es más estable
- Configura **screenshots** en los pasos críticos para facilitar la depuración
- Usa **reutilización de sesión** cuando múltiples nodos Web Flow/Smart Locators necesiten compartir el mismo navegador
- Para formularios con autocompletado o máscaras, considera usar el nodo [Smart Locators](smart-locators.md) con la acción `type` (escritura carácter por carácter)
- El modo **headless: false** es útil durante el desarrollo para visualizar lo que hace la prueba
