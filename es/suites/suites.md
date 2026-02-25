# Suites de Prueba

**Las suites** permiten agrupar múltiples escenarios (flujos) para ejecución secuencial o no secuencial. Son ideales para pruebas de regresión, smoke tests y ejecuciones programadas.

---

## Crear una Suite

1. En el menú lateral, haga clic en **Suites**
2. Haga clic en **+ Nueva Suite**
3. Complete:

| Campo | Requerido | Descripción |
|-------|-----------|-------------|
| **Nombre** | Sí | Nombre de la suite |
| **Proyecto** | Sí | Proyecto al que pertenece la suite |
| **Escenarios** | Sí | Seleccione los escenarios y el orden |
| **Detener en Fallo** | No | Interrumpir al primer escenario que falle |
| **Modo de Ejecución** | No | Secuencial (predeterminado) o No secuencial |

4. Haga clic en **Crear**

<!-- ![Crear suite](../../assets/images/criar-suite.png) -->
*Imagen: Formulario de creación de suite con selección de escenarios y alternador de detener en fallo*

---

## Modo de Ejecución

La suite ofrece dos modos de ejecución:

### Secuencial (predeterminado)

Los escenarios se ejecutan **uno a la vez**, en el orden definido. Puede reordenarlos arrastrando los escenarios en la lista.

```
Suite: Regresión Login (Secuencial)
├── 1. Login con credenciales válidas    ⏳ esperando
├── 2. Login con contraseña incorrecta   ⏳ esperando
├── 3. Login con email inválido          ⏳ esperando
├── 4. Recuperación de contraseña        ⏳ esperando
└── 5. Logout                            ⏳ esperando
→ Ejecuta: 1, luego 2, luego 3...
```

Use el modo secuencial cuando:
- Los escenarios **dependen unos de otros** (ej: login antes de operaciones)
- Necesita un **orden de ejecución garantizado**
- Quiere usar la opción **Detener en Fallo** para interrumpir en el primer fallo

### No secuencial

Los escenarios se ejecutan **de forma independiente**, sin orden garantizado. Todos los escenarios se lanzan y ejecutan en paralelo.

```
Suite: Regresión Login (No secuencial)
├── 1. Login con credenciales válidas    ▶ ejecutando
├── 2. Login con contraseña incorrecta   ▶ ejecutando
├── 3. Login con email inválido          ▶ ejecutando
├── 4. Recuperación de contraseña        ▶ ejecutando
└── 5. Logout                            ▶ ejecutando
→ Todos ejecutan al mismo tiempo
```

Use el modo no secuencial cuando:
- Los escenarios son **independientes** entre sí
- Quiere **reducir el tiempo total** de ejecución de la suite
- El orden de ejecución **no importa**

---

## Detener en Fallo

> **Nota:** La opción "Detener en Fallo" solo tiene efecto en el modo **Secuencial**. En el modo no secuencial, como todos los escenarios se ejecutan de forma independiente, esta opción no se aplica.

Cuando **Detener en Fallo** está activado:
- Si un escenario falla, los escenarios siguientes **no se ejecutan**
- Útil cuando los escenarios dependen unos de otros (ej: login antes de operaciones)

Cuando está desactivado:
- Todos los escenarios se ejecutan independientemente de los fallos
- Útil para suites de regresión completa donde quiere ver todos los fallos

---

## Ejecutar una Suite

### Ejecución Manual

1. En la lista de suites, haga clic en el botón **Ejecutar** (▶️)
2. Todos los escenarios se ejecutarán en orden
3. El resultado aparece en la lista de ejecuciones

### Ejecución Programada

Las suites pueden ejecutarse automáticamente en horarios definidos.

---

## Programación

### Configurar la Programación

1. Edite la suite
2. En la sección **Programación**, active el alternador
3. Ingrese la **expresión cron**
4. El sistema mostrará los próximos 5 horarios de ejecución

[Programación de suite](../../assets/images/suites.mp4)
*Imagen: Sección de programación con campo cron, alternador de activación y vista previa de los próximos horarios*

### Expresiones Cron

El formato cron tiene 5 campos:

```
┌───────────── minuto (0-59)
│ ┌─────────── hora (0-23)
│ │ ┌───────── día del mes (1-31)
│ │ │ ┌─────── mes (1-12)
│ │ │ │ ┌───── día de la semana (0-7, 0 y 7 = domingo)
│ │ │ │ │
* * * * *
```

### Ejemplos Comunes

| Expresión | Significado |
|-----------|-------------|
| `0 8 * * *` | Todos los días a las 8:00 |
| `0 8 * * 1-5` | Lunes a viernes a las 8:00 |
| `0 */2 * * *` | Cada 2 horas |
| `30 9 * * 1` | Todos los lunes a las 9:30 |
| `0 0 * * *` | Todos los días a medianoche |
| `0 6,12,18 * * *` | A las 6h, 12h y 18h |
| `*/15 * * * *` | Cada 15 minutos |
| `0 9 1 * *` | Primer día del mes a las 9:00 |

### Activar/Desactivar

Puede activar y desactivar la programación sin eliminar la expresión cron. Cuando está desactivada, la expresión se conserva para reactivación futura.

---

## Resultados de la Suite

Después de la ejecución, la suite muestra:

| Información | Descripción |
|-------------|-------------|
| **Estado General** | Éxito (todos pasaron) o Fallo (alguno falló) |
| **Escenarios Ejecutados** | Cuántos fueron procesados |
| **Escenarios con Éxito** | Cuántos pasaron |
| **Escenarios con Fallo** | Cuántos fallaron |
| **Duración Total** | Tiempo total de ejecución |

Cada escenario individual también tiene su resultado detallado (logs, capturas de pantalla, etc.).

---

## Alarmas

Puede configurar **alarmas** para ser notificado cuando las suites fallen. Consulte la sección de [Administración](../administracion/administracion-enterprise.md) para configurar alarmas.

---

## Consejos

- **Ordene estratégicamente** — en modo secuencial, coloque los escenarios de configuración (login, preparación) primero
- **Use "Detener en Fallo"** cuando los escenarios dependen unos de otros (modo secuencial)
- **Desactive "Detener en Fallo"** para suites de regresión completa
- **Use el modo no secuencial** para suites de regresión con escenarios independientes — reduce el tiempo total
- **Programe en horas de baja actividad** — evite conflictos con el uso manual
- **Monitoree con alarmas** — configure notificaciones para fallos en suites programadas - versión Enterprise
