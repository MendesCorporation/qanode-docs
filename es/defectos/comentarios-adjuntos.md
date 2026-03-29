# Comentarios, Adjuntos e Historial

> **Permiso requerido:** `bug.view`

Toda la colaboración y todo el movimiento de un defecto quedan registrados en su detalle. Esta página explica cómo usar comentarios, adjuntos e historial para seguir y documentar el ciclo de vida de un defecto.

---

## Comentarios

Los comentarios son el espacio de discusión del defecto. Úselos para:

- Registrar hallazgos durante la investigación
- Comunicar bloqueos al equipo
- Documentar hipótesis probadas
- Coordinar traspasos entre equipos

### Cómo comentar

1. Acceda al detalle del defecto
2. Desplácese hasta la sección **Comentarios**
3. Escriba su mensaje en el campo de texto
4. Haga clic en **Comentar**

El comentario se guarda con su nombre, foto de perfil y fecha/hora. Además de aparecer en la sección de comentarios, también genera un evento en el **historial** del defecto.

---

## Adjuntos

Los adjuntos permiten incluir archivos relevantes directamente en el defecto — capturas de pantalla, logs, videos de reproducción, informes de error, entre otros.

### Cómo adjuntar un archivo

1. Acceda al detalle del defecto
2. Desplácese hasta la sección **Adjuntos**
3. Haga clic en **Agregar Adjunto** o arrastre el archivo al área indicada
4. El archivo se carga y queda disponible para todos con acceso al defecto

### Eliminación de adjuntos

La eliminación de adjuntos sigue una regla de autoría:

| Quién puede eliminar | Condición |
|---------------------|-----------|
| El propio usuario que lo envió | Puede eliminar cualquier archivo que él mismo envió |
| Admin / Super Admin con `bug.delete_attachment_any` | Puede eliminar cualquier adjunto de cualquier persona |
| Otros usuarios | No pueden eliminar archivos enviados por terceros |

Para eliminar:
1. Localice el adjunto en la sección **Adjuntos**
2. Haga clic en el ícono de eliminación junto al archivo
3. Confirme la acción

---

## Historial

El historial es el rastro auditable completo del defecto. Cada evento registrado muestra quién lo hizo, qué hizo y cuándo.

### Qué genera un evento en el historial

| Acción | Evento generado |
|--------|----------------|
| Defecto abierto | "Defecto creado por [usuario]" |
| Transición de estado | "Estado cambiado de [X] a [Y] por [usuario]" + nota de transición |
| Claim | "Asumido por [usuario]" |
| Cambio de responsable | "Responsable cambiado a [usuario]" |
| Cambio de cola | "Cola cambiada a [cola]" |
| Comentario agregado | "Comentario de [usuario]" |
| Sandbox creado | "Sandbox de investigación creado" |
| Sandbox descartado | "Sandbox de investigación descartado" |
| Adjunto agregado | Genera un evento en el historial con el nombre del archivo; el archivo también aparece en la sección **Adjuntos** |
| Adjunto eliminado | Genera un evento en el historial con el nombre del archivo eliminado; la eliminación también afecta la sección **Adjuntos** |

### Cómo acceder al historial

El historial está en la sección **Historial** en el detalle del defecto, en orden cronológico. Puede ver toda la línea de tiempo del defecto desde la creación hasta el momento actual.

---

## Nota de transición

Al tramitar un defecto (cambiar el estado), puede agregar una **nota de transición**. Esta nota:

- Aparece en el historial junto con el evento de cambio de estado
- Es opcional en las transiciones
- Ayuda a documentar el motivo del cambio — por ejemplo, al rechazar un defecto, explique por qué no fue reproducible

---

## Buenas prácticas

- **Comente antes de tramitar** — si la tramitación cambiará la posesión del defecto a otro equipo, registre el contexto de la investigación en un comentario antes de pasarlo
- **Use la nota de transición para decisiones** — al cerrar, corregir o rechazar, la nota de transición es el lugar correcto para registrar qué se hizo o descubrió
- **Adjunte evidencias de reproducción** — las capturas de pantalla y logs del sandbox son temporales; si son relevantes para el registro definitivo, descárguelos y adjúntelos directamente en el defecto
- **El historial es inmutable** — los comentarios y eventos del historial no pueden editarse ni eliminarse, garantizando trazabilidad completa
