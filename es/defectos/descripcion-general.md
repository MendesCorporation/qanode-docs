# Módulo de Defectos — Descripción General

> **Disponible en:** QANode Enterprise

El módulo de Defectos transforma una ejecución fallida en un proceso controlado de seguimiento. En lugar de que una ejecución con fallo simplemente quede registrada en el historial, puede abrir un defecto formal, asignarlo a una persona o equipo, seguir su investigación y cerrarlo con un historial completo de auditoría.

---

## Lo que ofrece el módulo

- **Apertura de defecto** desde una ejecución fallida o manualmente
- **Flujo de trabajo configurable** — su organización define los estados y las transiciones del proceso
- **Colas y asignación** — los defectos pueden dirigirse a equipos o personas específicas
- **Claim** — un usuario toma posesión del defecto antes de actuar
- **Investigación en sandbox** — copia desechable del flujo original para investigar sin afectar ejecuciones oficiales ni KPIs
- **Campos personalizados** — campos adicionales definidos por su organización
- **Comentarios, adjuntos e historial completo** — toda interacción con el defecto queda registrada

---

## Cómo acceder

El módulo de Defectos aparece en el menú lateral con el ícono de **bug** (🐛). Al hacer clic, se muestra la lista de todos los defectos de la instancia con filtros por estado, severidad, prioridad, cola y responsable.

Para abrir un defecto directamente desde una ejecución fallida, acceda al detalle de la ejecución y haga clic en **Abrir Defecto**.

---

## Permisos

El acceso a cada funcionalidad del módulo está controlado por permisos asignados al rol del usuario. Los permisos disponibles son:

| Permiso | Qué permite |
|---------|-------------|
| `bug.view` | Ver defectos, comentar y gestionar sus propios adjuntos |
| `bug.create` | Abrir nuevos defectos |
| `bug.edit` | Editar campos personalizados del defecto |
| `bug.assign` | Asignar defectos a personas/colas y hacer claim |
| `bug.transition` | Tramitar en las transiciones normales del flujo de trabajo |
| `bug.transition_any` | Bypass administrativo — tramitar a cualquier estado sin restricción de posesión |
| `bug.run_sandbox` | Crear y descartar sandboxes de investigación |
| `bug.delete_attachment_any` | Eliminar adjuntos enviados por cualquier persona |
| `bug.configure_workflow` | Configurar el flujo de trabajo, estados, transiciones y campos personalizados |

Los permisos se configuran en **Configuración → Roles**.

---

## Conceptos fundamentales

### Flujo de trabajo

El flujo de trabajo define qué estados existen, qué transiciones son posibles entre ellos y qué campos aparecen en cada etapa. Cada instancia tiene exactamente un flujo de trabajo activo. Ver [Workflow Builder](./workflow-builder.md).

### Estado

Cada defecto siempre está en un estado. El estado inicial es definido en el flujo de trabajo y es donde el defecto nace. Los estados finales indican cierre — cuando un defecto llega a un estado final, la fecha de cierre se registra automáticamente.

### Transición

Una transición es el camino entre dos estados. Cada transición puede definir campos obligatorios que deben completarse al tramitar.

### Cola

Una cola representa un equipo o grupo responsable de un conjunto de defectos. Cuando un defecto está **solo en cola** (sin responsable directo), cualquier miembro de esa cola con el permiso adecuado puede actuar sobre él. Si existe un responsable directo, la posesión de ese usuario prevalece en el flujo normal.

### Claim

Claim es el acto de tomar posesión del defecto. Al hacer claim, el defecto pasa a ser suyo. La tramitación normal requiere que el defecto esté claimado por el usuario que va a tramitar.

### Sandbox

El sandbox es una copia desechable del flujo original utilizada para investigar el fallo. Los cambios en el sandbox no afectan el flujo oficial ni cuentan en los KPIs e informes. Ver [Sandbox de Investigación](./sandbox.md).

---

## Navegación

| Página | Qué aprenderá |
|--------|---------------|
| [Workflow Builder](./workflow-builder.md) | Cómo configurar estados, transiciones, campos y campos personalizados |
| [Ciclo de Vida del Defecto](./ciclo-de-vida.md) | Cómo abrir, asignar, hacer claim y tramitar defectos |
| [Sandbox de Investigación](./sandbox.md) | Cómo investigar fallos sin afectar ejecuciones oficiales |
| [Comentarios, Adjuntos e Historial](./comentarios-adjuntos.md) | Cómo colaborar y seguir el historial de un defecto |
