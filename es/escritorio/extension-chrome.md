# Extensión Chrome — QANode Recorder

El **QANode Recorder** es una extensión para Google Chrome que **graba sus interacciones** con páginas web y las convierte automáticamente en pasos de prueba compatibles con los nodos **Web Flow** o **Smart Locators** de QANode.

---

## Instalación

1. Abra Chrome y acceda a `chrome://extensions`
2. Active el **Modo de desarrollador** (toggle en la esquina superior derecha)
3. Haga clic en **Cargar descomprimida**
4. Seleccione la carpeta `extension/` de QANode
5. La extensión aparecerá en la barra de herramientas

---

## Modo del Recorder

Antes de iniciar la grabación, elija el **modo del recorder** en el popup de la extensión:

| Modo | Selectores | Cuándo Usar |
|------|------------|-------------|
| **Web Flow** (predeterminado) | CSS / XPath | Sitios con `data-testid`, `id` o selectores CSS estables |
| **Smart Locators** | Semánticos (getByRole, getByLabel, etc.) | Sitios con buenas prácticas de accesibilidad |

Cada modo genera un nodo del mismo nombre en el canvas al pegar el JSON.

> **Nota:** No es posible cambiar de modo durante la grabación. Si hay pasos grabados, cambiar de modo los eliminará.

Para más detalles sobre las diferencias entre los dos nodos, consulte:
- [Web Flow](../nodos/web/web-flow.md)
- [Smart Locators](../nodos/web/smart-locators.md)

---

## Modos de Grabación

La extensión opera en tres modos de interacción durante la grabación:

### REC — Grabación Normal

Graba automáticamente sus interacciones:

| Interacción | Acción Generada |
|-------------|----------------|
| **Clic** en elemento | `click` con selectores del elemento |
| **Escribir** en campo | `type` con texto y selectores |
| **Seleccionar** opción | `select` con valor seleccionado |
| **Desplazar** la página | `scroll` con posición |
| **Navegar** a URL | `navigate` con URL |
| **Recargar** página | `refresh` |

### CTRL+A — Modo Assert

Captura elementos para crear **aserciones** (verificaciones):

1. Presione **Ctrl+A** para activar el modo assert
2. Haga clic en el elemento a verificar
3. Se crea un paso `assert` con el texto/valor del elemento

### CTRL+E — Modo Extract

Captura elementos para **extracción de datos**:

1. Presione **Ctrl+E** para activar el modo extract
2. Haga clic en el elemento a extraer
3. Se crea un paso `extract` que captura el texto/valor

---

## Cómo Usar

### Grabando una Prueba

1. Haga clic en el ícono de la extensión en la barra de herramientas
2. Seleccione el **modo del recorder** (Web Flow o Smart Locators)
3. Haga clic en **Start** para iniciar la grabación
4. El indicador se pone rojo indicando grabación activa
5. Navegue e interactúe con el sitio normalmente
6. Use **Ctrl+A** para agregar verificaciones
7. Use **Ctrl+E** para agregar extracciones
8. Haga clic en el ícono nuevamente y haga clic en **Stop** para detener

[Popup de la extensión](../../assets/images/web-recorder.mp4)
*Imagen: Popup de la extensión mostrando botones REC/STOP, lista de pasos grabados y botón Copiar JSON*

### Usando en QANode

1. En el popup de la extensión, haga clic en **Copy JSON**
2. En QANode, abra el editor de flujos
3. Presione **Ctrl+V** en el canvas
4. Se creará un nodo automáticamente según el modo seleccionado:
   - Modo **Web Flow** → nodo Web Flow con `selectorStrategies` (CSS/XPath)
   - Modo **Smart Locators** → nodo Smart Locators con `locator` (getByRole, getByLabel, etc.)
5. Revise y ajuste los pasos según sea necesario

---

## Estrategias de Selector

### Modo Web Flow

En el modo Web Flow, la extensión genera automáticamente **múltiples selectores CSS/XPath** para cada elemento, ordenados por prioridad:

| Prioridad | Estrategia | Ejemplo |
|-----------|------------|---------|
| 1 | `data-testid` | `[data-testid="login-btn"]` |
| 2 | `id` | `#submit-button` |
| 3 | `data-qa` | `[data-qa="email-input"]` |
| 4 | `name` (inputs) | `input[name="email"]` |
| 5 | `aria-label` | `[aria-label="Enviar"]` |
| 6 | `placeholder` | `input[placeholder="E-mail"]` |
| 7 | `href` (links) | `a[href="/login"]` |
| 8 | Texto (botones/links) | `button:has-text("Entrar")` |
| 9 | XPath (fallback) | `//button[@type="submit"]` |

El selector más específico y estable siempre tiene prioridad. Esto garantiza pruebas más resilientes.

### Modo Smart Locators

En el modo Smart Locators, la extensión convierte los elementos capturados en **localizadores semánticos** de Playwright:

| Prioridad | Método | Ejemplo |
|-----------|--------|---------|
| 1 | `getByTestId` | Elemento con `data-testid="submit-btn"` |
| 2 | `getByLabel` | Campo con `aria-label="E-mail"` |
| 3 | `getByPlaceholder` | Campo con `placeholder="Digite seu nome"` |
| 4 | `getByText` | Elemento con texto visible |
| 5 | `getByRole` | Botón, enlace, encabezado (fallback) |

