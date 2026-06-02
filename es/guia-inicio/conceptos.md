# Conceptos Fundamentales

Antes de profundizar en QANode, es importante comprender los conceptos clave que forman la base de la plataforma.

---

## Visión General de la Jerarquía

```
Proyectos
├── Escenarios (Flujos)
│   ├── Nodos (Nodes)
│   │   ├── Configuración
│   │   ├── Conexiones (Edges)
│   │   └── Outputs/Inputs
│   └── Ejecuciones (Runs)
│       ├── Pasos (Steps)
│       ├── Logs
│       └── Artefactos y archivos
├── Componentes
│   ├── Contrato de Entrada
│   ├── Flujo Reutilizable
│   └── Contrato de Salida
└── Suites
    ├── Escenarios ordenados
    ├── Programación (Cron)
    └── Ejecuciones de Suite
```

---

## Proyectos

Un **proyecto** es el nivel más alto de organización. Agrupa escenarios de prueba y suites relacionados con un sistema, funcionalidad o equipo.

Cada proyecto tiene:
- **Nombre** y **descripción**
- **Estado**: Activo, Completado, Cancelado o Archivado
- **Fechas**: Inicio y fin planificados
- **Documentación**: Archivos adjuntos (requisitos, especificaciones, etc.)
- **Estadísticas**: Número de escenarios, suites y ejecuciones recientes

---

## Escenarios (Flujos)

Un **escenario** (también llamado **flujo**) es un caso de prueba representado visualmente como un grafo de nodos conectados. Cada escenario describe una secuencia de acciones a ejecutar — como navegar por un sitio web, llamar a una API o consultar una base de datos.

Los escenarios están compuestos por:
- **Nodos (Nodes)**: Acciones individuales (hacer clic, escribir, hacer una solicitud, etc.)
- **Conexiones (Edges)**: Vínculos entre nodos que definen el orden de ejecución
- **Configuraciones**: Parámetros de cada nodo

### Ejemplo Visual

```
[Navigate] → [Fill email] → [Fill senha] → [Click login] → [Assert welcome]
```

En este ejemplo, cada caja es un nodo y las flechas representan conexiones que indican el flujo de ejecución.

---

## Nodos (Nodes)

Los **nodos** son los bloques fundamentales de un escenario. Cada nodo ejecuta una acción específica y puede producir outputs que son consumidos por los nodos siguientes.

### Categorías de Nodos

| Categoría | Color | Nodos |
|-----------|-------|-------|
| **Control de Flujo** | 🟡 Amarillo | If, Switch, Loop |
| **Web** | 🔵 Azul | Smart Web Flow, Web Flow, Smart Locators |
| **Mobile** | 🔴 Rojo claro | Mobile Flow |
| **API** | 🟣 Morado | HTTP Request |
| **Base de Datos** | 🟢 Verde | PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, MongoDB |
| **Archivos** | 🟤 Dorado | Generar Archivo, Extraer Archivo, Base64 a Archivo, Archivo a Base64 |
| **Infraestructura** | 🔵 Azul Claro | SSH Command |
| **Performance** | 🟫 Ámbar | Load Test |
| **Utilidades** | ⚪ Gris | Set Variable, Log, Wait, Stop and Fail, Custom JavaScript |
| **Nodos Personalizados** | 🩷 Rosa | Nodos de proveedores externos |

### Anatomía de un Nodo

Cada nodo tiene:

- **Handles de Entrada** (●): Puntos donde llegan conexiones de otros nodos
- **Handles de Salida** (●): Puntos desde donde salen conexiones hacia otros nodos
- **Configuración**: Campos específicos del tipo de nodo
- **Outputs**: Datos producidos después de la ejecución (accesibles mediante expresiones)
- **Label**: Nombre identificador del nodo en el flujo

### Outputs y el Sistema de Expresiones

Cuando un nodo se ejecuta, produce **outputs** — datos que pueden ser usados por los nodos siguientes. Para acceder a estos datos, usa el sistema de **expresiones** con la sintaxis `{{ }}`:

```
{{ steps["Nombre del Nodo"].outputs.propiedad }}
```

**Ejemplos:**
```
{{ steps["http-request"].outputs.body.token }}    → Token de una respuesta de API
{{ steps.login.outputs.extracts.userName }}   → Texto extraído de una página
{{ steps.query.outputs.rows[0].email }}       → Primer email del resultado SQL
```

