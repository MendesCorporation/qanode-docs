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
| [wait](#wait) | Morado | Esperar condición |
| [scroll](#scroll) | Amarillo | Desplazar página |
| [refresh](#refresh) | Naranja | Recargar página |
| [frame](#frame) | Gris | Alternar frame |
| [rightClick](#rightclick) | Amarillo | Clic con botón derecho |
| [upload](#upload) | Azul | Subir archivos |
| [dialog](#dialog) | Morado | Manejar dialogs (alert/confirm) |
| [dragDrop](#dragdrop) | Azul | Arrastrar y soltar |
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

### frame

Cambia a un iframe. Soporta los modos `name` y `main` (volver al principal).

---

## Evidencias (Screenshots por Paso)

Idéntico a Web Flow — cada paso puede tener configuración individual de captura de screenshot.

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `sessionId` | `string` | ID de la sesión del navegador |
| `extracts` | `object` | Datos extraídos (clave → valor) |
| `asserts` | `object` | Resultados de las aserciones (clave → boolean) |

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
