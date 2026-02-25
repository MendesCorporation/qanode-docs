# Inicio Rápido

En esta guía, crearás tu primera prueba automatizada en QANode en pocos minutos. Crearemos una prueba web simple que navega hasta una página, completa un formulario y verifica el resultado.

---

## Paso 1: Crea un Proyecto

1. En el menú lateral, haz clic en **Proyectos**
2. Haz clic en el botón **+ Nuevo Proyecto**
3. Completa:
   - **Nombre**: `Mi Primer Proyecto`
   - **Estado**: Activo
4. Haz clic en **Crear**

[Creando un proyecto](../../assets/images/criar-projeto.mp4)

*Imagen: Modal de creación de proyecto con campos de nombre y estado*

---

## Paso 2: Crea un Flujo (Escenario de Prueba)

1. Dentro del proyecto, haz clic en la pestaña **Escenarios**
2. Haz clic en **+ Nuevo Escenario**
3. Ingresa el nombre: `Inicio de sesión exitoso`
4. Serás llevado al **Editor de Flujos**

[Editor de flujos vacío](../../assets/images/editor-fluxo.mp4)
*Imagen: Editor de flujos con el canvas vacío y la paleta de nodos a la izquierda*

---

## Paso 3: Agrega los Nodos

### Agregando un nodo Web Flow

1. En la paleta de nodos a la izquierda, localiza la categoría **Web**
2. Arrastra el nodo **Smart Locators** al canvas
3. Haz clic en el nodo para abrir el panel de propiedades a la derecha

### Configurando los Pasos

Usaremos el sitio de práctica **the-internet.herokuapp.com**, una página pública creada para pruebas de automatización. La propia página muestra las credenciales: usuario `tomsmith` y contraseña `SuperSecretPassword!`.

En el panel de propiedades, configura los siguientes pasos:

#### Paso 1: Navegar a la página de inicio de sesión

1. Haz clic en **+ Agregar Paso**
2. Selecciona la acción **Navigate**
3. En el campo **URL**, escribe: `https://the-internet.herokuapp.com/login`

#### Paso 2: Completar el campo de usuario

1. Haz clic en **+ Agregar Paso**
2. Selecciona la acción **Fill**
3. Configura el localizador:
   - **Método**: `getByLabel`
   - **Valor**: `Username`
4. En el campo **Texto**, escribe: `tomsmith`

#### Paso 3: Completar la contraseña

1. Haz clic en **+ Agregar Paso**
2. Selecciona la acción **Fill**
3. Configura el localizador:
   - **Método**: `getByLabel`
   - **Valor**: `Password`
4. En el campo **Texto**, escribe: `SuperSecretPassword!`

#### Paso 4: Hacer clic en el botón de inicio de sesión

1. Haz clic en **+ Agregar Paso**
2. Selecciona la acción **Click**
3. Configura el localizador:
   - **Método**: `getByRole`
   - **Valor**: `button`
   - **Nombre**: `Login`

#### Paso 5: Verificar el resultado

1. Haz clic en **+ Agregar Paso**
2. Selecciona la acción **Assert**
3. Configura:
   - **Modo**: `textContains`
   - **Localizador**: `getByText` → `You logged into a secure area!`
   - **Texto Esperado**: `You logged into a secure area!`

[Pasos configurados](../../assets/images/configurando-passos.mp4)
*Imagen: Panel de propiedades de Smart Locators con 5 pasos configurados*

---

## Paso 4: Guarda y Ejecuta

1. Haz clic en el botón **Guardar** (ícono de disquete) en la barra superior
2. Haz clic en el botón **Ejecutar** (ícono de play) ▶️

QANode:
1. Abrirá un navegador (modo headless por defecto)
2. Navegará a `https://the-internet.herokuapp.com/login`
3. Completará el campo Username con `tomsmith`
4. Completará el campo Password con `SuperSecretPassword!`
5. Hará clic en el botón Login
6. Verificará si el mensaje `You logged into a secure area!` aparece en la página

### Resultado de la Ejecución

Después de la ejecución, cada paso mostrará su estado:

- ✅ **Verde**: Paso ejecutado con éxito
- ❌ **Rojo**: Paso falló
- ⏭️ **Gris**: Paso no ejecutado (omitido)

[Resultado de la ejecución](../../assets/images/execucao.mp4)
*Imagen: Editor mostrando los nodos con indicadores de éxito/fallo y el panel de resultados*

Haz clic en cualquier nodo para ver detalles:
- **Logs**: Mensajes de cada paso ejecutado
- **Outputs**: Datos extraídos (textos, valores, etc.)
- **Screenshots**: Capturas de pantalla (si están configuradas)
- **Duración**: Tiempo de ejecución de cada paso

---

## Paso 5: Agrega Capturas de Pantalla (Evidencias)

Para capturar evidencias durante la ejecución:

1. Expande cualquier paso en el panel de propiedades
2. En la sección **Evidencia**, activa **Capturar Screenshot**
3. Selecciona el modo:
   - **Antes**: Captura antes de ejecutar el paso
   - **Después**: Captura después de ejecutar el paso
   - **Ambos**: Captura antes y después

Las capturas de pantalla se almacenarán como artefactos de la ejecución y se incluirán en los reportes PDF.

---

## Próximos Pasos

¡Felicitaciones! Creaste y ejecutaste tu primera prueba automatizada. Ahora explora:

- [Conceptos Fundamentales](conceptos.md) — Comprende mejor la estructura de QANode
- [Editor de Flujos](../editor-flujos/vision-general.md) — Domina el editor visual
- [Referencia de Nodos](../nodos/control-flujo/if.md) — Conoce todos los nodos disponibles
- [Expresiones](../expresiones/expresiones.md) — Usa datos dinámicos en tus pruebas
- [Suites de Prueba](../suites/suites.md) — Agrupa y programa tus pruebas

---

## Ejemplo con API

¿Quieres probar una API? Aquí hay un ejemplo rápido usando **JSONPlaceholder**, una API REST pública y gratuita para pruebas:

1. Arrastra un nodo **HTTP Request** al canvas
2. Configura:
   - **Método**: `GET`
   - **URL**: `https://jsonplaceholder.typicode.com/users/1`
3. Arrastra un nodo **If** y conéctalo a la salida del HTTP Request
4. Configura la condición: `{{ steps["http-request"].outputs.status === 200 }}`
5. En la salida **true**, agrega un nodo **Log** con el mensaje: `Usuario encontrado: {{ steps["http-request"].outputs.json.name }}`

Este flujo hace una solicitud GET a JSONPlaceholder y verifica si el estado es 200. Si es así, muestra el nombre del usuario retornado (`Leanne Graham`).
