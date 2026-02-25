# Editor de Flujos — Visión General

El **Editor de Flujos** es el corazón de QANode. Es donde creas, configuras y ejecutas tus escenarios de prueba de forma visual.

---

## Interfaz del Editor

El editor está dividido en cuatro áreas principales:

<!-- ![Visión general del editor de flujos](../../assets/images/editor-visao-geral.png) -->
*Imagen: Editor de flujos mostrando las cuatro áreas: barra superior, paleta de nodos, canvas y panel de propiedades*

### 1. Barra Superior

La barra superior contiene:

| Elemento | Descripción |
|----------|-------------|
| **Botón Volver** | Regresa al proyecto (muestra una advertencia si hay cambios sin guardar) |
| **Nombre del Flujo** | Campo editable con el nombre del escenario |
| **Botón Guardar** | Guarda el flujo actual (Ctrl+S) |
| **Botón Ejecutar** | Ejecuta el flujo y muestra los resultados |
| **Menú de Opciones** | Exportar, importar, duplicar y otras opciones |

### 2. Paleta de Nodos (Lado Izquierdo)

La paleta muestra todos los nodos disponibles organizados por categoría:

- **Control de Flujo** — If, Switch, Loop, Merge
- **Web** — Web Flow, Smart Locators
- **API** — HTTP Request
- **Base de Datos** — PostgreSQL, MySQL, MariaDB, Oracle, MongoDB
- **Infraestructura** — SSH Command
- **Utilidades** — Set Variable, Log, Wait, Stop and Fail, Custom JavaScript
- **Nodos Personalizados** — Nodos de proveedores registrados (si los hay)

Para agregar un nodo, **arrástralo** desde la paleta al canvas.

### 3. Canvas (Área Central)

El canvas es el área de trabajo donde posicionas y conectas los nodos. Funcionalidades:

| Acción | Cómo Hacerlo |
|--------|-------------|
| **Mover el canvas** | Haz clic y arrastra el fondo (o usa la rueda del mouse) |
| **Zoom** | Rueda del mouse o pellizco en el trackpad |
| **Seleccionar nodo** | Haz clic en el nodo |
| **Mover nodo** | Arrastra el nodo |
| **Seleccionar múltiples** | Mantén Shift y haz clic, o arrastra un cuadro de selección |
| **Eliminar nodo** | Selecciona y presiona Delete o Backspace |
| **Conectar nodos** | Arrastra desde un handle de salida (●) hacia un handle de entrada (●) |
| **Eliminar conexión** | Haz clic en la conexión y presiona Delete |
| **Pegar flujo** | Ctrl+V con JSON de flujo copiado (p. ej., del Grabador de Chrome) |

### 4. Panel de Propiedades (Lado Derecho)

Al seleccionar un nodo, el panel de propiedades aparece a la derecha con:

- **Ejecutar**: Ejecución del nodo en modo aislado
- **Variables**: Abre las propiedades de nodos anteriores, variables locales y variables globales
- **Configuración**: Campos específicos del tipo de nodo
- **Outputs**: Esquema de salida del nodo (después de la ejecución, muestra los datos reales)
- **Continuar en Fallo**: Toggle para continuar el flujo incluso si el nodo falla

---

## Flujo de Trabajo Típico

1. **Crea el escenario** desde un proyecto
2. **Arrastra los nodos** de la paleta al canvas
3. **Conecta los nodos** arrastrando entre handles
4. **Configura cada nodo** usando el panel de propiedades
5. **Guarda** el flujo (Ctrl+S)
6. **Ejecuta** y verifica los resultados

---

## Orden de Ejecución

QANode ejecuta los nodos en **orden topológico**, determinado por las conexiones:

```
[A] → [B] → [D]
         ↘
[C] ──────→ [E]
```

En este ejemplo:
- `A` se ejecuta primero
- `B` y `C` pueden ejecutarse a continuación (después de `A`)
- `D` se ejecuta después de `B`
- `E` se ejecuta después de `B` y `C`

> **Nota:** Si un nodo tiene múltiples entradas, solo se ejecuta cuando todos los nodos anteriores han completado.

---

## Indicadores Visuales

Después de una ejecución, los nodos muestran indicadores de estado:

| Indicador | Significado |
|-----------|-------------|
| 🟢 Borde verde | Nodo ejecutado con éxito |
| 🔴 Borde rojo | Nodo falló |
| ⚪ Borde gris | Nodo omitido (rama no activada) |
| 🔵 Borde azul | Nodo en ejecución |

---

## Atajos de Teclado

| Atajo | Acción |
|-------|--------|
| `Ctrl+S` | Guardar flujo |
| `Delete` / `Backspace` | Eliminar nodo o conexión seleccionada |
| `Ctrl+C` | Copiar nodo |
| `Ctrl+V` | Pegar nodo (JSON del Grabador de Chrome) |
| `Ctrl+Z` | Deshacer |
| `Ctrl+Shift+Z` | Rehacer |

---

## Cambios Sin Guardar

El editor detecta automáticamente cuando hay cambios sin guardar. Al intentar salir de la página (mediante el botón volver, navegación o cierre de pestaña), se mostrará una advertencia:

> **"¿Salir sin guardar?"**
> Tienes cambios sin guardar. Si sales, tus cambios se perderán.

Esto protege contra la pérdida accidental de trabajo. La advertencia solo aparece cuando hay cambios reales, no al abrir un flujo sin modificarlo.

---

## Próximos Pasos

- [Trabajando con Nodos](trabajando-con-nodos.md) — Detalles sobre cómo agregar, conectar y configurar nodos
- [Ejecución y Depuración](ejecucion-depuracion.md) — Cómo ejecutar flujos y diagnosticar fallos
