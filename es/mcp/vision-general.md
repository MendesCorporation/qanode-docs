# MCP — Integración con IA

> **Disponible en:** QANode Enterprise

El **MCP** conecta herramientas de IA con QANode para que el usuario pueda pedir acciones desde la IDE, un chat corporativo u otro cliente compatible.

Con esta integración, un cliente de IA puede operar QANode con los permisos del usuario autenticado: crear escenarios, ajustar flujos, consultar ejecuciones, leer documentación de proyectos, crear dashboards, investigar fallos, trabajar con defectos y usar los nodos visuales de la plataforma.

---

## Cuándo Usar

Use MCP cuando quiera:

- pedir a una IA que cree o ajuste escenarios en QANode;
- automatizar flujos que pasan por web, API, base de datos, archivos, SSH o mobile;
- investigar un escenario existente que está fallando;
- transformar documentación de proyecto en escenarios iniciales;
- crear dashboards o consultar métricas de QANode en lenguaje natural;
- abrir o analizar defectos mientras trabaja en el código.

MCP no reemplaza el editor visual. Todo lo creado sigue apareciendo en QANode como proyecto, escenario, nodo, ejecución, dashboard, suite o defecto normal.

---

## Cómo Conectar

1. Genere un token en **Configuración → Tokens de Acceso**
2. Configure el cliente MCP usado por su herramienta de IA
3. Informe la URL de la API de QANode
4. Envíe el token en el header `Authorization`

Endpoint:

```text
https://qanode.empresa.com/api/mcp
```

Header:

```text
Authorization: Bearer qnt_xxxxx
```

Ejemplo genérico:

```json
{
  "mcpServers": {
    "qanode": {
      "url": "https://qanode.empresa.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${QANODE_TOKEN}"
      }
    }
  }
}
```

El formato exacto depende del cliente de IA, pero la URL y el header son los mismos.

---

## Seguridad Y Permisos

MCP respeta los mismos permisos de QANode.

| Área | Regla |
|------|-------|
| Proyectos | Acceso limitado a los proyectos permitidos al usuario |
| Escenarios | Creación, edición y ejecución siguen los permisos del usuario |
| Ejecuciones | La consulta de ejecuciones respeta permisos de visualización |
| Credenciales | Los secretos nunca se devuelven en texto claro |
| Variables secretas | Los valores secretos se muestran enmascarados |
| Dashboards | Consultas y widgets respetan los permisos de datos |
| Defectos | Workflow, campos obligatorios y transiciones siguen aplicando |

El token no concede acceso extra. Solo autentica el cliente de IA como ese usuario o integración.

---

## Qué Puede Pedir

| Área | Ejemplos de uso |
|------|-----------------|
| **Proyectos** | Buscar proyectos, crear proyecto y leer documentos adjuntos |
| **Escenarios** | Crear flujos, agregar nodos, conectar etapas, validar y ejecutar |
| **Smart Web** | Grabar navegación web, validar pantallas, extraer datos y reutilizar sesión cuando sea necesario |
| **Mobile** | Grabar jornadas Android/iOS con Appium y guardarlas como Mobile Flow |
| **Base de Datos** | Consultar schema y crear consultas o validaciones de base |
| **Archivos** | Generar, extraer y reutilizar archivos en flujos |
| **Variables** | Crear variables globales o de proyecto para reutilización |
| **Credenciales** | Crear credenciales cuando el usuario informa los datos necesarios |
| **Suites** | Crear, editar, ejecutar, programar y diagnosticar fallos |
| **Defectos** | Abrir, comentar, adjuntar, tramitar e investigar defectos |
| **Dashboards** | Crear paneles, widgets y consultas rápidas sobre datos de QANode |
| **Componentes** | Crear componentes reutilizables cuando el usuario lo pide explícitamente |

---

## Cómo Pedir

Cuanto más claro sea el pedido, mejor será el resultado al usar QANode desde un cliente de IA. Incluya:

- el proyecto donde se debe crear el escenario;
- la URL o sistema objetivo;
- credenciales o nombre de la credencial ya guardada;
- datos de prueba permitidos;
- resultado esperado;
- reglas de limpieza, como eliminar datos creados al final;
- si el flujo solo debe crearse o también ejecutarse.

