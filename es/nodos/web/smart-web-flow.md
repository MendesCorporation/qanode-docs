# Nodo Smart Web Flow

El nodo **Smart Web Flow** es el nodo recomendado para automatización web en QANode. Combina grabación por extensión Chrome, localizadores semánticos, selectores CSS/XPath, contexto de página, validaciones de efecto esperado, evidencias y mecanismos conservadores de recuperación para ejecutar escenarios en aplicaciones web modernas.

Fue diseñado para sistemas enterprise y páginas dinámicas como CRMs, ERPs, ServiceNow, Salesforce, SAP, Jira, portales internos, aplicaciones con iframes, menús, grids, kanban, componentes ricos e interfaces que cambian partes del DOM entre ejecuciones.

> **Recomendación:** Para nuevos escenarios web, prefiera **Smart Web Flow**. [Web Flow](web-flow.md) y [Smart Locators](smart-locators.md) siguen disponibles por compatibilidad y casos específicos.

---

## Visión General

[Visión General](../../assets/images/nos-smart-web-flow-visao-geral.mp4)

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `smart-web-flow` |
| **Categoría** | Web |
| **Entrada** | `in` |
| **Salida** | `out` |
| **Grabación recomendada** | QANode Recorder en modo Smart Web Flow |
| **Localización de elementos** | Semántica + CSS/XPath + identidad del objetivo + contexto |
| **Uso recomendado** | Flujos web nuevos, aplicaciones dinámicas, evidencias y recuperación asistida |

---

## Cuándo Usar

Use **Smart Web Flow** cuando:

- el flujo será grabado con la extensión Chrome;
- la pantalla tiene menús, dropdowns, modales, iframes, listas, tablas, grids o cards;
- los elementos pueden cambiar de posición entre ejecuciones;
- necesita screenshots por paso;
- desea validaciones automáticas después de acciones importantes;
- desea recibir sugerencias de mejora después de ejecuciones exitosas;
- el mismo flujo debe ejecutarse con datos variables, como número de ticket, cliente, card, item o registro.

Considere otro nodo cuando:

- ya existe un Web Flow legado simple y estable;
- necesita un selector CSS/XPath manual muy específico y no quiere usar metadatos Smart Web;
- la automatización no es web, como API, base de datos o mobile nativo.

---

## Configuración General

| Campo | Tipo | Predeterminado | Descripción |
|-------|------|----------------|-------------|
| **Modo de Sesión** | `new` / `reuse` | `new` | Abre una nueva sesión de navegador o reutiliza una existente |
| **Session ID** | `string` | — | ID usado cuando el modo de sesión es `reuse` |
| **Headless** | `boolean` | `true` | Ejecuta el navegador sin interfaz visible |
| **Estrategia de Storage** | `inMemory` / `persisted` | `inMemory` | Controla si cookies/localStorage se descartan o se persisten |
| **Clave de Storage** | `string` | — | Identificador usado para guardar/reutilizar storage persistido |
| **Browser** | `chromium` / `firefox` / `webkit` | `chromium` | Navegador usado en la ejecución |
| **Dispositivo** | `desktop` / `mobile` | `desktop` | Viewport desktop o emulación mobile |
| **Viewport** | preset o `custom` | `1920x1080` | Tamaño de ventana en modo desktop |
| **Self Healing** | `boolean` | `true` | Permite recuperación conservadora cuando el objetivo original no se encuentra |
| **Escaneo de accesibilidad** | `boolean` | `false` | Ejecuta análisis de accesibilidad y guarda evidencia |
| **Auditoría de performance** | `boolean` | `false` | Colecta métricas de navegación, red y APIs por pantalla |

Para flujos que exigen login, MFA o SSO, use la extensión Chrome para capturar la sesión y configure storage persistido. Use `inMemory` para pruebas aisladas y `persisted` para aplicaciones que necesitan mantener autenticación entre ejecuciones.

---

## Cómo Encuentra Elementos

Cada paso puede cargar varias pistas para encontrar el objetivo correcto. En vez de depender de un único selector, Smart Web Flow combina señales diferentes:

| Señal | Ejemplo | Cuándo ayuda |
|-------|---------|--------------|
| **Localizador semántico** | `getByRole`, `getByLabel`, `getByText` | Botones, enlaces, campos, opciones y elementos accesibles |
| **Selectores CSS/XPath** | `#email`, `[data-testid="submit"]`, XPath | Páginas con IDs, atributos estables o estructura conocida |
| **Identidad del objetivo** | texto, role, tag, atributos, posición aproximada | Confirma que el selector encontrado aún representa el mismo elemento |
| **Contexto de fila/card** | texto de alcance, item padre, fila de tabla | Pantallas con elementos repetidos |
| **Contexto de control** | label cercano, valor esperado, tipo del control | Inputs customizados, combos, calendarios y checkboxes estilizados |
| **Frame/Iframe** | contexto de frame grabado | Aplicaciones que renderizan contenido dentro de iframes |

