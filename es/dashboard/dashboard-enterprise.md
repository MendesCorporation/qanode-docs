# Dashboard

El **Dashboard** ofrece una vista consolidada del estado de sus pruebas mediante **widgets** configurables: tarjetas de métricas, gráficos y tablas.

---

## Visión General

[Dashboard principal](../../assets/images/dashboard-visao-geral.mp4)
*Imagen: Dashboard con múltiples widgets: tarjetas de métricas, gráfico de barras y tabla*

---

## Dashboards Múltiples

QANode admite múltiples dashboards con diferentes configuraciones de visibilidad:

| Tipo | Visibilidad | Descripción |
|------|-------------|-------------|
| **Privado** | Solo el creador | Dashboard personal |
| **Público** | Todos los usuarios | Dashboard compartido |
| **Por Rol** | Usuarios con roles específicos | Dashboard por equipo/rol |
| **Sistema** | Todos (solo lectura) | Dashboards predeterminados de QANode |

[Dashboards con diferentes configuraciones de visibilidad](../../assets/images/dashboard-visibilidade-painel.mp4)

---

## Widgets

### Tipos de Widget

| Tipo | Descripción | Uso Típico |
|------|-------------|------------|
| **Tarjeta de Métrica** | Valor numérico único con formato condicional | Total de ejecuciones, tasa de éxito |
| **Gauge** | Indicador semicircular con mínimo, máximo y valor actual | Tasa de éxito, SLA, progreso |
| **Gráfico de Barras** | Barras verticales con una o más series | Ejecuciones por día, fallos por proyecto |
| **Barras Apiladas** | Barras verticales con múltiples series apiladas | Comparación acumulada entre categorías |
| **Barras Horizontales** | Barras horizontales con una o más series | Rankings, comparaciones categóricas |
| **Gráfico de Línea** | Líneas de tendencia | Evolución de la tasa de éxito |
| **Línea Apilada** | Múltiples líneas con series acumuladas | Evolución por estado o equipo |
| **Gráfico de Área** | Área rellena | Acumulado de ejecuciones |
| **Área Apilada** | Áreas acumuladas por serie | Volumen por estado a lo largo del tiempo |
| **Gráfico de Torta** | Distribución proporcional | Proporción éxito/fallo |
| **Donut** | Distribución proporcional con centro libre para total | Distribución por estado con valor central |
| **Funnel** | Etapas en formato de embudo | Conversión, calidad o etapas de proceso |
| **Calendar Heatmap** | Intensidad por día en calendario | Fallos por día, ejecuciones por día |
| **Scatter** | Puntos por eje X/Y | Duración por cantidad de pasos |
| **Bubble** | Scatter con tamaño variable | Duración, pasos y severidad |
| **Tabla** | Datos tabulares | Lista de últimas ejecuciones |

### Creando un Widget

El asistente de creación de widgets tiene 3 pasos:

[Creando un Widget](../../assets/images/dashboard-criando-widget.mp4)

#### Paso 1: Fuente de Datos

Defina de dónde provendrán los datos:

**Query Builder (Recomendado):**

| Campo | Descripción |
|-------|-------------|
| **Tabla** | Tabla de datos (ejecuciones, proyectos, flujos, etc.) |
| **Columnas** | Campos a seleccionar |
| **Filtros** | Condiciones (igual a, contiene, mayor que, etc.) |
| **Agrupamiento** | Agrupar por campo (con formato de fecha) |
| **Agregación** | COUNT, SUM, AVG, MIN, MAX |
| **Ordenamiento** | Campo y dirección (ASC/DESC) |
| **Pivot** | Crear múltiples series a partir de un campo |
| **Límite** | Máximo de registros (hasta 1000) |

**Lógica de Filtros:** Combine múltiples filtros con los operadores **AND** u **OR**.

**Formato de Fecha en Agrupamiento:**

| Formato | Resultado |
|---------|-----------|
| Solo Fecha | `2024-01-15` |
| Solo Hora | `14:30` |
| Fecha y Hora | `2024-01-15 14:30` |
| Mes/Año | `2024-01` |
| Año | `2024` |

**SQL Directo:**

Para consultas más complejas, utilice el modo SQL con el editor Monaco. Prefiera Query Builder para widgets simples y use SQL cuando necesite CTEs, joins, agregaciones condicionales o cálculos que serían difíciles de expresar en el constructor:

```sql
SELECT
  DATE("startedAt") AS dia,
  status,
  COUNT(*)::int AS total
FROM "Run"
WHERE "isSandbox" = false
  AND "startedAt" >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY dia, status
ORDER BY dia;
```

> El modo SQL requiere el permiso `dashboard.sql`.

> El SQL también soporta CTEs (Common Table Expressions) con la cláusula `WITH`. Las consultas que modifican datos (INSERT, UPDATE, DELETE, DROP, etc.) siguen bloqueadas.

