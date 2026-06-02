# Componentes Reutilizables 

Los **Componentes Reutilizables** permiten convertir una parte de una automatización en un bloque que puede reutilizarse en varios escenarios. En vez de copiar los mismos nodos de login, preparación de datos, consulta auxiliar o validación común en muchos escenarios, se crea un componente una vez, se definen sus entradas y salida, se publica y se usa dentro de los escenarios.

Piense en un componente como un **subflujo con contrato**:

- el componente recibe valores de entrada;
- ejecuta su propia secuencia de nodos;
- devuelve una salida al escenario que lo llamó.

---

## Cuándo Usar

Use componentes cuando exista una secuencia que se repite en muchos escenarios, por ejemplo:

- autenticarse en un sistema;
- crear datos de prueba estándar;
- consultar datos en API o base de datos y normalizar el resultado;
- ejecutar una validación de negocio común;
- encapsular una pequeña automatización web usada como preparación para otros tests.

Evite componentes para acciones muy específicas de un único escenario. Si la lógica aparece una sola vez, mantener los nodos directamente en el escenario suele ser más simple.

---

## Dónde Encontrar

En el menú lateral, acceda a **Componentes**.

La lista permite:

- crear un nuevo componente;
- buscar por nombre;
- filtrar por categoría;
- filtrar por estado **Publicado** o **Borrador**;
- copiar el nombre o ID del componente;
- eliminar componentes, respetando los permisos del usuario.

Cuando un componente está siendo usado por escenarios, QANode avisa antes de eliminarlo para evitar remociones accidentales.

---

## Creando un Componente

1. Acceda a **Componentes**.
2. Haga clic en **Nuevo Componente**.
3. Informe:
   - **Nombre**: nombre mostrado en la lista y en el editor de escenarios;
   - **Categoría**: grupo usado para organizar la paleta de componentes;
   - **Color**: color visual del componente en el canvas.
4. Cree el componente.

QANode abre el editor del componente. Usa la misma base visual del editor de flujos, pero en **modo componente**.

[Creando un Componente](../../assets/images/criar-componente.mp4)

---

## Editor del Componente

Todo componente tiene dos nodos especiales:

| Nodo | Función |
|------|---------|
| **Input** | Define los datos que el escenario debe enviar al componente |
| **Output** | Define el valor que el componente devuelve al escenario |

Estos nodos están protegidos en el editor. Forman parte del contrato del componente y no deben eliminarse.

Entre **Input** y **Output**, agregue nodos normales de QANode: Web, API, base de datos, utilidades, control de flujo y otros nodos disponibles en su ambiente.

---

## Entradas del Componente

En el nodo **Input**, configure los campos que el componente espera recibir.

| Campo | Descripción |
|-------|-------------|
| **Nombre del Campo** | Nombre usado para mapear el valor en el escenario |
| **Tipo** | `string`, `number`, `boolean`, `object`, `array` o `archivo` |
| **Obligatorio** | Define si el escenario debe completar este valor |
| **Datos de Prueba** | Valor usado al probar el componente directamente en el editor |

Ejemplo de entradas:

| Nombre | Tipo | Obligatorio |
|--------|------|-------------|
| `email` | `string` | Sí |
| `password` | `string` | Sí |
| `perfil` | `string` | No |
| `documento` | `archivo` | No |

[Definiendo entradas del Componente](../../assets/images/entradas-componente.mp4)

Durante la ejecución del componente, los nodos internos pueden usar estas entradas mediante expresiones, como cualquier otro output de nodo:

```
{{ steps.Input.outputs.email }}
```

Para campos de tipo **Archivo**, el valor recibido es un `fileRef`. En la prueba aislada del componente, el campo muestra un botón de upload para adjuntar el archivo de prueba. Dentro del componente, úselo normalmente:

```
{{ steps.Input.outputs.documento }}
```

Ese valor puede pasarse a nodos como **Extraer Archivo**, **HTTP Request**, **SSH Command**, **Mobile Flow** o **Custom JavaScript**.

> Use nombres simples y estables para los campos. Esto facilita el uso del componente en escenarios y evita expresiones difíciles de mantener.

---

## Salida del Componente

En el nodo **Output**, configure lo que el componente devuelve al escenario.

| Campo | Descripción |
|-------|-------------|
| **Nombre del Campo** | Nombre de la salida expuesta al escenario |
| **Tipo** | Tipo esperado del valor |
| **Valor** | Valor literal o expresión calculada a partir de los nodos internos |

[Definindo salida del Componente](../../assets/images/saida-componente.mp4)

Ejemplo:

```
Nombre: token
Tipo: string
Valor: {{ steps.login.outputs.body.token }}
```

Para devolver un archivo:

```
Nombre: reporte
Tipo: archivo
Valor: {{ steps["file-generate"].outputs.fileRef }}
```

El escenario que llama al componente podrá usar la salida como output del nodo de componente.

> En esta versión, el componente trabaja con una salida principal en el nodo Output. Si necesita devolver más de un dato, use un objeto como salida, por ejemplo `{ token, userId, role }`.

