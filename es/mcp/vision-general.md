# MCP — Integración con IA

> **Disponible en:** QANode Enterprise

El **MCP** conecta herramientas de IA con QANode para que el usuario trabaje con proyectos, escenarios, ejecuciones, dashboards y defectos directamente desde la IDE, un chat corporativo u otro cliente compatible.

En la práctica, el cliente conectado al MCP llama recursos de QANode usando los permisos del usuario autenticado. Todo lo creado o modificado sigue existiendo dentro de QANode como un objeto normal de la plataforma.

---

## Cuándo Usar

Use MCP cuando quiera trabajar con QANode sin salir del entorno donde ya está investigando, documentando o implementando:

- crear o ajustar escenarios en QANode con asistencia de IA;
- automatizar flujos que pasan por web, API, base de datos, archivos, SSH o mobile;
- investigar un escenario existente que está fallando;
- transformar documentación de proyecto en escenarios iniciales;
- crear dashboards o consultar métricas de QANode en lenguaje natural;
- abrir o analizar defectos mientras trabaja en el código.

MCP no reemplaza el editor visual. Todo lo creado sigue apareciendo en QANode como proyecto, escenario, nodo, ejecución, dashboard, suite o defecto normal.

---

## Cómo Conectar

1. Genere un token en **Configuración → Tokens de Acceso**
2. Configure el cliente MCP compatible
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

El formato exacto depende del cliente compatible, pero la URL y el header son los mismos.

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

El token no concede acceso extra. Solo autentica el cliente conectado como ese usuario o integración.

---

## Qué Permite Hacer MCP

| Área | Qué queda disponible |
|------|-----------------|
| **Proyectos** | Buscar proyectos, consultar documentos adjuntos y usar ese contexto en escenarios o diagnósticos |
| **Escenarios** | Crear flujos, importar nodos, agregar etapas, conectar, validar y ejecutar |
| **Smart Web** | Crear automatizaciones web, validar pantallas, extraer datos y reutilizar sesiones cuando sea necesario |
| **Mobile** | Crear jornadas Android/iOS con Appium y guardarlas como Mobile Flow |
| **Base de Datos** | Consultar schema y crear consultas o validaciones de base |
| **Archivos** | Generar, extraer y reutilizar archivos en flujos |
| **Variables** | Crear variables globales o de proyecto para reutilización |
| **Credenciales** | Registrar credenciales a partir de los datos proporcionados por el usuario |
| **Suites** | Crear, editar, ejecutar, programar y diagnosticar fallos |
| **Defectos** | Abrir, comentar, adjuntar, tramitar e investigar defectos |
| **Dashboards** | Crear paneles, widgets y consultas rápidas sobre datos de QANode |
| **Componentes** | Crear componentes reutilizables para flujos recurrentes |

---

## Cómo Funciona Una Solicitud

Cuando el usuario solicita una acción desde el cliente conectado, el flujo general es:

1. El cliente envía la solicitud a QANode por MCP.
2. QANode valida autenticación, permisos y datos necesarios.
3. La acción se aplica en objetos reales de QANode, como escenario, ejecución, dashboard o defecto.
4. El resultado vuelve al cliente con los datos disponibles para ese usuario.

Para escenarios y pruebas, información como proyecto, sistema objetivo, credencial, datos de prueba, resultado esperado y reglas de limpieza ayuda a QANode a crear un flujo más cercano al uso real.

Ejemplo:

```text
En el proyecto Task Manager, crea un escenario que acceda a localhost:3008,
haga login con el usuario de prueba, cree una tarea, la marque como completada
y valide en la base que el estado quedó completed.
```

Las acciones sensibles, como usar credenciales, crear datos en sistemas reales, ejecutar escenarios destructivos o tramitar defectos, deben tratarse con el mismo cuidado que tendrían dentro de QANode.

---

## Smart Web Y Referencia Del Objetivo

En páginas con tablas, cards, kanban o listas repetidas, es común que el mismo botón o estado aparezca varias veces en pantalla.

La **Referencia del objetivo** de Smart Web registra qué fila, card o item debe servir como contexto antes de hacer clic, validar o extraer un valor. Esto vale tanto para flujos creados en el editor como para flujos creados o ajustados por MCP.

Ejemplo:

```text
Valida que el estado esté Pagado en el pedido creado en esta ejecución,
no en cualquier pedido antiguo de la pantalla.
```

Esto ayuda a QANode a actuar en el registro correcto incluso cuando los datos cambian en cada ejecución.

