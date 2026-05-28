# Extensión Chrome — QANode Recorder

El **QANode Recorder** es una extensión para Google Chrome que graba interacciones con páginas web y las convierte en pasos de prueba compatibles con los nodos **Smart Web Flow**, **Web Flow** o **Smart Locators** de QANode.

---

## Instalación

Instálela desde Chrome Web Store:

[QANode Recorder en Chrome Web Store](https://chromewebstore.google.com/detail/qanode-recorder/gmdklmpafacfkjmljgnoafcjjcdjiajo)

Después de instalar:

1. Fije el ícono de QANode Recorder en Chrome.
2. Abra el sitio que desea grabar.
3. Haga clic en el ícono de la extensión.
4. Elija el modo de grabación.
5. Haga clic en **Start**.

En ambientes de desarrollo interno, la extensión también puede instalarse manualmente por `chrome://extensions` usando **Cargar descomprimida**, cuando el equipo QANode lo indique.

---

## Modo del Recorder

Antes de iniciar la grabación, elija el **modo del recorder** en el popup:

| Modo | Estrategia | Cuándo Usar |
|------|------------|-------------|
| **Smart Web Flow** (recomendado) | Semántica + CSS/XPath + identidad + contexto | Flujos web nuevos, aplicaciones enterprise, menús, modales, grids, iframes, drag/drop y páginas dinámicas |
| **Web Flow** | CSS / XPath | Flujos legados o sitios con `data-testid`, `id` y selectores CSS muy estables |
| **Smart Locators** | Semánticos (`getByRole`, `getByLabel`, etc.) | Flujos semánticos legados y páginas accesibles más simples |
| **XPath Inspector** | Candidatos XPath | Diagnóstico manual de selectores; no genera nodo de flujo |

Cada modo de grabación crea el nodo correspondiente en el canvas al pegar el JSON.

> No es posible cambiar el modo de grabación mientras existen pasos grabados. Cambiar el modo limpia la grabación actual.

Vea también:

- [Smart Web Flow](../nodos/web/smart-web-flow.md)
- [Web Flow](../nodos/web/web-flow.md)
- [Smart Locators](../nodos/web/smart-locators.md)

---

## Modos de Grabación

La extensión opera en modos de interacción durante la grabación:

### Smart Web Capture: Touch Mode e Inspect Mode

Cuando el modo del recorder es **Smart Web Flow**, la extensión permite elegir cómo se capturan los pasos:

| Modo | Cómo Funciona | Cuándo Usar |
|------|---------------|-------------|
| **Touch mode** | Usted usa el sitio normalmente y la extensión graba clics, llenados, scroll, navegación, drag/drop y atajos especiales. | Mejor para grabar flujos rápidos, siguiendo exactamente el camino manual del usuario. |
| **Inspect mode** | La extensión abre paneles flotantes sobre la página. Usted hace clic en un elemento para inspeccionarlo, revisa las acciones sugeridas y elige el paso que desea grabar. | Mejor para crear o corregir pasos con más precisión, principalmente en páginas dinámicas, listas, cards, menús, iframes y elementos parecidos. |

En **Inspect mode**, la página muestra dos paneles:

- **Controls**: muestra los pasos ya grabados, permite pausar, finalizar y copiar el JSON, eliminar pasos, copiar un paso individual y reorganizar la lista.
- **Selected element / Actions**: muestra el elemento seleccionado, las acciones posibles para ese objetivo y los datos capturados por la extensión, como target, estrategias, cantidad de matches, confianza y ancestros.

Al hacer clic en un elemento en Inspect mode, la extensión no asume automáticamente que usted quiere hacer clic en él en el escenario. Primero selecciona el objetivo y muestra acciones como **Click**, **Fill**, **Select**, **Hover**, **Press key**, **Wait visible**, **Assert**, **Extract**, **Extract list**, **Extract table** o **Drag**, según el tipo de elemento detectado.

Esto ayuda a evitar pasos accidentales y deja más claro qué localizadores serán enviados a QANode. Las estrategias mostradas en el panel son las estrategias que la extensión puede enviar en el paso. Puede desmarcar estrategias o ancestros que no desea usar antes de grabar la acción.

El panel también muestra la cantidad de matches cuando es posible. Las estrategias sin un match útil se ignoran para reducir ruido. Si una estrategia aparece con muchos matches, prefiera otra más específica o revise el objetivo antes de grabar la acción.

Para acciones guiadas:

- **Extract list**: seleccione el item repetido; la extensión destaca items parecidos y luego permite elegir los campos que desea extraer dentro del item seleccionado.
- **Drag**: seleccione el item de origen; la extensión mantiene el origen destacado y pide el destino del drop.
- **Press key**: seleccione el objetivo e informe la tecla deseada.
- **Fill**: seleccione el campo e informe el valor a llenar.

### REC — Grabación Normal

Graba interacciones automáticamente:

| Interacción | Acción generada |
|-------------|-----------------|
| **Clic** en elemento | `click` con información del objetivo |
| **Escribir** en campo | `fill`/`type` con texto y objetivo |
| **Seleccionar** opción | `selectOption`/`select` con valor seleccionado |
| **Desplazar** página | `scroll` con posición |
| **Arrastrar y soltar** elemento | `drag` con origen, destino y/o coordenadas |
| **Navegar** a URL | `navigate` con URL |
| **Recargar** página | `refresh` |

En modo **Smart Web Flow**, las acciones grabadas también pueden cargar identidad del objetivo, texto de alcance, estrategias alternativas, contexto de iframe, efectos esperados, evidencias y datos de sesión.

Cuando un clic abre una nueva pestaña/ventana, la extensión puede insertar un paso **Cambiar página / pestaña** (`switchPage`) después del clic para que los siguientes pasos continúen en la página correcta.

### Ctrl+A — Modo Assert

Captura elementos para crear aserciones:

1. Presione **Ctrl+A**.
2. Haga clic en el elemento que desea verificar.
3. Se crea un paso `assert` con el texto/valor del elemento.

### Ctrl+E — Modo Extract

Captura un valor único:

1. Presione **Ctrl+E**.
2. Haga clic en el elemento a extraer.
3. Se crea un paso `extract`.

Cuando el valor del elemento es dinámico, la extensión evita depender solo del texto capturado. Prioriza selectores únicos y, cuando es necesario, graba la posición del item entre los matches visibles para que QANode extraiga el valor futuro, y no solo busque el texto visto durante la grabación.

### Ctrl+Shift+E — Modo Extract List

Captura elementos repetidos como filas de tabla, cards o items de lista:

1. Presione **Ctrl+Shift+E**.
2. El indicador cambia a **LIST:ITEM** — haga clic en el elemento repetido que representa un item de la lista, como una fila de tabla o un card.
3. Overlays cian destacan todos los items detectados con el mismo patrón.
4. El indicador cambia a **LIST:0** — haga clic en los campos que desea extraer dentro de cada item, como columna nombre o columna precio.
5. Cada campo seleccionado se destaca en naranja y se agrega al paso. El contador aumenta con cada campo.
6. Haga clic en el **indicador REC** para finalizar — se graba un paso `extractList` con todos los campos seleccionados.

Presionar **Ctrl+A** o **Ctrl+E** durante la captura de lista cancela el modo y vuelve a la grabación normal.

### Ctrl+Alt+W — Modo Wait

Crea una espera explícita para un elemento de la página:

1. Presione **Ctrl+Alt+W**.
2. Haga clic en el elemento que debe estar visible antes de continuar.
3. Se crea un paso `wait`.

Use este modo cuando la página tiene carga asíncrona, componentes con demora, animaciones o elementos que dependen de respuestas de API.

### Ctrl+Alt+T — Modo Extract Table

Disponible en modo **Smart Web Flow**. Captura una tabla o grid como datos tabulares estructurados:

1. Presione **Ctrl+Alt+T**.
2. Haga clic en la tabla, grid o región tabular.
3. El recorder crea un paso `extractTable`.
4. En QANode, revise el nombre de salida y el límite de filas si es necesario.

Para cards, listas visuales o tablas muy customizadas, **Extract List** puede ser mejor porque permite elegir los campos manualmente.

### XPath Inspector

XPath Inspector ayuda a investigar selectores directamente en la página. No graba un flujo de prueba.

1. Seleccione **XPath Inspector** en el popup.
2. Haga clic en **Activate**.
3. Pase por la página para ver elementos visibles destacados.
4. Haga clic en el elemento a inspeccionar.
5. Revise los candidatos XPath generados y su unicidad.
6. Copie el XPath si es necesario.

El elemento seleccionado parpadea en la página, ayudando a confirmar que el XPath apunta al objetivo correcto.

---

## Grabando una Prueba

1. Haga clic en el ícono de la extensión en la barra de herramientas.
2. Seleccione el **modo del recorder** (Smart Web Flow, Web Flow o Smart Locators).
3. En **Smart Web Flow**, elija **Touch mode** o **Inspect mode**.
4. Haga clic en **Start** para iniciar la grabación.
5. En Touch mode, el indicador queda rojo mostrando grabación activa.
6. Navegue e interactúe con el sitio normalmente o, en Inspect mode, seleccione elementos y elija acciones en los paneles flotantes.
7. Use **Ctrl+A** para agregar verificaciones.
8. Use **Ctrl+E** para agregar extracciones de valor único.
9. Use **Ctrl+Shift+E** para agregar extracciones de lista.
10. Use **Ctrl+Alt+W** para agregar waits explícitos.
11. Use **Ctrl+Alt+T** para agregar extracción de tabla en modo Smart Web Flow.
12. Haga clic nuevamente en el ícono y haga clic en **Stop**, o finalice desde el panel del Inspect mode.

---

## Uso en QANode

1. En el popup de la extensión, haga clic en **Copy JSON**.
2. En QANode, abra el editor de flujos.
3. Presione **Ctrl+V** en el canvas.
4. Se crea un nodo según el modo seleccionado:
   - **Smart Web Flow** → `smartWebSteps`
   - **Web Flow** → `webSteps` con estrategias CSS/XPath
   - **Smart Locators** → `smartSteps` con localizadores semánticos
5. Revise y ajuste los pasos generados cuando sea necesario.

---

## Metadatos del Smart Web Flow

En modo Smart Web Flow, la extensión graba más que un selector. Un paso puede incluir:

- localizador semántico principal;
- selectores CSS/XPath alternativos;
- identidad del elemento;
- texto, role, tag y atributos útiles;
- contexto de fila, card o item repetido;
- información de iframe;
- posición visual aproximada;
- efectos esperados después de la acción;
- configuración de evidencias.

Esto permite que QANode ejecute el paso con más contexto y evite depender solo de un selector frágil.

Cuando la extensión detecta que el objetivo está dentro de una fila, card o item repetido, puede grabar un **Texto de alcance**. QANode lo usa para encontrar primero el item correcto y solo después resolver el botón, enlace, campo o control dentro de él.

Ejemplo:

```
Texto de alcance: INC0010008
Objetivo: botón "Abrir"
```

---

## Arrastrar y Soltar

La extensión graba operaciones de **arrastrar y soltar** en los modos Smart Web Flow, Web Flow y Smart Locators.

Durante la grabación, QANode Recorder intenta capturar:

- el elemento de origen antes de que empiece el movimiento;
- el destino del drop, cuando es identificable;
- coordenadas de origen y destino como fallback;
- metadatos del gesto para reproducir el movimiento con más precisión.

En **Smart Web Flow**, la grabación combina identidad del item arrastrado, identidad del destino, contexto visual/estructural y coordenadas de apoyo cuando sea necesario. Esto ayuda en tableros, columnas kanban, listas reordenables y zonas de drop que cambian durante el gesto.

En algunos sitios, principalmente aplicaciones enterprise con drag/drop muy personalizado, el navegador puede bloquear el movimiento realizado por la propia extensión durante la grabación. En esos casos, la extensión informa que no pudo mover visualmente el item, pero aún puede grabar el paso para que QANode lo ejecute después.

Revise el paso de drag/drop cuando:

- el destino sea un canvas o área libre sin elemento claro;
- existan varios elementos con el mismo texto;
- el elemento sea solo un ícono sin `aria-label`;
- la página mueva o reordene elementos durante el gesto;
- el destino sea una columna vacía, lista virtualizada o área que solo aparece durante el arrastre.

Para aplicaciones enterprise, prefiera agregar `data-testid`, `data-qa`, `aria-label` o nombres accesibles a botones y áreas de drop.

---

## Funciones Inteligentes

### Deduplicación

La extensión evita pasos duplicados:

- **Clic + Navegación**: cuando un clic dispara navegación, el `navigate` extra se suprime.
- **Digitación consecutiva**: entradas repetidas en el mismo campo se consolidan.
- **Scroll**: pequeños desplazamientos se ignoran.

### Formularios

- **Focus out** guarda el valor cuando el campo pierde foco.
- **Enter** guarda el input y registra el clic en el botón submit.
- **Change** guarda valores de dropdown/select inmediatamente.

### Persistencia

La grabación sobrevive a recargas de página:

- los pasos se guardan en `chrome.storage.local`;
- después de recargar, la grabación continúa desde donde quedó;
- se agrega automáticamente un paso `refresh`;
- al eliminar un paso en el popup, el estado interno de la página también se sincroniza para que el paso eliminado no vuelva.

### Sesión del Navegador

Cuando es posible, la extensión captura cookies y localStorage junto con el JSON. Al pegar en QANode, esta sesión puede importarse como storage persistido, evitando login manual antes de la primera ejecución.

Revise sesiones importadas cuando la autenticación expira rápido, depende de MFA/IP/dispositivo o cuando el escenario debe empezar sin cookies.

### Detección de Navegación

La extensión usa APIs del navegador para distinguir:

- **Navegación** a nueva URL → `navigate`;
- **Recarga** de la misma URL → `refresh`;
- **Clic → navegación** → suprime `navigate` redundante;
- **Clic → nueva pestaña** → puede agregar `switchPage`.

### Frames e iFrames

Cuando la acción ocurre dentro de un iframe accesible, la extensión intenta grabar el contexto del frame antes de la acción. Puede usar selector, nombre y URL del frame para que la ejecución vuelva al mismo contexto.

Iframes cross-origin o muy dinámicos pueden requerir revisar el paso `frame` en QANode.

---

## Efectos Esperados

En modo Smart Web Flow, algunos pasos incluyen efectos esperados como `domStable`, `targetVisible`, `overlayOpened` o `pageOpened`.

La UI no expone edición individual de cada efecto esperado. El usuario controla la llave del paso **Usar efectos esperados**. Manténgala activa cuando la acción debe probar que la página cambió, abrió menú, mostró modal, abrió pestaña o dejó visible el siguiente objetivo. Desactívela en pantallas volátiles o cuando un `wait`/`assert` manual sea más adecuado.

---

## Formato de Exportación

El JSON copiado es compatible con el tipo de nodo seleccionado:

- **Smart Web Flow**: `type: "smart-web-flow"` y `smartWebSteps`;
- **Web Flow**: `type: "web-flow"` y `webSteps`;
- **Smart Locators**: `type: "smart-locators"` y `smartSteps`.

El JSON de Smart Web Flow puede incluir campos adicionales de identidad, contexto, efectos esperados y evidencias. Estos campos ayudan a la ejecución. En la UI, normalmente el usuario solo activa o desactiva el uso de efectos esperados del paso.

---

## Flujo de Trabajo Recomendado

```
1. Elija el modo del recorder, preferiblemente Smart Web Flow para flujos nuevos
2. Grabe la interacción con la extensión
3. Pegue en QANode (Ctrl+V)
4. Revise los pasos generados
5. Revise efectos esperados y texto de alcance cuando existan
6. Agregue waits donde sea necesario
7. Agregue asserts para validaciones
8. Ajuste selectores/localizadores si es necesario
9. Configure screenshots en pasos críticos
10. Guarde y ejecute
```

---

## Limitaciones

- **Solo Chrome** — no funciona en otros navegadores.
- **No toda espera es automática** — revise waits en páginas lentas, grids y cargas por API.
- **Sitios con CSP restrictivo** pueden bloquear la inyección del content script.
- **Iframes cross-origin** pueden no ser accesibles.

---

## Consejos

- Use **Smart Web Flow** para flujos nuevos, **Smart Locators** para flujos semánticos legados y **Web Flow** para selectores CSS/XPath tradicionales.
- Grabe acciones simples y refine en QANode.
- Use **Ctrl+A** en puntos de verificación.
- Use **Ctrl+E** para extracción de valor único.
- Use **Ctrl+Shift+E** para elementos repetidos como tablas y listas de cards.
- Use **Ctrl+Alt+W** cuando un elemento necesita aparecer antes del siguiente paso.
- Use **Ctrl+Alt+T** para tablas y grids en Smart Web Flow.
- Revise el **Texto de alcance** en filas, cards y listas.
- Prefiera `data-testid`, `data-qa`, `aria-label` y nombres accesibles cuando pueda influir en la aplicación.