---

## Probando el Componente

Antes de publicar, pruebe el componente en su propio editor.

1. Complete **Datos de Prueba** en los campos obligatorios del nodo Input.
2. Haga clic en **Probar Componente**.
3. Acompañe la ejecución en el panel de runs del editor.

Las ejecuciones de prueba validan el componente de forma aislada. No sustituyen las ejecuciones oficiales de los escenarios que usarán el componente.

Si un campo obligatorio no tiene dato de prueba, QANode bloquea la ejecución de prueba hasta que se informe el valor.

[Probando el Componente](../../assets/images/testar-component.mp4)

---

## Guardando y Publicando

Al guardar el componente, QANode registra:

- el flujo interno del componente;
- el contrato de entrada;
- el contrato de salida;
- nombre, categoría y color.

Para que el componente aparezca en la paleta de escenarios, debe estar **Publicado**.

Reglas importantes:

- los componentes en borrador no aparecen para uso en escenarios;
- al cambiar el flujo interno, las entradas o la salida, el componente vuelve a requerir revisión/publicación;
- publicar confirma que el contrato actual está listo para usarse en otros escenarios.

[Guardando y Publicando a un componente](../../assets/images/salvar-e-publicar.mp4)

---

## Historial de Versiones — Enterprise

En QANode Enterprise, los componentes publicados pueden mantener historial de versiones.

Cada publicación relevante puede generar un snapshot con el flujo interno, contrato de entrada, contrato de salida y metadatos del componente. Esto permite consultar versiones anteriores y restaurar una versión cuando sea necesario.

Cómo usar:

1. Abra el componente.
2. Acceda al **Historial de Versiones**.
3. Seleccione una versión para visualizar en modo de consulta.
4. Use **Restaurar esta versión** cuando quiera volver al contrato y al flujo de ese snapshot.

> La cantidad de versiones retenidas por componente es definida por el Super Admin en **Configuración → General**.

---

## Usando en un Escenario

En el editor de escenarios:

1. Abra la pestaña **Componentes** en la paleta lateral.
2. Localice el componente publicado.
3. Arrastre el componente al canvas.
4. Conéctelo como un nodo común.
5. Complete los campos de entrada en el panel de propiedades.

Las entradas aceptan valores literales y expresiones:

```
email: {{ variables.USUARIO_TESTE }}
password: {{ variables.SENHA_TESTE }}
perfil: admin
```

Durante la ejecución, QANode ejecuta el componente como parte del escenario. Si una entrada obligatoria está vacía, el escenario se bloquea antes de ejecutar para evitar fallas difíciles de diagnosticar.

[Usando a un componente en un Escenario](../../assets/images/usar-em-cenario.mp4)

---

## Usando Outputs del Componente

Después de que el componente se ejecuta, su salida queda disponible como output del nodo en el escenario.

Ejemplo:

```
{{ steps.loginComponente.outputs.token }}
```

Si la salida principal es un objeto:

```
{{ steps.prepararUsuario.outputs.result.userId }}
{{ steps.prepararUsuario.outputs.result.email }}
```

Si la salida es un archivo:

```
{{ steps.generarReporte.outputs.reporte }}
```

El panel de variables muestra el archivo con nombre, tipo y tamaño en los detalles, pero el valor que debe arrastrarse a otros nodos es el `fileRef`.

[Usando Outputs del Componente](../../assets/images/usar-output-component.mp4)

---

## Permisos

En ambientes con control de acceso, los componentes respetan permisos propios:

| Permiso | Qué permite |
|---------|-------------|
| `component.view` | Ver la lista y abrir componentes |
| `component.create` | Crear y editar componentes propios |
| `component.edit` | Editar componentes creados por otros usuarios |
| `component.delete.own` | Eliminar componentes propios |
| `component.delete` | Eliminar cualquier componente |
| `component.publish` | Publicar componentes |

---

## Buenas Prácticas

- Use componentes para lógicas realmente compartidas.
- Mantenga el contrato pequeño y claro.
- Nombre entradas y salidas con términos de negocio, no detalles técnicos internos.
- Use categorías para organizar componentes por dominio, como `Login`, `Datos de prueba`, `Pedidos` o `CRM`.
- Pruebe el componente de forma aislada antes de publicar.
- Al cambiar un componente usado por muchos escenarios, valide al menos un escenario consumidor antes de liberarlo al equipo.
- Prefiera retornar un objeto cuando la salida necesita cargar varios valores relacionados.

---

## Limitaciones y Cuidados

- Los componentes publicados pueden ser usados por varios escenarios; cambios en el contrato pueden requerir ajustes en los escenarios consumidores.
- Los componentes en borrador no aparecen en la paleta del editor de escenarios.
- Los nodos Input y Output forman parte de la estructura del componente y no deben tratarse como nodos comunes.
- Si el componente depende de sesión web, credenciales o variables, documente eso en el nombre, categoría o estándar de uso del equipo.
