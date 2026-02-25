# Informes

QANode genera automáticamente **informes PDF** después de cada ejecución y permite generar informes consolidados por período, con exportación y envío por correo electrónico.

---

## Informes de Ejecución

Después de cada ejecución de escenario o suite, se genera automáticamente un **informe PDF** con:

- **Resumen**: Estado, duración, fecha de ejecución
- **Detalles por Nodo**: Estado individual, registros, salidas, duración
- **Capturas de Pantalla**: Imágenes capturadas (cuando están configuradas en los nodos web)
- **Errores**: Detalles de los fallos encontrados

El informe está disponible en la lista de ejecuciones y puede descargarse en cualquier momento.

---

## Informes Consolidados

Genere informes que abarcan múltiples ejecuciones en un período:

1. En el menú lateral, haga clic en **Informes**
2. Seleccione el **período** (fecha de inicio y fin)
3. (Opcional) Filtre por **proyecto**
4. Haga clic en **Generar Informe**

<!-- ![Generar informe](../../assets/images/gerar-relatorio.png) -->
*Imagen: Pantalla de informes con selección de período, filtro de proyecto y botón de generación*

### Métricas del Informe

| Métrica | Descripción |
|---------|-------------|
| **Tasa de Éxito** | Porcentaje de ejecuciones exitosas |
| **Total de Ejecuciones** | Cantidad total de ejecuciones en el período |
| **Ejecuciones Exitosas** | Cantidad de ejecuciones aprobadas |
| **Ejecuciones Fallidas** | Cantidad de ejecuciones reprobadas |
| **Duración Promedio** | Tiempo promedio de ejecución |
| **Principales Fallos por Escenario** | Escenarios que más fallaron |
| **Principales Fallos por Tipo de Nodo** | Tipos de nodo que más fallaron |

### Gráfico de Ejecuciones por Día

El informe incluye un gráfico de barras que muestra la distribución de ejecuciones exitosas y fallidas a lo largo de los días del período seleccionado.

### Métricas por Proyecto

Cuando se selecciona un proyecto específico, se muestran métricas adicionales:

| Métrica | Descripción |
|---------|-------------|
| **Total de Escenarios/Suites** | Cantidad en el proyecto |
| **Escenarios Completados** | Escenarios que pasaron |
| **Escenarios Pendientes** | Escenarios aún no ejecutados |
| **Escenarios Fallidos** | Escenarios que fallaron |
| **Progreso** | % del tiempo transcurrido vs % escenarios ejecutados |

---

## Exportación

### Descarga PDF

1. Genere el informe
2. Haga clic en **Exportar PDF**
3. El archivo se descargará con el nombre: `report_{proyecto}_{inicio}_{fin}.pdf`

### Envío por Correo Electrónico

1. Genere el informe
2. Haga clic en **Enviar por Correo**
3. Seleccione los destinatarios:
   - **Usuarios registrados** — seleccione de la lista
   - **Correos adicionales** — agregue direcciones personalizadas
4. Haga clic en **Enviar**

El informe PDF se enviará como **archivo adjunto** del correo electrónico.

> **Requisito:** El SMTP debe estar configurado en Configuración para el envío de correos. Consulte [Administración](../administracion/administracion-enterprise.md).

---

## Plantilla de Informe

El diseño del informe PDF es configurable. Acceda a **Configuración** → **Plantilla de Informe** para personalizar:

| Configuración | Descripción |
|---------------|-------------|
| **Imagen de Fondo** | Imagen de fondo de las páginas |
| **Logo** | Logo mostrado en el encabezado |
| **Mostrar Logo** | Activar/desactivar logo |
| **Incluir Capturas** | Incluir capturas de pantalla |
| **Tamaño de Capturas** | Dimensión de las imágenes en el informe |
| **Incluir Registros** | Incluir registros de cada nodo |
| **Máximo de Registros** | Límite de líneas de registro por nodo |
| **Incluir Salidas** | Incluir datos de salida de los nodos |
| **Incluir Duración** | Mostrar duración de cada nodo |
| **Incluir ID del Nodo** | Mostrar identificador técnico |
| **Numeración de Pasos** | Numerar los pasos secuencialmente |
| **Sección de Resumen** | Incluir resumen al inicio |
| **Detalles de Errores** | Incluir detalles completos de los errores |
| **Metadatos** | Fecha, ID de ejecución, nombre del proyecto/flujo |

<!-- ![Plantilla de informe](../../assets/images/template-relatorio.png) -->
*Imagen: Pantalla de configuración de la plantilla con vista previa de las opciones*

---

## Consejos

- **Configure la plantilla** antes de generar informes — especialmente logo e imagen de fondo
- **Incluya capturas de pantalla** para evidencias visuales — fundamental para auditorías
- **Use el filtro por proyecto** para informes enfocados
- **Programe envíos** — use suites programadas + alarmas para informes automáticos
- **Limite los registros** — los informes con demasiados registros se vuelven extensos; use 10-20 líneas por nodo
