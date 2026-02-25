# Dashboard

El **Dashboard** ofrece una vista consolidada del estado de sus pruebas mediante **widgets** configurables: tarjetas de métricas, gráficos y tablas.

---

## Visión General

[Dashboard principal](../../assets/images/dashboard.mp4)
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

---

## Widgets

### Tipos de Widget

| Tipo | Descripción | Uso Típico |
|------|-------------|------------|
| **Tarjeta de Métrica** | Valor numérico único con formato condicional | Total de ejecuciones, tasa de éxito |
| **Gráfico de Barras** | Barras verticales con una o más series | Ejecuciones por día, fallos por proyecto |
| **Gráfico de Línea** | Líneas de tendencia | Evolución de la tasa de éxito |
| **Gráfico de Área** | Área rellena | Acumulado de ejecuciones |
| **Gráfico de Torta** | Distribución proporcional | Proporción éxito/fallo |
| **Tabla** | Datos tabulares | Lista de últimas ejecuciones |

### Creando un Widget

El asistente de creación de widgets tiene 3 pasos:

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

Para consultas más complejas, utilice el modo SQL con el editor Monaco:

```sql
SELECT
  DATE(started_at) as dia,
  status,
  COUNT(*) as total
FROM runs
WHERE started_at >= NOW() - INTERVAL '30 days'
GROUP BY dia, status
ORDER BY dia
```

> El modo SQL requiere el permiso `dashboard.sql`.

#### Paso 2: Visualización

Elija el tipo de gráfico y mapee los campos:

| Configuración | Descripción |
|---------------|-------------|
| **Tipo de Gráfico** | Tarjeta, Barras, Línea, Área, Torta, Tabla |
| **Eje X** | Campo para el eje horizontal |
| **Eje Y** | Campo para el eje vertical (valor numérico) |
| **Serie** | Campo para múltiples series (cuando se usa pivot) |
| **Leyenda** | Mostrar/ocultar leyenda |
| **Tooltip** | Mostrar valores al pasar el cursor |

#### Paso 3: Apariencia

| Configuración | Descripción |
|---------------|-------------|
| **Título** | Nombre mostrado en el widget |
| **Colores** | Colores personalizados por serie/categoría |
| **Formato Condicional** | Reglas de color basadas en valores |

**Formato Condicional** (para tarjetas y tablas):

| Operador | Ejemplo |
|----------|---------|
| `>` | Si valor > 90 → verde |
| `<` | Si valor < 50 → rojo |
| `=` | Si valor = "failed" → rojo |
| `contains` | Si contiene "error" → amarillo |

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

- **Arrastrar** un widget para reposicionarlo
- **Redimensionar** arrastrando la esquina inferior derecha
- La cuadrícula se ajusta automáticamente para evitar superposiciones

---

## Ejemplos de Widgets

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

---

## Consejos

- **Comience simple** — una tarjeta con el total de ejecuciones + un gráfico de barras diario
- Use **formato condicional** para destacar problemas (tasa inferior al 80% = rojo)
- **Dashboards por equipo** — cree dashboards con visibilidad por rol
- Use el **modo pivot** para crear gráficos con múltiples series sin SQL
- **Limite los datos** — las consultas grandes pueden afectar el rendimiento
