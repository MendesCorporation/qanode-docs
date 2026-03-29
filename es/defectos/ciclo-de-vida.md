# Ciclo de Vida del Defecto

Esta página describe el camino completo de un defecto — desde la apertura hasta el cierre — incluyendo cómo funcionan la asignación, las colas, el claim y la tramitación en la práctica.

---

## 1. Apertura de un defecto

### Desde una ejecución fallida

Esta es la forma más común de abrir un defecto. Cuando una ejecución termina con fallo:

1. Acceda al detalle de la ejecución
2. Haga clic en **Abrir Defecto**
3. Complete los campos requeridos por el flujo de trabajo (definidos en el estado inicial)
4. Haga clic en **Guardar**

El defecto se crea con un vínculo directo a la ejecución, al flujo y, cuando aplica, al paso que originó el fallo. Este vínculo se conserva aunque el flujo sea editado después — el defecto guarda una foto del estado original del escenario.

> **Permiso requerido:** `bug.create`

### Manualmente

Para abrir un defecto sin vínculo con una ejecución:

1. Acceda a **Defectos** en el menú lateral
2. Haga clic en **Nuevo Defecto**
3. Complete los campos obligatorios
4. Haga clic en **Guardar**

---

## 2. Identificación del defecto

Cada defecto recibe automáticamente:

- **Bug Key** — código único de referencia en el formato `BUG-N` (ej.: `BUG-42`), útil para mencionar en comentarios, correos o sistemas externos
- **Fecha de creación** y **reportador** — quién lo abrió y cuándo

---

## 3. Asignación — Colas y Responsables

Después de abrirse, un defecto necesita ser dirigido a alguien para trabajarlo. Esto puede ocurrir de dos formas:

### Cola

Una cola representa un equipo. Cuando un defecto está en una cola:
- Cualquier miembro de esa cola con `bug.assign` puede **hacer claim** y tomar posesión del defecto
- El defecto es visible para todo el equipo de la cola

Las colas pueden aplicarse automáticamente cuando el flujo de trabajo define una **cola predeterminada** para un estado — al tramitar a ese estado, el defecto es automáticamente dirigido a la cola configurada.
Las colas pueden aplicarse automáticamente cuando el flujo de trabajo define una **cola predeterminada** para un estado — en ausencia de una elección manual de enrutamiento, el defecto se dirige a la cola configurada.

### Responsable directo

Cuando un defecto es asignado directamente a un usuario:
- Solo ese usuario puede tramitar en el flujo normal
- Otros usuarios, incluso de la misma cola, no pueden hacer claim en el flujo normal mientras exista un responsable directo

### Sin asignación

Un defecto sin cola y sin responsable puede ser claimado por cualquier usuario con `bug.assign`.

---

## 4. Claim — Tomando posesión

**Claim** es el acto de poner el defecto a su nombre para trabajar en él. La tramitación normal requiere que el defecto esté claimado por el usuario que va a tramitar.

Para hacer claim:

1. Abra el detalle del defecto
2. Haga clic en **Asumir**

### ¿Cuándo puedo hacer claim?

| Situación del defecto | Quién puede hacer claim |
|-----------------------|------------------------|
| En cola, sin responsable | Cualquier miembro de esa cola con `bug.assign` |
| Sin cola y sin responsable | Cualquier usuario con `bug.assign` |
| Con responsable directo | Nadie en el flujo normal; solo quien tiene `bug.transition_any` puede actuar por bypass administrativo |
| En estado final | Nadie — el estado final bloquea el claim |

### Qué hace el claim

- Pone su usuario como responsable directo del defecto
- Mantiene la cola asociada (el defecto continúa vinculado al equipo)

### Qué no hace el claim

El claim no elimina la cola asociada al defecto. Sin embargo, si existe un responsable directo, esa posesión pasa a prevalecer en el flujo normal. En la práctica, otros miembros de la misma cola ya no pueden hacer claim ni tramitar el defecto mientras esté asignado a alguien.

---

## 5. Tramitación — Cambiando el estado

Tramitar significa mover el defecto de un estado a otro siguiendo las transiciones definidas en el flujo de trabajo.

### Cómo tramitar

1. Abra el detalle del defecto
2. Los botones de tramitación disponibles aparecen según las transiciones que parten del estado actual
3. Haga clic en la transición deseada (ej.: "Iniciar Investigación", "Corregir", "Cerrar")
4. Complete los campos obligatorios de la transición (si los hay)
5. Agregue una nota opcional
6. Confirme

### Reglas de tramitación

**Para tramitar en el flujo normal (`bug.transition`):**
- El defecto debe estar claimado por su usuario
- Si el defecto está solo en cola (sin responsable directo), debe hacer claim primero
- Si el defecto pertenece a otro responsable, no puede tramitar

**Para tramitar con bypass administrativo (`bug.transition_any`):**
- Puede tramitar a cualquier estado, independientemente de quién sea el responsable
- No necesita claim previo
- Útil para gerentes y administradores del proceso

### Qué ocurre después de tramitar

- El estado del defecto se actualiza
- Si la transición no informa manualmente un nuevo responsable o una nueva cola, y el estado de destino tiene una **cola predeterminada**, el defecto se dirige a ella
- Si el estado de destino es **final**, la fecha de cierre se registra
- Si el defecto regresa de un estado final a un estado activo, la fecha de cierre se elimina
- La transición genera un evento en el **historial** del defecto con la nota proporcionada

---

## 6. Situaciones prácticas

### Defecto detenido en cola de QA, sin responsable

1. Un usuario de QA con `bug.assign` hace claim → el defecto queda a su nombre
2. Investiga y hace clic en "Corregir" → el defecto pasa al siguiente estado
3. Si el siguiente estado tiene cola predeterminada de Dev, el defecto se dirige automáticamente a la cola de Dev

### Defecto con responsable directo

1. Solo ese responsable puede tramitar en el flujo normal
2. Otro usuario con `bug.transition` no puede hacer claim ni tramitar mientras ya exista un responsable directo
3. Un gerente con `bug.transition_any` puede avanzar el defecto sin restricción

### Defecto cerrado siendo reabierto

1. Un usuario con `bug.transition_any` tramita de vuelta a un estado activo (ej.: "Reabierto")
2. La fecha de cierre se elimina automáticamente
3. El historial registra la reapertura con quién lo hizo y cuándo

---

## 7. Lista de defectos

La pantalla de listado de defectos muestra todos los defectos de la instancia con filtros por:

- **Estado** — filtre por uno o más estados del flujo de trabajo
- **Severidad** — crítico, alto, medio, bajo
- **Prioridad** — urgencia del defecto
- **Cola** — equipo responsable
- **Responsable** — usuario asignado

Cada fila de la lista muestra el Bug Key, título, estado, severidad, prioridad, responsable y fecha de creación. Haga clic en cualquier defecto para abrir el detalle.

---

## 8. Detalle del defecto

La pantalla de detalle del defecto reúne todo en un solo lugar:

| Sección | Qué muestra |
|---------|-------------|
| **Encabezado** | Bug Key, título, estado actual, reportador y fecha |
| **Información** | Severidad, prioridad, responsable, cola, proyecto y ejecución de origen |
| **Campos personalizados** | Campos adicionales definidos por la organización |
| **Transiciones** | Botones para tramitar al siguiente estado |
| **Sandbox** | Botón para abrir investigación en entorno aislado |
| **Comentarios** | Discusión sobre el defecto |
| **Adjuntos** | Archivos enviados al defecto |
| **Historial** | Rastro completo de todos los cambios y transiciones |
