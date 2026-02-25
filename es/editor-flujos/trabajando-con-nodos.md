# Trabajando con Nodos

Esta guía detalla cómo agregar, conectar, configurar y gestionar nodos en el editor de flujos.

---

## Agregando Nodos

### Arrastrar desde la Paleta

La forma principal de agregar nodos es arrastrando desde la **paleta de nodos** (lado izquierdo) hacia el **canvas**:

1. Localiza la categoría deseada en la paleta
2. Haz clic y arrastra el nodo al canvas
3. Suéltalo en la posición deseada

[Arrastrando un nodo desde la paleta](../../assets/images/flow-editor.mp4)
*Imagen: Nodo siendo arrastrado desde la paleta de nodos al canvas*

### Pegar JSON (Grabador de Chrome)

También puedes pegar nodos copiados del **Grabador de Chrome** (extensión):

1. En el Grabador de Chrome, haz clic en **Copiar JSON**
2. En el editor de flujos, presiona **Ctrl+V**
3. El nodo Web Flow se agregará con todos los pasos grabados

---

## Conectando Nodos

### Creando Conexiones

Para conectar dos nodos:

1. Pasa el mouse sobre el **handle de salida** (●) del nodo origen
2. Haz clic y arrastra hasta el **handle de entrada** (●) del nodo destino
3. Suelta para crear la conexión

<!-- ![Conectando nodos](../../assets/images/conectar-nos.png) -->
*Imagen: Línea siendo arrastrada desde un handle de salida hasta un handle de entrada*

### Handles de Entrada y Salida

| Tipo | Posición | Descripción |
|------|----------|-------------|
| **Entrada** (in) | Parte superior del nodo | Recibe datos de nodos anteriores |
| **Salida** (out) | Parte inferior del nodo | Envía datos a nodos siguientes |

Algunos nodos tienen **múltiples handles de salida**:

- **If** → `true` y `false`
- **Switch** → Un handle por case + `default`
- **Loop** → `loop` (cuerpo del loop) y `done` (salida del loop)

### Eliminando Conexiones

Para eliminar una conexión:
1. Haz clic en la línea de conexión para seleccionarla
2. Presiona **Delete** o **Backspace**

---

## Configurando Nodos

### Panel de Propiedades

Al hacer clic en un nodo, el panel de propiedades se abre a la derecha. Cada tipo de nodo tiene campos específicos, pero todos comparten:

| Campo | Descripción |
|-------|-------------|
| **Label** | Nombre del nodo (mostrado en el canvas) |
| **Continuar en Fallo** | Si está activado, el flujo continúa aunque este nodo falle |

### Usando Expresiones en los Campos

La mayoría de los campos aceptan **expresiones** con la sintaxis `{{ }}`:

```
{{ steps["Nombre del Nodo"].outputs.propiedad }}
{{ variables.miVariable }}
```

Esto permite que los nodos usen datos producidos por nodos anteriores. Por ejemplo:

- **URL de navegación**: `{{ variables.BASE_URL }}/login`
- **Texto para completar**: `{{ steps["Nombre del Nodo"].outputs.result.email }}`
- **SQL**: `SELECT * FROM users WHERE email = '{{ steps.extract.outputs.extracts.email }}'`

> Para más detalles, consulta [Expresiones](../expresiones/expresiones.md).

---

## Nodos con Múltiples Pasos

Algunos nodos admiten **múltiples pasos internos**, haciéndolos más potentes:

### Web Flow y Smart Locators

Estos nodos permiten agregar varios pasos de automatización web dentro de un único nodo:

1. En el panel de propiedades, haz clic en **+ Agregar Paso**
2. Selecciona el tipo de acción (navigate, click, fill, assert, etc.)
3. Configura los parámetros del paso
4. Repite para agregar más pasos

Los pasos se ejecutan **secuencialmente**, en el orden en que aparecen en la lista.

Puedes:
- **Reordenar** pasos arrastrando
- **Expandir/Contraer** pasos para ver detalles
- **Eliminar** pasos haciendo clic en el ícono de papelera
- **Configurar evidencias** (capturas de pantalla) individualmente por paso

### SSH Command

El nodo SSH también admite múltiples pasos (comandos):

1. Cada paso es un comando a ejecutar
2. Los pasos se ejecutan secuencialmente en la misma conexión SSH
3. Opcionalmente, espera un texto específico en la salida (match string)

---

## Nodos de Control de Flujo

### Creando Ramas Condicionales

Para crear una rama condicional:

1. Agrega un nodo **If** al canvas
2. Configura la condición en el panel de propiedades
3. Conecta la salida **true** al camino que debe ejecutarse cuando la condición sea verdadera
4. Conecta la salida **false** al camino alternativo

**Ejemplo:**

```
[HTTP Request] → [If status === 200]
                       │ true → [Log "Éxito"]
                       │ false → [Log "Error"] → [Stop and Fail]
```

### Condición Simple vs. Constructor Visual

El nodo **If** ofrece dos modos de configuración:

**Modo Simple:**
```javascript
{{ steps["http-request"].outputs.status === 200 }}
```

**Modo Constructor Visual:**
- Campo: `steps["http-request"].outputs.status`
- Operador: `===`
- Valor: `200`

El modo constructor visual es más amigable y no requiere conocimientos de JavaScript.

---

## Nodos con Constructor de Consultas Visual

Los nodos de base de datos (PostgreSQL, MySQL, etc.) ofrecen un **constructor de consultas visual** además de la opción de SQL directo:

### Presets Disponibles

| Preset | Descripción |
|--------|-------------|
| **Custom SQL** | Escribe SQL libremente |
| **SELECT** | Constructor visual de SELECT |
| **EXISTS** | Verifica si existe un registro |
| **COUNT** | Cuenta registros |
| **ASSERT** | Verifica si un valor corresponde |
| **INSERT** | Constructor visual de INSERT |
| **UPDATE** | Constructor visual de UPDATE |
| **DELETE** | Constructor visual de DELETE |

El constructor de consultas permite seleccionar tablas, columnas, condiciones WHERE, ORDER BY y LIMIT sin escribir SQL.

---

## Consejos y Buenas Prácticas

### Nombra tus Nodos

Da nombres descriptivos a tus nodos cambiando el **label**. Esto facilita la lectura del flujo y hace las expresiones más claras:

```
{{ steps.login.outputs.json.token }}
```
es más legible que:
```
{{ steps["HTTP Request 2"].outputs.json.token }}
```

### Usa Grupos Lógicos

Organiza nodos relacionados cerca entre sí en el canvas. Por ejemplo, agrupa los nodos de inicio de sesión en un área y los nodos de verificación en otra.

### Configura Evidencias

Para pruebas web, activa las capturas de pantalla en los pasos críticos. Esto facilita la depuración y genera evidencias en los reportes.

### Usa Continuar en Fallo con Cuidado

El toggle **Continuar en Fallo** es útil para nodos de verificación (assert) donde quieres registrar todos los fallos en lugar de detenerte en el primero. Úsalo con moderación — en general, los fallos deben interrumpir el flujo.

---

## Próximos Pasos

- [Ejecución y Depuración](ejecucion-depuracion.md) — Cómo ejecutar y diagnosticar problemas
- [Referencia de Nodos](../nodos/control-flujo/if.md) — Detalles de cada tipo de nodo
