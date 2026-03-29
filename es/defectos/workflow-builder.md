# Workflow Builder

> **Permiso requerido:** `bug.configure_workflow`

El Workflow Builder es la pantalla donde su organización define cómo fluirán los defectos — qué estados existen, qué transiciones son posibles entre ellos, qué campos aparecen en cada etapa y qué campos personalizados tendrá el módulo.

Acceda en **Configuración → Flujo de Trabajo de Defectos**.

---

## Descripción general de la pantalla

La pantalla está dividida en dos áreas principales:

- **Estados** — lista de todos los estados del flujo de trabajo con sus campos y configuraciones
- **Transiciones** — lista de todas las transiciones posibles entre los estados

Todos los cambios deben guardarse con el botón **Guardar Flujo de Trabajo** para entrar en vigor.

---

## Gestión de Estados

### Crear un estado

1. Haga clic en **Agregar Estado**
2. Defina:
   - **Nombre** — nombre mostrado en la interfaz (ej.: "En Triaje", "En Investigación")
   - **Estado Inicial** — marque si este es el estado donde todo defecto nace al abrirse (solo uno puede ser inicial)
   - **Estado Final** — marque si este estado representa el cierre del defecto (puede haber más de uno)
   - **Cola Predeterminada** — si se especifica, se usa como cola de destino cuando no se envía una elección manual de enrutamiento
   - **Permite asignación manual** — si está activado, el usuario puede elegir libremente el responsable directo al tramitar a este estado

### Editar un estado

Haga clic en el nombre del estado o en el ícono de edición junto a él. Las mismas configuraciones de creación están disponibles para edición.

### Eliminar un estado

Haga clic en el ícono de eliminación junto al estado.

> **Atención:** No es posible eliminar un estado que aún tiene defectos abiertos asociados. Mueva o cierre esos defectos antes de eliminar el estado.

### Reordenar estados

Arrastre los estados por su asa lateral para reorganizar el orden en que aparecen en la interfaz.

---

## Campos del estado

El estado inicial define qué campos aparecen al abrir el defecto. Los formularios de tramitación se definen en las transiciones.

Los campos nativos disponibles son:

| Campo | Descripción |
|-------|-------------|
| Título | Título del defecto |
| Descripción | Descripción detallada |
| Severidad | Nivel de impacto (crítico, alto, medio, bajo) |
| Prioridad | Urgencia de resolución |
| Resolución | Resultado del cierre (ej.: "Corregido", "No Reproducible") |
| Responsable | Usuario asignado al defecto |
| Cola | Equipo responsable del defecto |

Además de los campos nativos, todos los **campos personalizados activos** también están disponibles.

### Visible vs. Obligatorio

Para cada campo puede definir:

- **Visible** — el campo aparece en la interfaz pero no necesita completarse
- **Obligatorio** — el campo debe completarse para que la operación finalice

> **Nota:** Un campo obligatorio siempre aparece en la interfaz, incluso si no está marcado como visible.

---

## Gestión de Transiciones

Una transición define el camino de un estado a otro. Sin una transición creada, no es posible mover un defecto entre dos estados en el flujo normal.

### Crear una transición

1. Haga clic en **Agregar Transición**
2. Defina:
   - **Nombre** — etiqueta mostrada en el botón de tramitación (ej.: "Iniciar Investigación", "Corregir", "Rechazar")
   - **Estado de Origen** — estado actual del defecto
   - **Estado de Destino** — estado al que va el defecto al tramitar

### Campos de la transición

Al igual que en los estados, cada transición puede tener sus propios campos visibles y obligatorios. Estos campos aparecen en el modal de tramitación cuando el usuario hace clic en el botón de transición.

Use campos de transición para capturar información relevante en el momento del cambio — por ejemplo, exigir **Resolución** al cerrar, o una **Cola** al pasar el defecto a otro equipo.

### Eliminar una transición

Haga clic en el ícono de eliminación junto a la transición. Eliminar una transición no afecta los defectos existentes — solo impide que nuevos defectos usen ese camino.

---

## Flujo de trabajo predeterminado

Cuando no existe ningún flujo de trabajo activo en la instancia, QANode crea automáticamente un flujo de trabajo predeterminado con los siguientes estados:

| Estado | Tipo |
|--------|------|
| Abierto | Inicial |
| En Progreso | — |
| Corregido | — |
| Rechazado | — |
| Reabierto | — |
| Cerrado | Final |

Puede usar este flujo de trabajo como punto de partida y ajustarlo según el proceso de su organización.

> **Importante:** El flujo de trabajo predeterminado solo se crea cuando no existe ningún flujo activo. Si ya tiene un flujo configurado, el predeterminado nunca sobrescribirá el suyo.

---

## Campos Personalizados

Los campos personalizados permiten que su organización agregue información específica al proceso de defectos — como número de ticket externo, sistema afectado, versión del build, entre otros.

Acceda en la propia pantalla del **Workflow Builder**, en la sección **Campos Personalizados**.

### Crear un campo personalizado

1. Haga clic en **Agregar Campo**
2. Defina:
   - **Nombre** — identificador interno del campo (sin espacios, ej.: `ticket_externo`)
   - **Etiqueta** — nombre mostrado en la interfaz (ej.: "Ticket Jira")
   - **Tipo** — tipo de valor (texto, número, fecha, selección)
   - **Activo** — si el campo está disponible para usar

### Usando campos personalizados en el flujo de trabajo

Después de crear un campo personalizado y dejarlo activo, queda disponible para agregarse como visible u obligatorio en cualquier estado o transición del flujo de trabajo.

### Desactivar vs. eliminar

- **Desactivar** — el campo deja de aparecer en la interfaz pero los datos ya guardados se conservan
- **Eliminar** — el campo se elimina permanentemente

> **Atención:** No es posible desactivar o eliminar un campo que esté en uso por un flujo de trabajo activo. Elimine el campo del flujo de trabajo antes de desactivarlo.

---

## Buenas prácticas

- **Empiece simple** — un flujo de trabajo con 4 o 5 estados y transiciones directas es más fácil de operar que uno con muchas desviaciones
- **Use cola predeterminada cuando quiera dirigir la etapa a un equipo específico** — esto ayuda a sugerir la cola correcta cuando la transición no envía otra cola manualmente
- **Exija resolución solo al cierre** — campos como "Resolución" tienen más sentido como obligatorios en la transición de cierre
- **Evite estados huérfanos** — todo estado (excepto los finales) debe tener al menos una transición de salida, de lo contrario los defectos pueden quedar atascados