El objetivo es ser resiliente sin hacer clic en “cualquier cosa parecida”. Cuando la pantalla es ambigua, Smart Web Flow tiende a fallar con diagnóstico en vez de elegir un objetivo inseguro.

---

## Texto de Alcance

El **Texto de alcance** aparece cuando el paso fue grabado dentro de un item repetido, como:

- fila de tabla;
- card de kanban;
- item de lista;
- bloque de resultado;
- contenedor con varios botones iguales.

Indica a QANode **en qué fila/card/lista debe buscar antes de resolver el objetivo**.

Ejemplo:

| Ticket | Cliente | Acción |
|--------|---------|--------|
| INC0010001 | Apptrix | Abrir |
| INC0010002 | QANode | Abrir |

Si el objetivo es el botón **Abrir** de la fila `INC0010002`, el localizador del botón no es suficiente. El texto de alcance puede ser:

```
INC0010002
```

QANode busca primero la fila/card que contiene ese texto y solo después resuelve el elemento dentro de ese alcance.

El texto de alcance acepta expresiones:

```
{{ variables.numeroTicket }}
{{ steps["Crear ticket"].outputs.extracts.numero }}
```

Buenas prácticas:

- use un texto corto y único dentro del item, como número, código, nombre o título;
- evite usar el texto completo de un card grande si una parte menor identifica el item;
- para datos variables, use `{{ }}` en vez de dejar el valor grabado fijo;
- cuando sea posible, pida al equipo de desarrollo que exponga `data-testid`, `data-qa`, labels o nombres accesibles.

---

## Orden de Estrategias

En una ejecución normal, Smart Web Flow intenta resolver el paso usando las estrategias grabadas y los metadatos disponibles:

1. estrategia preferida aprendida, cuando fue validada por ejecuciones exitosas anteriores;
2. localizadores semánticos, cuando aplican;
3. selectores CSS/XPath registrados;
4. contexto de fila/card/control cuando el objetivo es repetido o customizado;
5. recuperación conservadora cuando **Self Healing** está habilitado;
6. falla con diagnóstico cuando no hay confianza suficiente.

El usuario normalmente no necesita configurar este orden manualmente. Lo importante es revisar si el paso grabado representa la intención correcta: acción, objetivo, texto de alcance, datos, evidencia y efecto esperado.

---

## Efectos Esperados

Algunos pasos grabados tienen **Efectos esperados** generados automáticamente por la extensión o por el motor. Describen lo que QANode observó como resultado probable de una acción.

| Efecto | Qué valida |
|--------|------------|
| `domStable` | La página quedó estable por un corto período |
| `networkIdle` | La red quedó ociosa dentro del límite configurado |
| `urlContains` | La URL contiene un fragmento esperado |
| `titleContains` | El título de la página contiene un fragmento esperado |
| `elementVisible` | Un selector específico quedó visible |
| `targetVisible` | El próximo objetivo esperado quedó visible |
| `overlayOpened` | Un menú, dropdown, modal u overlay abrió |
| `pageOpened` | Una nueva pestaña/ventana fue abierta |

### Cómo Interpretar

En la UI, el usuario **no configura cada efecto esperado individualmente**. El control disponible es la llave **Usar efectos esperados**, que activa o desactiva la validación automática de efectos para ese paso.

Mantenga **Usar efectos esperados** activo cuando la acción debe probar que la pantalla cambió:

- un clic abre un menú;
- un botón abre un modal;
- una acción navega a otra página;
- una acción abre nueva pestaña;
- un hover revela un submenú;
- un clic debe dejar visible el siguiente campo.

Desactívelo en ese paso cuando:

- la página es muy dinámica y el efecto apunta a contenido volátil;
- el efecto depende de datos que siempre cambian;
- el mismo contenedor abre, pero el siguiente objetivo real depende de otro paso;
- el paso ya tiene una espera explícita más adecuada.

Para control manual, use `wait` y `assert`. Son más adecuados para esperas y validaciones definidas directamente por el usuario.

Los efectos esperados grabados por Smart Web Flow empiezan de forma conservadora. Cuando QANode observa el mismo escenario ejecutándose con éxito repetidas veces y el mismo objetivo confirmado, puede convertir esa pista en una validación más fuerte. Esto evita que un timing accidental o cambio temporal de DOM rompa escenarios demasiado pronto.

---

## Self Healing

Cuando **Self Healing** está habilitado, Smart Web Flow puede recuperar el objetivo si las estrategias grabadas ya no encuentran el elemento.