Ejemplo:

```text
En el proyecto Task Manager, crea un escenario que acceda a localhost:3008,
haga login con el usuario de prueba, cree una tarea, la marque como completada
y valide en la base que el estado quedó completed.
```

Cuando falte información importante, confirme los datos antes de crear registros, cambiar datos reales o ejecutar acciones destructivas.

---

## Smart Web Y Referencia Del Objetivo

En páginas con tablas, cards, kanban o listas repetidas, es común que el mismo botón o estado aparezca varias veces en pantalla.

En estos casos, use la **Referencia del objetivo** de Smart Web. Ella indica qué fila, card o item debe usarse como referencia antes de hacer clic, validar o extraer un valor.

Ejemplo:

```text
Valida que el estado esté Pagado en el pedido creado en esta ejecución,
no en cualquier pedido antiguo de la pantalla.
```

Esto ayuda a QANode a actuar en el registro correcto incluso cuando los datos cambian en cada ejecución.

---

## Sesiones Web Y Mobile

Cuando un escenario se divide en más de un nodo Smart Web o Mobile, informe si los nodos necesitan continuar en la misma sesión.

Use esto cuando:

- el primer nodo hace login y el segundo continúa autenticado;
- una compra o registro empieza en una pantalla y termina en otra;
- la app mobile necesita mantener el mismo estado entre partes del flujo.

Para sesiones web ya autenticadas, como sistemas enterprise con SSO, use sesiones persistidas con nombre. Para flujos comunes que hacen login durante la propia ejecución, una sesión temporal por run suele ser suficiente.

---

## Documentos De Proyecto

MCP permite que clientes de IA lean documentos adjuntos al proyecto como contexto para crear escenarios.

QANode puede leer:

- PDF con texto;
- PDF escaneado o imagen con OCR;
- DOCX;
- TXT, JSON y CSV;
- XLSX con vista previa de planillas;
- metadatos de otros archivos.

Para documentos grandes, pida leer páginas o rangos específicos.

---

## Dashboards

Puede pedir a un cliente de IA que cree dashboards o responda preguntas puntuales sobre los datos de QANode.

Ejemplos:

```text
¿Cuántas ejecuciones tuvimos en mayo?
```

```text
Crea un dashboard público con total de ejecuciones en el mes,
tasa de éxito, fallos recientes y evolución diaria.
```

Para widgets simples, QANode usa el constructor visual. Para consultas más complejas, el modo SQL puede usarse cuando el usuario tiene permiso.

---

## Defectos

Con MCP, un cliente de IA puede ayudar a investigar y acompañar defectos.

Ejemplos:

- abrir defecto a partir de una ejecución fallida;
- analizar logs y evidencias de la ejecución original;
- descargar adjuntos del defecto;
- comentar cuando el usuario lo pida;
- tramitar a una fila o usuario, cuando esté permitido;
- abrir sandbox para investigar sin afectar el escenario oficial.

El diagnóstico vuelve en el chat. No se convierte automáticamente en comentario público del defecto.

---

## Buenas Prácticas

- Use tokens diferentes para CI/CD y MCP cuando quiera separar auditoría.
- Revoque tokens que ya no estén en uso.
- Prefiera credenciales guardadas en lugar de pegar contraseñas en el prompt.
- Pida validaciones explícitas cuando el objetivo sea un escenario de prueba.
- Revise escenarios creados por automatización antes de usarlos en suites críticas.
- Use dashboards por rol cuando el panel tenga datos de un equipo específico.
- En defectos, pida que se explique el diagnóstico antes de tramitar o comentar.

---

## Próximos Pasos

- [Tokens de Integración](../ci-cd/tokens-integracion.md) — Genere y gobierne tokens usados por CI/CD y MCP
- [Smart Web Flow](../nodos/web/smart-web-flow.md) — Entienda grabación, localizadores y referencia del objetivo
- [Mobile Flow](../nodos/mobile/mobile-flow.md) — Entienda grabación mobile con Appium
- [Dashboard - Enterprise](../dashboard/dashboard-enterprise.md) — Vea paneles, widgets y SQL
- [Defectos - Descripción General](../defectos/descripcion-general.md) — Vea workflow, sandbox y adjuntos
