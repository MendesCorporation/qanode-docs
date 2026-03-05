# Inspector Mobile

El **Inspector Mobile** es una herramienta visual integrada en QANode para grabar y construir pasos de automatización mobile de forma interactiva. En lugar de escribir XPath y selectores manualmente, interactúas directamente con la pantalla del dispositivo y los pasos se generan automáticamente.

---

## Cómo Acceder

El Inspector Mobile se abre desde el editor de flujos:

1. Agrega un nodo **Mobile Flow** al flujo
2. Configura la credencial mobile (dispositivo, plataforma, app)
3. Haz clic en el botón con el **icono de lupa** en el panel de propiedades del nodo

> En la **edición Desktop**, Appium se inicia automáticamente al conectar. En la edición web/servidor, asegúrate de que Appium esté en ejecución antes de abrir el inspector.

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

Muestra la captura de pantalla actualizada del dispositivo en tiempo real. Puedes:
- **Tocar** elementos para generar pasos `tap`
- **Arrastrar** para generar pasos `swipe`
- **Clic derecho** para inspeccionar un elemento sin tocarlo (funciona en cualquier modo)

### Área de Pasos Grabados

Lista todos los pasos registrados durante la sesión. Cada acción muestra:
- Tipo de acción (tap, type, swipe, etc.)
- Descripción del objetivo o texto
- Botón para eliminar el paso individualmente

### Panel del Elemento Seleccionado

Muestra la información del elemento tocado en modo **Inspeccionar**:
- Clase nativa (`android.widget.Button`, `XCUIElementTypeTextField`, etc.)
- Resource ID, Accessibility ID, XPath
- Bounds (posición y tamaño en pantalla)
- Texto actual y atributos relevantes

---

## Modos de Operación

### Modo Toque (predeterminado)

En el modo **Toque**, cada clic en la pantalla del dispositivo:
1. Envía el toque al dispositivo via Appium
2. Graba un paso `tap` o `tap-coords` con los selectores del elemento tocado
3. Actualiza la captura de pantalla automáticamente tras la acción

Para **deslizar** (swipe): haz clic y arrastra en la pantalla del dispositivo — el gesto se ejecuta en el dispositivo y se graba como paso `swipe`.

### Modo Inspeccionar

En el modo **Inspeccionar**, hacer clic en un elemento **no envía el toque al dispositivo** — solo muestra la información del elemento en el panel lateral. Usa este modo para:
- Explorar la jerarquía de elementos sin interactuar con la app
- Copiar selectores para uso manual
- Identificar el mejor selector antes de crear un paso

> **Consejo:** También puedes hacer **clic derecho** en cualquier lugar de la pantalla para inspeccionar un elemento sin cambiar al modo Inspeccionar.

---

## Grabando Pasos

### Paso de Toque (tap)

1. Asegúrate de estar en el **Modo Toque**
2. Haz clic en el elemento deseado en la pantalla del dispositivo
3. El inspector detecta el elemento, extrae sus selectores y graba un paso `tap`
4. La pantalla se actualiza automáticamente

### Paso de Escritura (type)

Después de tocar un campo de texto:
1. El inspector detecta que el campo está activo
2. Escribe el texto en tu teclado físico — el inspector captura las pulsaciones en tiempo real
3. El texto escrito se agrupa y se graba como paso `type`
4. Presionar **Backspace** borra caracteres y actualiza el paso

### Paso de Swipe

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

## Guardando los Pasos

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

1. **`accessibility_id`** — Cuando el elemento tiene content-description o accessibility label
2. **`id`** — Cuando el elemento tiene resource-id (Android) o nombre único (iOS)
3. **`xpath`** con `@bounds` — Selector basado en las coordenadas del elemento (más preciso para la pantalla actual)
4. **Coordenadas de fallback** — X/Y absolutos usados si todos los selectores fallan

> Los selectores con `@bounds` son generados por el inspector para alta precisión en la pantalla específica. En flujos automatizados, si la resolución u orientación cambia, los selectores por `accessibility_id` o `id` son más robustos.

---

## Consejos

- Usa el **Modo Inspeccionar** (o clic derecho) para identificar el mejor selector antes de grabar, especialmente en pantallas con muchos elementos similares
- **Graba un flujo completo** de una vez — conectar, ejecutar el caso de uso en la app, guardar — para obtener pasos con contexto real
- Tras guardar, revisa los pasos en el nodo y agrega **capturas de evidencia** en los pasos críticos
- Para apps con animaciones largas, agrega pasos `wait` (`elementVisible`) después de los toques para asegurarte de que la pantalla cargó antes de la siguiente acción
- En la **edición Desktop**, la conexión con el dispositivo local es más rápida — úsala para desarrollo. Para pruebas en cloud (BrowserStack, etc.), configura la credencial SaaS antes de abrir el inspector
