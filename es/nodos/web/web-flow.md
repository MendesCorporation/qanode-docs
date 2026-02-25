# Nodo Web Flow

El nodo **Web Flow** permite automatizar interacciones con páginas web usando **selectores CSS, XPath, data-testid y texto**. Es ideal cuando necesitas selectores tradicionales y control detallado sobre la localización de elementos.

> **Consejo:** Si prefieres localizadores semánticos basados en accesibilidad (getByRole, getByLabel, etc.), usa el nodo [Smart Locators](smart-locators.md).

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

### Modo de Sesión

- **Nueva Sesión (`new`)**: Abre un nuevo navegador para cada ejecución. Ideal para pruebas aisladas.
- **Reutilizar Sesión (`reuse`)**: Usa una sesión de navegador ya abierta por otro nodo Web Flow/Smart Locators. Útil para dividir pruebas largas en múltiples nodos manteniendo el mismo navegador y cookies.

### Estrategia de Storage

- **En Memoria (`inMemory`)**: Las cookies y el localStorage se descartan al final de la ejecución.
- **Persistido (`persisted`)**: Las cookies y el localStorage se guardan con una clave y pueden reutilizarse en ejecuciones futuras. Ideal para mantener sesiones de inicio de sesión entre ejecuciones.

---

## Pasos (Web Steps)

El nodo Web Flow ejecuta una secuencia de **pasos** configurables. Cada paso representa una acción en el navegador.

### Acciones Disponibles

| Acción | Color | Descripción |
|--------|-------|-------------|
| [navigate](#navigate) | Azul | Navegar a una URL |
| [click](#click) | Amarillo | Hacer clic en un elemento |
| [type](#type) | Verde | Escribir texto en un campo |
| [wait](#wait) | Morado | Esperar una condición o tiempo |
| [assert](#assert) | Rojo | Verificar una condición en la página |
| [extract](#extract) | Cian | Extraer datos de un elemento |
| [hover](#hover) | Cian | Pasar el mouse sobre un elemento |
| [scroll](#scroll) | Amarillo | Desplazar la página |
| [refresh](#refresh) | Naranja | Recargar la página |
| [select](#select) | Morado | Seleccionar una opción de un dropdown |
| [press-key](#press-key) | Verde | Presionar una tecla |
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

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `sessionId` | `string` | ID de la sesión del navegador |
| `extracts` | `object` | Datos extraídos (clave → valor) |
| `asserts` | `object` | Resultados de las aserciones (clave → boolean) |

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