Puede ayudar cuando:

- el texto cambió levemente;
- un botón cambió de posición;
- un ID dejó de existir, pero el elemento mantiene el mismo role;
- un enlace cambió el texto, pero mantuvo contexto y destino similares;
- un campo fue renderizado por otro componente visual.

Evita aplicar recuperación cuando:

- existen muchos candidatos parecidos;
- la acción puede ser destructiva y el objetivo no está claro;
- el contexto cambió de forma incompatible;
- el mejor candidato no es claramente mejor que el segundo.

Cuando se usa recuperación, la ejecución la muestra como **Self-healing aplicado**. El usuario puede revisar y aplicar al flujo cuando tenga sentido.

Si el selector recuperado ya existe en el paso, QANode suprime sugerencias innecesarias de “aplicar optimización” para no pedir al usuario que aplique algo que ya tiene en la práctica.

---

## Estrategia Preferida Aprendida

Después de ejecuciones exitosas, Smart Web Flow puede aprender qué estrategia resolvió el paso con confiabilidad. Guarda información de éxito solo cuando la ejecución pasó. Ejecuciones fallidas no promocionan ni sobrescriben la estrategia preferida.

Esto ayuda a la resiliencia: una estrategia buena pero a veces más lenta no se reemplaza solo porque otra tentativa apareció después de un cambio de timing. En una ejecución futura, QANode puede intentar la estrategia preferida con polling rápido durante su ventana promedio aprendida y luego seguir con la lista normal si no resuelve.

---

## Acciones

| Acción | Descripción |
|--------|-------------|
| `navigate` | Navegar a una URL |
| `switchPage` | Cambiar a otra página/pestaña/ventana |
| `click` | Clicar un elemento |
| `dblclick` | Doble clic en un elemento |
| `fill` | Completar un campo instantáneamente |
| `type` | Digitar texto carácter por carácter |
| `check` | Marcar checkbox |
| `uncheck` | Desmarcar checkbox |
| `selectOption` | Seleccionar opción en select nativo o control compatible |
| `hover` | Mover el mouse sobre un elemento |
| `focus` | Enfocar un elemento |
| `press` | Presionar una tecla |
| `drag` | Arrastrar y soltar un elemento |
| `wait` | Esperar tiempo, selector, URL o condición |
| `scroll` | Desplazar la página o región con scroll |
| `refresh` | Recargar la página actual |
| `clock` | Controlar el tiempo del navegador para escenarios sensibles a fecha/hora |
| `frame` | Cambiar o limitar ejecución a un iframe |
| `assert` | Validar texto, visibilidad, URL, título o valor |
| `extract` | Extraer un valor único |
| `extractList` | Extraer items repetidos como filas o cards |
| `extractTable` | Extraer tabla/grid como filas estructuradas |
| `upload` | Subir archivos |
| `dialog` | Manejar diálogos del navegador |
| `setSlider` | Definir valor de slider/range |
| `evaluate` | Ejecutar JavaScript controlado en la página |
| `screenshot` | Capturar evidencia |

---

## Acciones Importantes

### navigate

Navega a una URL.

| Campo | Descripción |
|-------|-------------|
| **URL** | URL de destino, con soporte para expresiones `{{ }}` |
| **Esperar Hasta** | Evento de carga esperado |
| **Timeout** | Tiempo máximo de navegación |

### switchPage

Cambia la página activa cuando un clic o acción abre otra pestaña/ventana.

| Modo | Descripción |
|------|-------------|
| `last` | Usa la última pestaña abierta |
| `next` | Usa la siguiente pestaña en relación con la actual |
| `previous` | Usa la pestaña anterior |
| `match` | Busca una pestaña por fragmento de URL y/o título |

La extensión Chrome puede insertar este paso automáticamente cuando detecta un enlace que abre nueva pestaña.

### fill y type

Use `fill` para la mayoría de los campos. Use `type` cuando la aplicación depende de eventos reales de teclado, máscaras, autocomplete o comportamiento de rich text.

Para editores customizados, Smart Web Flow puede apuntar a regiones editables como `contenteditable`, textboxes renderizados por editores ricos o campos donde la extensión capturó el objetivo real de escritura.

### drag

Drag/drop guarda información de origen y destino. Cuando es posible, el recorder captura un contenedor de destino en vez de un card vecino volátil, mejorando el comportamiento en kanbans y listas reordenables. Si el destino es un área libre, canvas o zona de drop que solo existe mientras se arrastra, revise el paso manualmente.

Si la extensión avisa que no pudo mover el item durante la grabación, aun así revise el paso en QANode: en sitios muy personalizados, la ejecución puede reproducir el drag/drop incluso cuando la extensión no consigue mover visualmente el card.

