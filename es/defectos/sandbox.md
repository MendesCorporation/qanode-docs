# Sandbox de Investigación

> **Permiso requerido:** `bug.run_sandbox`

El sandbox es un entorno de investigación aislado vinculado a un defecto. Permite ejecutar el flujo original que causó el fallo, observar el comportamiento y realizar ajustes — sin afectar los datos oficiales, sin contaminar los informes y sin interferir con las ejecuciones reales del proyecto.

---

## Qué es el sandbox

Cuando se abre un defecto desde una ejecución fallida, QANode conserva una **foto del flujo original** en el momento en que se creó el defecto. El sandbox parte siempre de esa foto, no del flujo actual.

Esto significa que:

- Aunque alguien edite o elimine el flujo original después, el sandbox sigue basándose en el estado exacto que generó el problema
- La investigación parte siempre del escenario que reproduce el fallo, no de una versión más nueva que puede haber cambiado

---

## Abriendo el sandbox

1. Acceda al detalle del defecto
2. Haga clic en **Abrir Sandbox**
3. QANode crea una copia del flujo original marcada como sandbox
4. Se le redirige al editor con ese flujo

Si ya existe un sandbox activo para el defecto, el sistema reabre el existente en lugar de crear uno nuevo.

---

## Lo que puede hacer en el sandbox

Dentro del sandbox tiene acceso al editor de flujos completo y a accesos directos al defecto:

- **Ejecutar** el flujo para reproducir el fallo
- **Inspeccionar** los nodos y ver los resultados paso a paso
- **Modificar** el flujo para probar hipótesis de corrección
- **Volver a ejecutar** las veces que necesite
- **Agregar un comentario en el defecto** sin salir del sandbox (botón 💬 en la barra de herramientas)
- **Agregar un adjunto en el defecto** sin salir del sandbox (botón 📎 en la barra de herramientas)

### Comentario directo en el defecto

Haga clic en el botón de **comentario** (ícono de globo) en la barra de herramientas. Se abre un modal para escribir y guardar el comentario directamente en el defecto vinculado al sandbox — sin necesidad de volver a la pantalla del defecto.

Úselo para registrar hallazgos mientras aún está investigando: qué reprodujo, qué probó, qué encontró.

### Adjunto directo en el defecto

Haga clic en el botón de **clip** en la barra de herramientas. Se abre un modal con una zona de carga — pegue una captura de pantalla con Ctrl+V o haga clic para seleccionar un archivo. El archivo se envía directamente al defecto vinculado. Útil para adjuntar capturas de pantalla, logs o evidencias recopiladas durante la investigación, sin salir del sandbox.

Todas las ejecuciones realizadas dentro del sandbox:
- Se marcan como `sandbox` internamente
- **No aparecen en informes y dashboards oficiales**
- **No cuentan en los KPIs del proyecto**
- **No afectan el flujo original**

---

## Quién puede abrir el sandbox

El acceso al sandbox sigue las mismas reglas de posesión del defecto:

| Situación del defecto | Quién puede abrir sandbox |
|-----------------------|--------------------------|
| En cola, sin responsable | Cualquier miembro de esa cola con `bug.run_sandbox` |
| Asignado directamente a un usuario | Solo ese usuario en el flujo normal |
| En cola de otro equipo | Usuarios de fuera de la cola no pueden en el flujo normal |
| En estado final | Nadie — el sandbox está bloqueado en estado final |
| Usuario con `bug.transition_any` | Puede bypassar la posesión, pero aun así necesita `bug.run_sandbox` |

---

## Descartando el sandbox

Cuando la investigación termine:

1. Acceda al detalle del defecto
2. Haga clic en **Descartar Sandbox**
3. Confirme la acción

El descarte:
- Elimina el flujo sandbox
- Elimina todas las ejecuciones sandbox vinculadas
- Registra el evento en el historial del defecto

El descarte es definitivo — el sandbox eliminado no puede recuperarse. Si necesita investigar nuevamente, se puede crear un nuevo sandbox en cualquier momento (mientras el defecto no esté en estado final).

---

## Diferencia entre sandbox y ejecución oficial

| | Ejecución oficial | Ejecución sandbox |
|-|-------------------|-------------------|
| Aparece en informes | ✅ Sí | ❌ No |
| Cuenta en KPIs del proyecto | ✅ Sí | ❌ No |
| Basada en flujo actual | ✅ Sí | ❌ Usa foto del original |
| Se puede modificar libremente | ❌ No | ✅ Sí |

---

## Buenas prácticas

- **No descarte el sandbox antes de documentar** — si encontró la causa del fallo, agregue un comentario en el defecto con sus conclusiones antes de descartar
- **Use el sandbox para reproducir antes de corregir** — confirmar que puede reproducir el fallo en el sandbox es el primer paso de una buena investigación
- **Un sandbox a la vez** — el sistema reutiliza el sandbox existente. Si necesita un estado limpio, descarte el actual y abra uno nuevo