#### Paso 2: Visualización

Elija el tipo de gráfico y mapee los campos:

| Configuración | Descripción |
|---------------|-------------|
| **Tipo de Gráfico** | Métrica, Gauge, Barras, Barras Apiladas, Barras Horizontales, Línea, Área, Torta, Donut, Funnel, Heatmap, Scatter, Bubble o Tabla |
| **Eje X** | Campo para el eje horizontal |
| **Eje Y** | Campo para el eje vertical (valor numérico) |
| **Serie** | Campo para múltiples series (cuando se usa pivot) |
| **Label** | Campo de rótulo, usado por torta, donut, funnel, scatter, bubble y tablas |
| **Tamaño** | Campo numérico usado por Bubble |
| **Leyenda** | Mostrar/ocultar leyenda |
| **Tooltip** | Mostrar valores al pasar el cursor |

#### Paso 3: Apariencia

| Configuración | Descripción |
|---------------|-------------|
| **Título** | Nombre mostrado en el widget |
| **Colores** | Colores personalizados por serie/categoría |
| **Formato Condicional** | Reglas de color basadas en valores |
| **Columnas de Progreso** | Muestra columnas numéricas de la tabla como barra de progreso |
| **Límite / Objetivo** | Referencias visuales para Gauge e indicadores |

**Formato Condicional** (para tarjetas y tablas):

| Operador | Ejemplo |
|----------|---------|
| `>` | Si valor > 90 → verde |
| `<` | Si valor < 50 → rojo |
| `=` | Si valor = "failed" → rojo |
| `contains` | Si contiene "error" → amarillo |

### Tablas Con Progreso

En widgets de tabla, las columnas numéricas pueden mostrarse como barras de progreso. Esto es útil para acompañar:

- avance de proyectos;
- porcentaje de éxito;
- cobertura de escenarios;
- consumo de SLA;
- progreso de pipelines.

Cada columna de progreso puede tener color propio y valores mínimo/máximo. Campos textuales, IDs y fechas no se recomiendan para progreso.

### Formato Condicional En Tablas

Las reglas de formato condicional permiten destacar filas o celdas según valores de la propia consulta.

Ejemplos:

| Regla | Resultado |
|-------|-----------|
| `status = failed` | Fila en rojo |
| `tasa_exito < 80` | Indicador de atención |
| `fallos > 0` | Destacar escenarios con fallo |

Use formato condicional para llamar atención sobre problemas, pero evite aplicar demasiados colores al mismo tiempo.

---

## Diseño del Dashboard

El dashboard utiliza una **cuadrícula de 12 columnas** responsiva:

| Propiedad | Descripción |
|-----------|-------------|
| **Ancho** | 1 a 12 columnas |
| **Alto** | 1 a 10 filas |
| **Posición X** | Columna de inicio (0-11) |
| **Posición Y** | Fila de inicio |

### Reorganizando Widgets

[Reorganizando Widgets](../../assets/images/dashboard-reorganizar-widgets.mp4)

- **Arrastrar** un widget para reposicionarlo
- **Redimensionar** arrastrando la esquina inferior derecha
- La cuadrícula se ajusta automáticamente para evitar superposiciones
- Los tamaños predeterminados se optimizan por tipo: métricas empiezan menores, mientras gráficos y tablas empiezan mayores.

---

## Ejemplos de Widgets

[Ejemplos de Widgets](../../assets/images/dashboard-tipos-de-widgets.mp4)

### Tarjeta: Tasa de Éxito

```
Tabla: runs
Agregación: COUNT(*)
Filtro: status = "success" AND started_at >= hoy - 30 días
```

### Gráfico de Barras: Ejecuciones por Día

```
Tabla: runs
Agrupamiento: started_at (solo fecha)
Agregación: COUNT(*)
Pivot: status
Tipo: Barras
```

Resultado: Barras apiladas con colores diferentes para éxito/fallo por día.

### Tabla: Últimos Fallos

```
Tabla: runs
Filtro: status = "failed"
Columnas: nombre del flujo, fecha, duración, error
Ordenamiento: started_at DESC
Límite: 10
```

---

## Auto-Refresh

Los widgets pueden configurarse para actualizarse automáticamente en intervalos definidos, manteniendo los datos siempre actualizados.

[Auto-Refresh](../../assets/images/dashboard-auto-refresh.mp4)

---

## Consejos

- **Comience simple** — una tarjeta con el total de ejecuciones + un gráfico de barras diario
- Use **formato condicional** para destacar problemas (tasa inferior al 80% = rojo)
- **Dashboards por equipo** — cree dashboards con visibilidad por rol
- Use el **modo pivot** para crear gráficos con múltiples series sin SQL
- **Limite los datos** — las consultas grandes pueden afectar el rendimiento