### extract

Extrae un valor único de la página.

| Campo | Descripción |
|-------|-------------|
| **Nombre** | Nombre del output |
| **Atributo** | `text`, `inputValue`, `href`, `src` o un atributo específico |

Cuando el paso viene de la extensión, Smart Web Flow trata la extracción como valor dinámico. El texto visto durante la grabación se usa como muestra/evidencia, pero el objetivo debe encontrarse por un selector estable, por contexto o por la posición correcta entre elementos equivalentes.

Esto ayuda en pantallas donde el valor puede cambiar en cada ejecución, por ejemplo contadores, estados, saldos, totales e indicadores.

El valor extraído queda disponible en:

```
{{ steps["Nombre del Nodo"].outputs.extracts.nombre }}
```

### clock

La acción `clock` controla el tiempo del navegador para escenarios que dependen de fecha/hora actual. Es útil para date pickers, expiración, reglas de SLA, banners por tiempo o pruebas que deben ser determinísticas.

---

## Evidencias

Cada paso puede capturar screenshots:

| Modo | Comportamiento |
|------|----------------|
| `before` | Captura antes de la acción |
| `after` | Captura después de la acción |
| `both` | Captura antes y después |
| `none` | Sin screenshot para el paso |

Use screenshots en puntos críticos: login, navegación, apertura de modal, envío de formulario, drag/drop, aserciones importantes y fallas que requieren diagnóstico visual.

---

## Outputs

Outputs típicos incluyen:

| Output | Descripción |
|--------|-------------|
| `success` | Si el nodo completó con éxito |
| `steps` | Detalles de ejecución paso a paso |
| `extracts` | Valores capturados por acciones extract |
| `tables` | Datos capturados por extractTable |
| `screenshots` | Archivos/URLs de evidencia |
| `sessionId` | ID de sesión del navegador, útil para reutilización |
| `currentUrl` | URL al final del nodo |

Ejemplo de acceso:

```
{{ steps["smart-web-flow"].outputs.extracts.customerName }}
{{ steps["smart-web-flow"].outputs.tables.orders.rows[0].status }}
{{ steps["smart-web-flow"].outputs.sessionId }}
```

---

## Flujo de Trabajo Recomendado

1. Grabe con QANode Recorder en modo **Smart Web Flow**.
2. Elija el modo de captura:
   - **Touch mode** para grabar interactuando normalmente con la página;
   - **Inspect mode** para seleccionar elementos, revisar acciones sugeridas y grabar pasos con más control.
3. Ejecute el flujo manualmente en el sitio o use los paneles de Inspect mode para montar las acciones.
4. Use los atajos de grabación cuando sea necesario: `Ctrl+A` para assert, `Ctrl+E` para extract, `Ctrl+Shift+E` para extract list, `Ctrl+Alt+W` para wait y `Ctrl+Alt+T` para extract table.
5. Detenga la grabación o finalice desde el panel de Inspect mode.
6. Pegue el JSON en el canvas de QANode.
7. Revise las acciones generadas.
8. Confirme el texto de alcance en filas/cards repetidos.
9. Mantenga efectos esperados activos en pasos donde la pantalla debe cambiar.
10. Agregue `wait` o `assert` explícitos para validaciones críticas de negocio.
11. Use variables para datos dinámicos.
12. Ejecute una vez e inspeccione evidencias.
13. Aplique solo sugerencias que coincidan con la intención real del usuario.

---

## Buenas Prácticas

- Prefiera texto de alcance corto y estable para items repetidos.
- Use variables en el texto de alcance cuando el objetivo cambia por datos de prueba.
- Mantenga efectos esperados activos para menús, modales, cambios de página y nuevas pestañas.
- Desactive efectos esperados solo en pasos volátiles o cuando ya agregó una espera explícita mejor.
- Use `fill` para inputs normales y `type` para máscaras, autocomplete y rich text.
- Use `switchPage` después de acciones que abren nuevas pestañas.
- En aplicaciones enterprise, pida al equipo de desarrollo `data-testid`, `data-qa`, `aria-label` y nombres accesibles.
- Revise pasos de drag/drop en kanban, listas virtuales y tableros dinámicos.

---

## Limitaciones

- Iframes cross-origin pueden limitar grabación o ejecución.
- Controles basados solo en canvas pueden requerir coordenadas o lógica customizada.
- Listas virtualizadas muy dinámicas pueden necesitar texto de alcance y waits explícitos.
- Self Healing es conservador y puede fallar cuando la UI está ambigua.
- Los efectos esperados son generados automáticamente; el usuario puede activarlos/desactivarlos, pero no configurar cada efecto interno desde la UI.