Los localizadores semánticos son más resilientes a cambios de diseño y reflejan cómo los usuarios realmente encuentran elementos en la página.

---

## Funciones Inteligentes

### Deduplicación

La extensión evita la duplicación de pasos:

- **Clic + Navegación**: Si un clic dispara una navegación, el paso `navigate` extra se suprime
- **Type + Type**: La escritura consecutiva en el mismo campo se consolida en un único paso
- **Scroll**: Los desplazamientos pequeños (< 200px) se ignoran

### Formularios

- **Focus out**: Guarda el valor del input cuando el campo pierde el foco
- **Enter**: Guarda el input y registra el clic en el botón submit
- **Change**: Guarda inmediatamente para dropdowns y selects

### Persistencia

La grabación sobrevive a **recargas de página**:
- Los pasos se guardan en `chrome.storage.local`
- Al recargar, la grabación continúa desde donde se dejó
- Se agrega automáticamente un paso `refresh`

### Detección de Navegación

La extensión usa la Performance API para distinguir entre:
- **Navegación** (nueva URL) → genera paso `navigate`
- **Recarga** (misma URL) → genera paso `refresh`
- **Clic → Navegación** (enlace clicado) → suprime el `navigate` redundante

---

## Formato de Exportación

El JSON copiado es compatible con el nodo correspondiente al modo seleccionado.

### Web Flow → Nodo Web Flow

```json
{
  "type": "web-flow",
  "position": { "x": 100, "y": 100 },
  "data": {
    "label": "Recorded Flow",
    "sessionMode": "new",
    "headless": true,
    "storageStrategy": "inMemory",
    "webSteps": [
      {
        "id": "step-1",
        "action": "navigate",
        "label": "Navigate to page",
        "config": {
          "url": "https://exemplo.com/login"
        },
        "evidence": {
          "takeScreenshot": true,
          "screenshotMode": "after"
        }
      },
      {
        "id": "step-2",
        "action": "type",
        "label": "Type \"user@email.com\"",
        "config": {
          "selectorStrategies": [
            { "type": "css", "value": "#email" },
            { "type": "css", "value": "input[name='email']" }
          ],
          "text": "user@email.com",
          "clearFirst": true
        }
      },
      {
        "id": "step-3",
        "action": "click",
        "label": "Click \"Entrar\"",
        "config": {
          "selectorStrategies": [
            { "type": "css", "value": "[data-testid='login-btn']" },
            { "type": "css", "value": "button[type='submit']" }
          ]
        }
      }
    ]
  }
}
```

### Smart Locators → Nodo Smart Locators

```json
{
  "type": "smart-locators",
  "position": { "x": 100, "y": 100 },
  "data": {
    "label": "Recorded Smart Locators",
    "sessionMode": "new",
    "headless": true,
    "storageStrategy": "inMemory",
    "smartSteps": [
      {
        "id": "step-1",
        "action": "navigate",
        "label": "Navigate to page",
        "config": {
          "url": "https://exemplo.com/login"
        },
        "evidence": {
          "takeScreenshot": true,
          "screenshotMode": "after"
        }
      },
      {
        "id": "step-2",
        "action": "fill",
        "label": "Fill \"user@email.com\"",
        "config": {
          "locator": {
            "method": "getByLabel",
            "value": "E-mail",
            "exact": true
          },
          "text": "user@email.com",
          "clearFirst": false
        }
      },
      {
        "id": "step-3",
        "action": "click",
        "label": "Click \"Entrar\"",
        "config": {
          "locator": {
            "method": "getByRole",
            "value": "button",
            "name": "Entrar",
            "exact": false
          }
        }
      }
    ]
  }
}
```

> **Diferencias principales:** En el modo Smart Locators, la acción `type` se convierte en `fill`, `select` en `selectOption`, y los selectores CSS son reemplazados por localizadores semánticos (`locator` en lugar de `selectorStrategies`).

---

## Flujo de Trabajo Recomendado

```
1. Elija el modo del recorder (Web Flow o Smart Locators)
2. Grabe la interacción con la extensión
3. Pegue en QANode (Ctrl+V)
4. Revise los pasos generados
5. Agregue waits donde sea necesario (la extensión no graba esperas)
6. Agregue asserts para validaciones (o grabe con Ctrl+A)
7. Ajuste selectores/localizadores si es necesario
8. Configure capturas de pantalla en los pasos críticos
9. Guarde y ejecute
```

---

## Limitaciones

- **Solo Chrome** — no funciona en otros navegadores
- **Sin waits automáticos** — agréguelos manualmente en QANode
- **Sin drag & drop** — no graba operaciones de arrastrar
- **Sitios con CSP restrictivo** — pueden bloquear la inyección del content script
- **iFrames cross-origin** — pueden no ser accesibles

---

## Consejos

- **Elija el modo correcto** — use Smart Locators para sitios accesibles, Web Flow para sitios con selectores CSS estables
- **Grabe acciones simples** y refine en QANode
- **Use Ctrl+A** (assert) durante la grabación en puntos de verificación
- **Use Ctrl+E** (extract) para capturar datos que se usarán en nodos posteriores
- **Agregue waits** en QANode para elementos que demoran en cargar
- **Revise los selectores** — en el modo Web Flow, prefiera `data-testid`; en el modo Smart Locators, prefiera `getByRole` o `getByLabel`
- **Grabe en segmentos** — para flujos largos, grabe por partes y combínelos en QANode