> Para detalles completos, consulta la documentación de [Expresiones](../expresiones/expresiones.md).

---

## Componentes Reutilizables

Un **componente** es un flujo reutilizable con contrato de entrada y salida. Permite encapsular una secuencia que aparece en varios escenarios, como login, preparación de datos, validación común o consulta auxiliar.

Los componentes tienen:

- **Input**: campos que el escenario debe informar;
- **Flujo interno**: nodos ejecutados dentro del componente;
- **Output**: valor retornado al escenario;
- **Estado**: borrador o publicado.

Después de publicado, el componente aparece en la paleta del editor de escenarios y puede arrastrarse al canvas como un nodo común.

> Para detalles, vea [Componentes Reutilizables](../componentes/vision-general.md).

---

## Conexiones (Edges)

Las **conexiones** vinculan la salida de un nodo con la entrada de otro, definiendo el orden de ejecución. QANode ejecuta los nodos en **orden topológico** — es decir, un nodo solo se ejecuta después de que todos los nodos conectados a su entrada hayan sido completados.

### Conexiones Condicionales

Algunos nodos tienen múltiples salidas condicionales:

- **If** → salidas `true` y `false`
- **Switch** → salidas para cada case + `default`
- **Loop** → salidas `loop` (siguiente iteración) y `done` (fin del loop)

Solo se ejecutará la ruta correspondiente a la condición evaluada. Los nodos en rutas no activadas serán **omitidos**.

---

## Ejecuciones (Runs)

Una **ejecución** es una instancia de un escenario siendo procesado. Al ejecutar un escenario, QANode:

1. Resuelve el orden topológico de los nodos
2. Ejecuta cada nodo secuencialmente
3. Evalúa expresiones e inyecta datos de los outputs anteriores
4. Registra logs, capturas de pantalla, archivos y artefactos
5. Disponibiliza los resultados de la ejecución

Cada ejecución tiene:
- **Estado**: `running`, `success` o `failed`
- **Duración**: Tiempo total de ejecución
- **Pasos**: Resultado individual de cada nodo
- **Artefactos**: Capturas de pantalla y archivos generados
- **Logs**: Mensajes detallados de cada paso

---

## Suites de Prueba

Una **suite** agrupa varios escenarios para ejecución secuencial. Las suites son útiles para:

- Ejecutar múltiples escenarios en orden
- Programar ejecuciones periódicas (diarias, por hora, etc.)
- Detener en la primera falla (opcional)

### Programación

Las suites pueden programarse usando **expresiones cron**:

| Expresión | Significado |
|-----------|-------------|
| `0 8 * * *` | Todos los días a las 8:00 |
| `0 */2 * * *` | Cada 2 horas |
| `0 9 * * 1-5` | Lunes a viernes a las 9:00 |
| `30 18 * * 5` | Viernes a las 18:30 |

---

## Variables

Las **variables** son valores globales reutilizables en cualquier escenario. Tipos soportados:

| Tipo | Uso |
|------|-----|
| `string` | Textos (URLs, nombres, mensajes) |
| `number` | Números (timeouts, límites) |
| `boolean` | Flags (true/false) |
| `json` | Objetos y arrays complejos |
| `secret` | Valores cifrados (contraseñas, tokens) |

Accede a las variables en tus flujos con: `{{ variables.nombreVariable }}`

---

## Credenciales

Las **credenciales** almacenan información de conexión de forma segura y centralizada. Tipos soportados:

- **HTTP/API** — URL base, autenticación, headers
- **PostgreSQL**, **MySQL**, **MariaDB**, **SQL Server**, **Oracle** — Datos de conexión a la base de datos
- **MongoDB** — URI o datos de conexión
- **SSH** — Host, usuario, contraseña o clave privada

Las credenciales pueden ser referenciadas por los nodos que necesitan una conexión, evitando que las contraseñas queden expuestas en los flujos.


---

## Próximos Pasos

Ahora que conoces los conceptos fundamentales:

- [Editor de Flujos](../editor-flujos/vision-general.md) — Aprende a usar el editor visual
- [Referencia de Nodos](../nodos/control-flujo/if.md) — Conoce todos los nodos en detalle
- [Expresiones](../expresiones/expresiones.md) — Domina el sistema de interpolación
