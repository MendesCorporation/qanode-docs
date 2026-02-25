# Proyectos

**Los proyectos** son la unidad principal de organización en QANode. Cada proyecto agrupa escenarios de prueba (flujos), suites, documentación y ejecuciones relacionadas con un sistema, funcionalidad o equipo.

---

## Crear un Proyecto

1. En el menú lateral, haga clic en **Proyectos**
2. Haga clic en **+ Nuevo Proyecto**
3. Complete los campos:

| Campo | Requerido | Descripción |
|-------|-----------|-------------|
| **Nombre** | Sí | Nombre del proyecto |
| **Descripción** | No | Descripción detallada |
| **Estado** | Sí | Estado actual del proyecto |
| **Fecha de Inicio** | No | Fecha de inicio planificada |
| **Fecha de Término** | No | Fecha de finalización planificada |

4. Haga clic en **Crear**

<!-- ![Crear proyecto](../../assets/images/criar-projeto.png) -->
*Imagen: Modal de creación de proyecto con todos los campos completados*

---

## Estado del Proyecto

| Estado | Descripción |
|--------|-------------|
| **Activo** | Proyecto en curso |
| **Completado** | Proyecto finalizado con éxito |
| **Cancelado** | Proyecto cancelado |
| **Archivado** | Proyecto archivado (no aparece en la lista principal) |

Para cambiar el estado, acceda al proyecto y use las opciones en el encabezado:
- **Completar** → Marca como completado
- **Cancelar** → Marca como cancelado
- **Archivar** → Mueve a archivados

---

## Página del Proyecto

La página de detalle del proyecto tiene las siguientes pestañas:

### Resumen

Muestra estadísticas y un resumen del proyecto:

- **Total de Escenarios** — Cantidad de flujos de prueba
- **Total de Suites** — Cantidad de suites
- **Escenarios Pendientes/Aprobados/Reprobados** — Estado de los escenarios
- **Ejecuciones Recientes** — Lista de las últimas ejecuciones
- **Progreso** — Comparación entre el tiempo transcurrido y los escenarios ejecutados

<!-- ![Resumen del proyecto](../../assets/images/projeto-visao-geral.png) -->
*Imagen: Página de resumen mostrando tarjetas de estadísticas, ejecuciones recientes y barra de progreso*

### Escenarios

Lista todos los escenarios (flujos) del proyecto:

| Información | Descripción |
|-------------|-------------|
| **Nombre** | Nombre del escenario |
| **Estado** | Último resultado (pendiente, éxito, fallo) |
| **Fecha de Creación** | Cuándo fue creado |

**Acciones:**
- **+ Nuevo Escenario** — Crea un nuevo flujo y abre el editor
- **Ejecutar** — Ejecuta el escenario rápidamente
- **Editar** — Abre el editor de flujos
- **Eliminar** — Elimina el escenario

### Suites

Lista las suites del proyecto con información de programación y último resultado.

> Para detalles sobre suites, consulte [Suites de Prueba](../suites/suites.md).

### Ejecuciones

Historial de todas las ejecuciones del proyecto:

| Información | Descripción |
|-------------|-------------|
| **Escenario/Suite** | Nombre de lo que fue ejecutado |
| **Estado** | Resultado (éxito, fallo, ejecutando) |
| **Duración** | Tiempo de ejecución |
| **Fecha** | Fecha y hora de la ejecución |

### Documentación

Gestión de archivos del proyecto:

- **Cargar** — Suba documentos (requisitos, especificaciones, etc.)
- **Descargar** — Descargue documentos adjuntos
- **Eliminar** — Elimine documentos

---

## Filtrado y Búsqueda

En la lista de proyectos, puede:

- **Buscar por nombre** — Campo de búsqueda en la parte superior
- **Filtrar por estado** — Menú desplegable de filtro (Activo, Completado, etc.)

---

## Duplicar Proyecto

Para crear una copia de un proyecto existente:

1. En la lista de proyectos, haga clic en las opciones (⋮) del proyecto
2. Seleccione **Duplicar**
3. Se creará un nuevo proyecto con los mismos escenarios

---

## Consejos

- Use el **estado** para hacer seguimiento del ciclo de vida de los proyectos
- **Archive** los proyectos antiguos para mantener la lista limpia
- **Las fechas de inicio y término** ayudan a hacer seguimiento del progreso vs. la planificación
- Use la pestaña de **Documentación** para mantener los requisitos y especificaciones junto al proyecto
