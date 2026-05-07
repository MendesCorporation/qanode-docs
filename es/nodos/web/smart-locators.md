# Nodo Smart Locators

El nodo **Smart Locators** permite automatizar interacciones con páginas web usando **localizadores semánticos de Playwright** — basados en roles ARIA, labels, placeholders y otros atributos de accesibilidad. Este enfoque es más resiliente que los selectores CSS, ya que refleja cómo los usuarios realmente encuentran los elementos en la página.

> **Consejo:** Si prefieres selectores CSS y XPath tradicionales, usa el nodo [Web Flow](web-flow.md).

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `smart-locators` |
| **Categoría** | Web |
| **Color** | Azul (#3b82f6) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Configuración General

Idéntica al nodo Web Flow:

| Campo | Tipo | Predeterminado | Descripción |
|-------|------|----------------|-------------|
| **Modo de Sesión** | `new` / `reuse` | `new` | Nueva sesión o reutilizar una existente |
| **Session ID** | `string` | — | ID de la sesión a reutilizar |
| **Headless** | `boolean` | `true` | Ejecutar sin interfaz gráfica visible |
| **Estrategia de Storage** | `inMemory` / `persisted` | `inMemory` | Persistir cookies/localStorage |
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
| **Self Healing** | `boolean` | `false` | Intenta recuperar localizadores cuando el elemento cambia y el locator original ya no resuelve el objetivo |

> Las opciones de Navegador & Dispositivo son idénticas al nodo Web Flow — consulta la [documentación de Web Flow](web-flow.md#navegador) para detalles sobre presets de viewport y dispositivos mobile disponibles.

### Auditoría de Rendimiento

Cuando **Auditoría de rendimiento** está habilitada, Smart Locators monitorea la sesión activa de Playwright y genera evidencias visuales de rendimiento por pantalla.

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

Cuando **Self Healing** está habilitado, Smart Locators puede recuperar automáticamente acciones basadas en localizadores semánticos cuando el locator original deja de encontrar el elemento esperado.

Además del propio locator semántico, el motor considera:

- texto y nombre accesible del elemento
- `role`
- `label`, `placeholder`, `title` y otros atributos relevantes
- contexto estructural de la página, como contenedor, formulario, heading y posición relativa

Esto ayuda en escenarios como:

- cambios de texto con el mismo significado
- cambios entre mayúsculas y minúsculas
- texto con o sin acentos
- variaciones cortas, como `Ingresar` → `Login` o `Siguiente` → `Continuar`

La función es conservadora: solo aplica la recuperación cuando encuentra un candidato claramente más fuerte que los demás, evitando “adivinar” en pantallas ambiguas.

Cuando un paso es recuperado, la ejecución muestra:

- el paso afectado
- la confianza de la recuperación
- el locator utilizado
- una opción **Aplicar al Flujo** para actualizar el flujo con el locator recuperado

---

## Sistema de Localizadores

### Métodos de Localización

En lugar de selectores CSS, Smart Locators usa **métodos semánticos** de Playwright:

| Método | Descripción | Ejemplo |
|--------|-------------|---------|
| `getByRole` | Localiza por ARIA role | Botón "Ingresar", Enlace "Inicio" |
| `getByText` | Localiza por texto visible | Texto "Bienvenido" |
| `getByLabel` | Localiza por label asociada | Campo con label "Correo electrónico" |
| `getByPlaceholder` | Localiza por placeholder | Campo con placeholder "Ingresa tu nombre" |
| `getByAltText` | Localiza por alt text | Imagen con alt "Logo" |
| `getByTitle` | Localiza por title | Elemento con title "Cerrar" |
| `getByTestId` | Localiza por data-testid | `data-testid="submit-btn"` |

### Configuración del Localizador

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Método** | `string` | Método de localización |
| **Valor** | `string` | Texto/valor a buscar |
| **Nombre** | `string` | Nombre del role (solo para `getByRole`) |
| **Exacto** | `boolean` | Coincidencia exacta (false = contiene) |
| **Nth** | `number` | Índice del elemento (cuando hay múltiples) |

### ARIA Roles Disponibles

Para el método `getByRole`, están disponibles los siguientes roles:

| Roles de Interacción | Roles de Estructura | Roles de Retroalimentación |
|----------------------|---------------------|---------------------------|
| `button` | `heading` | `alert` |
| `checkbox` | `list` | `alertdialog` |
| `combobox` | `listitem` | `dialog` |
| `link` | `navigation` | `progressbar` |
| `listbox` | `region` | `status` |
| `menu` | `row` | `tooltip` |
| `menuitem` | `cell` | — |
| `option` | `grid` | — |
| `radio` | `tab` | — |
| `searchbox` | `tablist` | — |
| `slider` | `tabpanel` | — |
| `spinbutton` | `toolbar` | — |
| `switch` | `tree` | — |
| `textbox` | `treeitem` | — |

**Ejemplo de uso:**
```
Método: getByRole
Valor: button
Nombre: Ingresar
```

Equivale en Playwright a: `page.getByRole('button', { name: 'Ingresar' })`

---

## Acciones Disponibles

| Acción | Color | Descripción |
|--------|-------|-------------|
| [navigate](#navigate) | Azul | Navegar a URL |
| [click](#click) | Amarillo | Hacer clic en elemento |
| [dblclick](#dblclick) | Amarillo | Doble clic |
| [fill](#fill) | Verde | Rellenar campo (instantáneo) |
| [type](#type) | Verde | Escribir carácter por carácter |
| [check](#check) | Verde | Marcar checkbox/radio |
| [uncheck](#uncheck) | Verde | Desmarcar checkbox |
| [selectOption](#selectoption) | Morado | Seleccionar opción de dropdown |
| [hover](#hover) | Cian | Pasar mouse sobre elemento |
| [focus](#focus) | Azul claro | Enfocar elemento |
| [press](#press) | Verde | Presionar tecla |
| [assert](#assert) | Rojo | Verificar condición |
| [extract](#extract) | Cian | Extraer datos |
| [extractList](#extractlist) | Esmeralda | Extraer lista de elementos repetidos |
| [wait](#wait) | Morado | Esperar condición |
| [scroll](#scroll) | Amarillo | Desplazar página |
| [refresh](#refresh) | Naranja | Recargar página |
| [clock](#clock) | Verde azulado | Controlar el reloj del navegador |
| [frame](#frame) | Gris | Alternar frame |
| [rightClick](#rightclick) | Amarillo | Clic con botón derecho |
| [upload](#upload) | Azul | Subir archivos |
| [dialog](#dialog) | Morado | Manejar dialogs (alert/confirm) |
| [drag](#drag) | Rosa | Arrastrar y soltar |
| [evaluate](#evaluate) | Blanco | Ejecutar JavaScript en la página |
| [screenshot](#screenshot) | Azul | Capturar screenshot |

---

## Detalles de las Acciones

### navigate

Navega a una URL.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **URL** | `string` | URL de destino (soporta `{{ }}`) |

---

### click

Hace clic en un elemento localizado semánticamente.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Localizador** | `locator` | Configuración del localizador |
| **Intentos** | `number` | Número de intentos (predeterminado: 3) |

El click espera a que el elemento sea visible, se desplaza hasta él si es necesario, y luego hace clic. Si falla, vuelve a intentarlo.

**Ejemplo:**
```
Método: getByRole
Valor: button
Nombre: Enviar
Intentos: 3
```

---

### dblclick

Doble clic en un elemento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Localizador** | `locator` | Configuración del localizador |
| **Intentos** | `number` | Número de intentos |

---

### fill

Rellena un campo de entrada **instantáneamente** (reemplaza el valor).

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Localizador** | `locator` | Configuración del localizador |
| **Texto** | `string` | Texto a rellenar (soporta `{{ }}`) |
| **Limpiar Primero** | `boolean` | Limpia el campo antes |

Usa `locator.fill()` de Playwright — ideal para campos simples.

---

### type

Escribe texto **carácter por carácter**, simulando la escritura real.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Localizador** | `locator` | Configuración del localizador |
| **Texto** | `string` | Texto a escribir |
| **Delay (ms)** | `number` | Intervalo entre teclas (predeterminado: 50ms) |
| **Limpiar Primero** | `boolean` | Limpia el campo antes |

Usa `locator.pressSequentially()` de Playwright — ideal para campos con:
- **Autocompletado** (sugerencias aparecen mientras se escribe)
- **Máscaras** (teléfonos, códigos postales)
- **Validación on-keypress** (valida cada tecla)

**Ejemplo:**
```
Localizador: getByPlaceholder → "Teléfono"
Texto: 123.456.789-00
Delay: 100ms
```

---

### check / uncheck

Marca o desmarca un checkbox/radio button.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Localizador** | `locator` | Configuración del localizador |

**Ejemplo:**
```
Método: getByLabel
Valor: Acepto los términos de uso
```

---

### selectOption

Selecciona una opción de un dropdown (`<select>`).

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Localizador** | `locator` | Configuración del dropdown |
| **Seleccionar Por** | `string` | `value`, `label` o `index` |
| **Valor** | `string` | Valor correspondiente |

---

### hover

Pasa el mouse sobre un elemento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Localizador** | `locator` | Configuración del localizador |
| **Intentos** | `number` | Número de intentos |
| **Espera Después (ms)** | `number` | Tiempo de espera tras hover |

---

### focus

Establece el foco en un elemento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Localizador** | `locator` | Configuración del localizador |

---

### press

Presiona una tecla del teclado en el contexto de un elemento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Localizador** | `locator` | Elemento que recibirá la tecla |
| **Tecla** | `string` | Nombre de la tecla (`Enter`, `Tab`, `Escape`, etc.) |
| **Espera Después (ms)** | `number` | Tiempo de espera tras presionar |

---

### drag

Arrastra un elemento localizado semánticamente y lo suelta sobre otro localizador o en coordenadas grabadas.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Localizador de Origen** | `locator` | Elemento que será arrastrado |
| **Localizador de Destino** | `locator` | Elemento donde será soltado, cuando exista |
| **Destino X / Y** | `number` | Coordenadas absolutas que se muestran cuando el destino no tiene un localizador confiable |
| **Esperar Después (ms)** | `number` | Tiempo de espera después de soltar |

Smart Locators intenta usar localizadores semánticos de Playwright, como `getByRole`, `getByText`, `getByLabel` y `getByTestId`. Cuando la página tiene elementos repetidos y el paso viene de la extensión, el JSON puede incluir metadatos internos de posición para ayudar al ejecutor a elegir el elemento más cercano al gesto original. Estos metadatos no aparecen como campos principales en el panel.

Para componentes de drag/drop, QANode también intenta promover el localizador semántico al contenedor interactivo real. Esto es útil cuando el texto clicado queda dentro de otro elemento arrastrable, por ejemplo:

- Un `<p>` dentro de un card arrastrable
- Una imagen dentro de un contenedor `draggable`
- Un área de drop marcada por clase o atributos de drop

Si el destino semántico es demasiado genérico o falla, pero existe `Destino X / Y`, Smart Locators puede usar las coordenadas grabadas como fallback. Esto permite automatizar canvas, Kanban y áreas de soltar que no tienen texto, role o label propios.

**Buenas prácticas:**

- Prefiere componentes con nombres accesibles (`aria-label`, texto visible, labels)
- Usa `data-testid` o `data-qa` para cards, columnas y áreas de drop críticas
- Evita botones solo con ícono sin `aria-label`
- Revisa grabaciones que dependan de `nth`, especialmente cuando hay textos repetidos
- Usa screenshots `before` y `after` para comprobar visualmente el movimiento

---

### assert

Verifica condiciones en la página. Ofrece múltiples modos de verificación:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Nombre** | `string` | Identificador de la aserción |
| **Localizador** | `locator` | Elemento objetivo (cuando aplica) |
| **Modo** | `string` | Tipo de verificación |
| **Texto Esperado** | `string` | Valor esperado |
| **Continuar en Fallo** | `boolean` | No fallar el nodo completo |

**Modos de Assert:**

| Modo | Descripción | Requiere Localizador |
|------|-------------|---------------------|
| `visible` | El elemento es visible | Sí |
| `hidden` | El elemento está oculto | Sí |
| `textContains` | El texto contiene el valor | Sí |
| `textEquals` | El texto es exactamente el valor | Sí |
| `hasCount` | Conteo de elementos | Sí |
| `enabled` | El elemento está habilitado | Sí |
| `disabled` | El elemento está deshabilitado | Sí |
| `checked` | El checkbox/radio está marcado | Sí |
| `hasValue` | El input tiene un valor específico | Sí |
| `hasAttribute` | El elemento tiene atributo con valor | Sí |
| `hasURL` | La URL de la página contiene/es igual a | No |
| `hasTitle` | El título de la página contiene/es igual a | No |

**Ejemplos:**

```
// Verificar que el botón "Enviar" es visible
Modo: visible, Localizador: getByRole("button", "Enviar")

// Verificar que el mensaje contiene "Éxito"
Modo: textContains, Localizador: getByText("Éxito"), Esperado: "Éxito"

// Verificar URL después del inicio de sesión
Modo: hasURL, Esperado: "/dashboard"

// Verificar que el campo está deshabilitado
Modo: disabled, Localizador: getByLabel("Correo electrónico")
```

---

### extract

Extrae datos de un elemento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Nombre** | `string` | Nombre de la extracción |
| **Localizador** | `locator` | Configuración del localizador |
| **Atributo** | `string` | Qué extraer |

**Atributos disponibles:**

| Atributo | Descripción |
|----------|-------------|
| `text` | Texto visible (`textContent`) |
| `innerText` | Texto interno |
| `innerHTML` | HTML interno |
| `inputValue` | Valor de campos de input |
| `href` | URL de enlaces |
| `src` | URL de imágenes |
| `value` | Valor genérico |

---

### extractList

Itera sobre una lista de elementos repetidos (filas de tabla, cards, ítems de lista) y extrae campos nombrados de cada uno, produciendo un array de objetos como output.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Nombre** | `string` | Nombre de la extracción (clave en el array de output) |
| **Localizador de Contenedor** | `locator` | Configuración del localizador para encontrar el contenedor de la lista |
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

### wait

Espera una condición.

| Modo | Descripción |
|------|-------------|
| `timeout` | Espera un tiempo fijo |
| `visible` | Espera a que el elemento sea visible |
| `hidden` | Espera a que el elemento desaparezca |
| `networkIdle` | Espera a que terminen las solicitudes de red |

---

### scroll

Desplaza la página.

| Modo | Descripción |
|------|-------------|
| `toLocator` | Desplaza hasta el elemento |
| `by` | Desplaza una cantidad fija de píxeles |
| `toBottom` | Desplaza hasta el final de la página |

---

### refresh

Recarga la página. Misma configuración que Web Flow.

---

### clock

Controla el reloj del navegador usando `page.clock` de Playwright. Es la misma acción disponible en Web Flow y puede usarse para validar comportamientos por fecha/hora sin depender del reloj real de la máquina.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Operación** | `setFixedTime` / `fastForward` / `pauseAt` / `resume` | Operación del reloj a ejecutar |
| **Fecha / Hora** | `string` | Fecha/hora usada por `setFixedTime` y `pauseAt` |
| **Duración** | `string` / `number` | Tiempo virtual avanzado por `fastForward` (`1000`, `05:00`, `02:34:10`) |
| **Recargar página después de aplicar** | `boolean` | Recarga la página después de aplicar el clock cuando está activado |
| **Esperar recarga hasta** | `load` / `domcontentloaded` / `networkidle` | Evento de carga usado cuando la recarga está activada |

Los nuevos pasos de clock comienzan con la recarga desactivada. Usa la recarga solo cuando la aplicación depende de la carga de la página para recalcular fechas u horarios.

---

### frame

Cambia a un iframe. Soporta los modos `name` y `main` (volver al principal).

---

## Evidencias (Screenshots por Paso)

Idéntico a Web Flow — cada paso puede tener configuración individual de captura de screenshot.

---

## Prueba de Accesibilidad

Smart Locators soporta escaneo automático de accesibilidad con **axe-core** integrado. Cuando está habilitado, el nodo audita la página después de cada paso que captura un screenshot, identificando violaciones WCAG/ARIA y marcando visualmente los elementos afectados.

> Smart Locators ya usa localizadores semánticos basados en roles ARIA — combinarlo con el escaneo de accesibilidad es especialmente útil para verificar que los elementos con los que interactúas cumplen los criterios de accesibilidad.

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

### Ejemplo: Verificar accesibilidad del formulario de inicio de sesión

```
Configuración del nodo:
  Accessibility Scan: true
  Fail When Severity >= : serious

Pasos:
1. navigate → https://misitio.com/login               (sin screenshot = sin escaneo)
2. fill → getByLabel("Correo electrónico") → "..."     → screenshot after → escaneo ejecutado
3. fill → getByLabel("Contraseña") → "..."             → screenshot after → escaneo ejecutado
4. click → getByRole("button", { name: "Ingresar" })  → screenshot after → escaneo ejecutado
5. wait → networkIdle
6. assert → "loginOk" → textContains → getByText("Bienvenido") → screenshot → escaneo ejecutado
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

## Ejemplo Completo: Inicio de Sesión con Smart Locators

```
Paso 1: navigate → https://misitio.com/login
Paso 2: fill → getByLabel("Correo electrónico") → "{{ variables.EMAIL }}"
Paso 3: fill → getByLabel("Contraseña") → "{{ variables.CONTRASENA }}"
Paso 4: click → getByRole("button", { name: "Ingresar" })
Paso 5: wait → networkIdle
Paso 6: assert → "loginOk" → textContains → getByText("Bienvenido")
Paso 7: extract → "userName" → getByRole("heading") → text
```

---

## Cuándo Usar Smart Locators vs Web Flow

| Escenario | Recomendación |
|-----------|--------------|
| Sitio con buenas prácticas de accesibilidad | **Smart Locators** |
| Sitio con data-testid en todos los elementos | **Web Flow** |
| Formularios con labels | **Smart Locators** |
| Elementos sin texto ni role | **Web Flow** (CSS/XPath) |
| Pruebas de accesibilidad | **Smart Locators** |
| Legacy / sitios sin semántica | **Web Flow** |

---

## Consejos

- **Prefiere `getByRole`** — es el más resiliente y refleja cómo las tecnologías asistivas encuentran los elementos
- **Usa `getByLabel`** para campos de formulario — es más estable que CSS
- **`getByTestId`** es excelente para elementos sin role/label claro
- Usa la acción **type** (en lugar de fill) para campos con máscaras o autocompletado
- El campo **Nth** permite seleccionar un elemento específico cuando múltiples coinciden con el localizador