---

## Sesiones Web Y Mobile

Cuando un escenario se divide en más de un nodo Smart Web o Mobile, la reutilización de sesión define si los nodos continúan en el mismo navegador, contexto web o sesión mobile.

Esto es útil cuando:

- el primer nodo hace login y el segundo continúa autenticado;
- una compra o registro empieza en una pantalla y termina en otra;
- la app mobile necesita mantener el mismo estado entre partes del flujo.

Para sesiones web ya autenticadas, como sistemas enterprise con SSO, use sesiones persistidas con nombre. Para flujos comunes que hacen login durante la propia ejecución, una sesión temporal por run suele ser suficiente.

Cuando un nodo posterior necesita reutilizar la sesión abierta por un nodo anterior, el **Almacenamiento de sesión** debe mantener esa sesión disponible. El comportamiento es el mismo en el editor y en acciones hechas por MCP.

---

## Nodos Omitidos en Ejecuciones

Los nodos marcados como **Omitir durante la ejecución** siguen visibles en el canvas, pero las ejecuciones normales registran esos nodos como omitidos y continúan el flujo sin ejecutarlos.

Este comportamiento también vale para ejecuciones iniciadas por MCP o escenarios editados por MCP. Es útil para borradores, experimentos y partes temporalmente aisladas del flujo, pero no reemplaza la corrección de un fallo real.

---

## Documentos De Proyecto

MCP expone documentos adjuntos al proyecto como contexto para crear escenarios, responder dudas o entender reglas de negocio.

Para documentos grandes, QANode permite buscar primero fragmentos relevantes dentro de los adjuntos del proyecto. Así el cliente encuentra páginas, secciones o partes del texto antes de abrir el documento completo.

QANode puede leer:

- PDF con texto;
- PDF escaneado o imagen con OCR;
- DOCX;
- TXT, JSON y CSV;
- XLSX con vista previa de planillas;
- metadatos de otros archivos.

Cuando el fragmento encontrado no sea suficiente, el cliente puede solicitar una página, un rango de páginas o el documento completo.

---

## Documentación Oficial de QANode

Además de los documentos de su proyecto, MCP también puede consultar la documentación oficial de QANode.

Este recurso ayuda a entender cómo configurar un nodo, confirmar un comportamiento del producto o localizar la página correcta de la documentación sin salir del cliente conectado.

Ejemplos:

```text
¿Cómo configuro un Smart Web Flow con sesión persistida?
```

```text
¿Qué opciones existen para crear widgets de dashboard?
```

La búsqueda usa el idioma solicitado cuando está disponible y devuelve los fragmentos más relevantes de la documentación.

---

## Dashboards

Por MCP, el usuario puede crear dashboards o consultar métricas de QANode en lenguaje natural.

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

Con MCP, el usuario puede investigar y acompañar defectos desde el cliente conectado.

Ejemplos:

- abrir defecto a partir de una ejecución fallida;
- analizar logs y evidencias de la ejecución original;
- descargar adjuntos del defecto;
- comentar cuando sea necesario;
- tramitar a una fila o usuario, cuando esté permitido;
- abrir sandbox para investigar sin afectar el escenario oficial.

El diagnóstico vuelve al cliente conectado. No se convierte automáticamente en comentario público del defecto.

---

## Buenas Prácticas

- Use tokens diferentes para CI/CD y MCP cuando quiera separar auditoría.
- Revoque tokens que ya no estén en uso.
- Prefiera credenciales guardadas en lugar de pegar contraseñas en el prompt.
- Defina validaciones explícitas cuando el objetivo sea un escenario de prueba.
- Revise escenarios creados por automatización antes de usarlos en suites críticas.
- Use dashboards por rol cuando el panel tenga datos de un equipo específico.
- En defectos, revise el diagnóstico antes de tramitar o comentar.

---

## Próximos Pasos

- [Tokens de Integración](../ci-cd/tokens-integracion.md) — Genere y gobierne tokens usados por CI/CD y MCP
- [Smart Web Flow](../nodos/web/smart-web-flow.md) — Entienda grabación, localizadores y referencia del objetivo
- [Mobile Flow](../nodos/mobile/mobile-flow.md) — Entienda grabación mobile con Appium
- [Dashboard - Enterprise](../dashboard/dashboard-enterprise.md) — Vea paneles, widgets y SQL
- [Defectos - Descripción General](../defectos/descripcion-general.md) — Vea workflow, sandbox y adjuntos
