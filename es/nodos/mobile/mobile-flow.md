# Nodo Mobile Flow

El nodo **Mobile Flow** permite automatizar interacciones con aplicaciones móviles nativas (Android e iOS) usando **Appium**. Soporta dispositivos reales, emuladores y servicios cloud como BrowserStack, Sauce Labs y LambdaTest.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `mobile-flow` |
| **Categoría** | Mobile |
| **Color** | Rojo claro (#f87171) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Requisitos Previos

### Appium

El nodo Mobile Flow requiere un servidor **Appium** accesible:

| Situación | Solución |
|-----------|----------|
| **Edición Desktop** | Appium se instala automáticamente en el primer uso y se inicia al ejecutar un flujo con nodo mobile |
| **Servidor propio** | Instalar manualmente: `npm install -g appium` + driver: `appium driver install uiautomator2` (Android) o `appium driver install xcuitest` (iOS) |
| **Cloud (BrowserStack, etc.)** | No se necesita Appium local — usa las credenciales del proveedor |

### Drivers de Appium

| Plataforma | Driver | Instalación |
|-----------|--------|-------------|
| Android | UiAutomator2 | `appium driver install uiautomator2` |
| iOS | XCUITest | `appium driver install xcuitest` (requiere Xcode en Mac) |

---

## Configuración General

| Campo | Tipo | Predeterminado | Descripción |
|-------|------|----------------|-------------|
| **Credencial** | `select` | — | Credencial Mobile con configuraciones de conexión |
| **Modo de Sesión** | `new` / `reuse` | `new` | Nueva sesión o reutilizar sesión existente |
| **Session ID** | `string` | — | ID para identificar/reutilizar la sesión |

### Modo de Sesión

- **Nueva Sesión (`new`)**: Abre una nueva sesión Appium en cada ejecución. Ideal para pruebas aisladas.
- **Reutilizar Sesión (`reuse`)**: Reutiliza una sesión ya creada por otro nodo Mobile Flow en el mismo flujo. Útil para dividir automatizaciones largas en múltiples nodos manteniendo el mismo dispositivo conectado.

---

## Credencial Mobile

La credencial centraliza la configuración de conexión y las capacidades del dispositivo. Configura en **Credenciales → Nueva Credencial → Mobile**.

### Modos de Conexión

| Modo | Descripción |
|------|-------------|
| **Local** | Appium ejecutándose en la misma máquina (predeterminado: `http://localhost:4723`) |
| **Self-hosted** | Appium en servidor propio con URL y token de autenticación |
| **SaaS** | Proveedor cloud (BrowserStack, Sauce Labs, LambdaTest, o Custom) |

### Capacidades del Dispositivo

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| **Plataforma** | `Android` o `iOS` | `Android` |
| **Nombre del Dispositivo** | Nombre o ID del dispositivo | `emulator-5554`, `Pixel 7` |
| **Versión de Plataforma** | Versión del SO | `13.0`, `17.0` |
| **App Package** | Paquete de la app Android | `com.miapp.android` |
| **App Activity** | Activity inicial de Android | `.MainActivity` |
| **Bundle ID** | Identificador de la app iOS | `com.miapp.ios` |
| **App Path** | Ruta al archivo .apk/.ipa | `/path/to/app.apk` |
| **UDID** | ID único del dispositivo real | `R58M20XXXXX` |
| **No Reset** | No reiniciar la app entre sesiones | `true` |
| **Full Reset** | Reiniciar completamente el estado de la app | `false` |
| **Automation Name** | Driver de automatización | `UiAutomator2` (Android), `XCUITest` (iOS) |
| **Capacidades Extra** | JSON con capacidades adicionales de Appium | `{"appium:language": "es"}` |

---

## Pasos (Mobile Steps)

El nodo Mobile Flow ejecuta una secuencia de **pasos** configurables. Cada paso representa una acción en el dispositivo.

### Acciones Disponibles

| Acción | Descripción |
|--------|-------------|
| [tap](#tap) | Tocar un elemento |
| [tap-coords](#tap-coords) | Tocar en coordenadas absolutas |
| [type](#type) | Escribir texto en un campo |
| [clear](#clear) | Limpiar el texto de un campo |
| [swipe](#swipe) | Deslizar en la pantalla |
| [scroll](#scroll) | Desplazar en una dirección |
| [wait](#wait) | Esperar una condición o tiempo |
| [assert](#assert) | Verificar una condición en la app |
| [extract](#extract) | Extraer texto o atributo de un elemento |
| [reset](#reset) | Reiniciar la aplicación |
| [back](#back) | Presionar el botón Atrás (Android) |
| [home](#home) | Presionar el botón Home (Android) |
| [key-event](#key-event) | Enviar un código de tecla Android |

---

## Estrategias de Selector Mobile

| Estrategia | Descripción | Ejemplo |
|------------|-------------|---------|
| `id` | Resource ID del elemento | `com.miapp:id/login_button` |
| `accessibility_id` | Content description / accessibility label | `Botón de inicio de sesión` |
| `xpath` | Expresión XPath en la jerarquía XML | `//android.widget.Button[@text="Iniciar sesión"]` |
| `class_name` | Nombre de clase nativa | `android.widget.EditText` |
| `-android uiautomator` | Selector UIAutomator2 (Android) | `new UiSelector().text("Iniciar sesión")` |
| `-ios predicate string` | NSPredicate (iOS) | `type == 'XCUIElementTypeButton' AND name == 'Login'` |
| `-ios class chain` | iOS Class Chain | `**/XCUIElementTypeButton[\`name == 'Login'\`]` |

> **Consejo:** El [Inspector Mobile](inspector-mobile.md) genera automáticamente los selectores al tocar los elementos, sin necesidad de escribir XPath manualmente.

---

## Detalle de las Acciones

### tap

Toca un elemento usando estrategias de selector.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Selectores** | `array` | Estrategias de selector del elemento |
| **Intentos** | `number` | Número de intentos (predeterminado: 3) |
| **Delay entre intentos (ms)** | `number` | Espera entre intentos (predeterminado: 500ms) |

Si el selector falla en todos los intentos y existen coordenadas de fallback (generadas por el inspector), el toque se realizará en las coordenadas directas.

---

### tap-coords

Toca en coordenadas absolutas en la pantalla.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **X** | `number` | Coordenada X en píxeles |
| **Y** | `number` | Coordenada Y en píxeles |

---

### type

Escribe texto en un campo de entrada.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Selectores** | `array` | Estrategias de selector del campo |
| **Texto** | `string` | Texto a escribir (soporta `{{ }}`) |
| **Limpiar Primero** | `boolean` | Limpia el campo antes de escribir |
| **Intentos** | `number` | Número de intentos de localización |

---

### clear

Limpia el contenido de un campo de entrada.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Selectores** | `array` | Estrategias de selector del campo |

---

### swipe

Desliza en la pantalla entre dos puntos.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **fromX / fromY** | `number` | Coordenadas de inicio |
| **toX / toY** | `number` | Coordenadas de destino |
| **Duración (ms)** | `number` | Velocidad del gesto (predeterminado: 350ms) |
| **Dirección** | `string` | Alternativa: `up`, `down`, `left`, `right` |
| **Porcentaje** | `number` | Porcentaje de pantalla a deslizar (con dirección) |

---

### scroll

Desplaza el contenido en una dirección.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Dirección** | `string` | `up`, `down`, `left`, `right` |
| **Selectores** | `array` | Selector del contenedor a desplazar (opcional) |

---

### wait

Espera una condición antes de continuar.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Modo** | `string` | Tipo de espera |
| **Timeout (ms)** | `number` | Tiempo máximo de espera |
| **Selectores** | `array` | Selectores (modo `elementVisible`) |

**Modos:**

| Modo | Descripción |
|------|-------------|
| `timeout` | Espera un tiempo fijo en milisegundos |
| `elementVisible` | Espera a que un elemento aparezca en pantalla |

---

### assert

Verifica una condición en la aplicación. Si falla, el paso se marca como fallido.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Nombre** | `string` | Identificador de la aserción (clave en el output) |
| **Modo** | `string` | Tipo de verificación |
| **Selectores** | `array` | Selectores del elemento |
| **Texto Esperado** | `string` | Valor esperado (modos texto) |
| **Continuar en Fallo** | `boolean` | No interrumpir el flujo si falla |

**Modos:**

| Modo | Descripción |
|------|-------------|
| `elementExists` | Verifica si el elemento está visible en pantalla |
| `textContains` | Verifica si el texto del elemento contiene el valor esperado |
| `textEquals` | Verifica si el texto del elemento es exactamente el valor esperado |

Los resultados de las aserciones están disponibles en los outputs:
```
{{ steps["mobile-flow"].outputs.asserts.nombreAsercion }}  →  true o false
```

---

### extract

Extrae texto o un atributo de un elemento.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Nombre** | `string` | Nombre de la extracción (clave en el output) |
| **Selectores** | `array` | Selectores del elemento |
| **Atributo** | `string` | Qué extraer: `text` (predeterminado) o nombre del atributo |

Los datos extraídos están disponibles en los outputs:
```
{{ steps["mobile-flow"].outputs.extracts.nombreExtraccion }}
```

---

### reset

Reinicia la aplicación, limpiando su estado (equivalente a desinstalar y reinstalar).

---

### back

Presiona el botón físico/virtual **Atrás** (Android — keycode 4).

---

### home

Presiona el botón **Home** (Android — keycode 3).

---

### key-event

Envía un keycode de Android directamente al dispositivo.

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Keycode** | `number` | Código de tecla Android (ej.: 4 = BACK, 3 = HOME, 66 = ENTER) |
| **Repeticiones** | `number` | Cuántas veces enviar el evento |

**Keycodes comunes:**

| Keycode | Tecla |
|---------|-------|
| 3 | HOME |
| 4 | BACK |
| 66 | ENTER |
| 67 | BACKSPACE/DEL |
| 82 | MENU |
| 84 | SEARCH |

---

## Evidencias (Capturas de pantalla)

Cada paso puede tener configuración de evidencia individual:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Capturar Pantalla** | `boolean` | Activar captura de pantalla |
| **Modo** | `before` / `after` / `both` | Cuándo capturar |
| **Nombre de archivo** | `string` | Nombre personalizado del archivo |
| **Esperar Estabilización** | `boolean` | Esperar pantalla estable antes de capturar |
| **Timeout de Estabilización (ms)** | `number` | Tiempo máximo esperando pantalla estable (predeterminado: 2000ms) |
| **Delay (ms)** | `number` | Tiempo adicional antes de capturar (predeterminado: 300ms) |

> Las capturas generadas están disponibles en la pestaña **Evidencias** del resultado de ejecución. Los toques y swipes se destacan con marcación visual verde sobre la captura.

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `sessionId` | `string` | ID de sesión mobile para reutilización en otros nodos |
| `extracts` | `object` | Datos extraídos por los pasos `extract` (clave → valor) |
| `asserts` | `object` | Resultados de aserciones (clave → `true`/`false`) |

---

## Ejemplo Completo

### Inicio de sesión en app Android

Pasos configurados en el nodo:

1. **wait** → Modo: `elementVisible`, Selector: `id=com.miapp:id/email_field` (timeout: 10000ms)
2. **tap** → Selector: `id=com.miapp:id/email_field`
3. **type** → Selector: `id=com.miapp:id/email_field`, Texto: `{{ variables.USER_EMAIL }}`
4. **type** → Selector: `id=com.miapp:id/password_field`, Texto: `{{ variables.USER_PASSWORD }}`, Limpiar: true
5. **tap** → Selector: `accessibility_id=Iniciar sesión`
6. **wait** → Modo: `elementVisible`, Selector: `id=com.miapp:id/home_title` (timeout: 15000ms)
7. **assert** → Nombre: `loginOk`, Modo: `elementExists`, Selector: `id=com.miapp:id/home_title`
8. **extract** → Nombre: `welcomeText`, Selector: `id=com.miapp:id/welcome_message`, Atributo: `text`

**Resultado tras la ejecución:**
```json
{
  "sessionId": "mi-sesion-android",
  "extracts": { "welcomeText": "¡Hola, Juan!" },
  "asserts": { "loginOk": true }
}
```

---

### Reutilizando Sesión entre Nodos

**Nodo 1 — Mobile Flow (Inicio de sesión)**
- Session ID: `sesion-android`
- Pasos: inicio de sesión en la app

**Nodo 2 — Mobile Flow (Verificar Pedido)**
- Modo de Sesión: `reuse`
- Session ID: `sesion-android`
- Pasos: navegar a pedidos, verificar estado

Ambos nodos comparten el mismo dispositivo y estado de sesión.

---

## Consejos

- Usa el **[Inspector Mobile](inspector-mobile.md)** para grabar pasos visualmente — genera los selectores automáticamente al tocar los elementos
- En la **edición Desktop**, Appium se inicia automáticamente cuando se ejecuta un flujo con nodo mobile — no se necesita configuración manual
- Prefiere `accessibility_id` como selector principal — es más estable que XPath y funciona en Android e iOS
- Configura **capturas de pantalla** en los pasos críticos para facilitar la identificación de fallos
- Usa **Continuar en Fallo** en aserciones de monitoreo para recopilar múltiples resultados sin interrumpir el flujo
- Para apps que tardan en iniciar, agrega un paso `wait` con `elementVisible` antes de interactuar
- El `Session ID` personalizado facilita la reutilización de sesión entre múltiples nodos Mobile Flow en el mismo flujo
