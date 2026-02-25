# Ejecución y Depuración

Aprende cómo ejecutar tus flujos, interpretar resultados y diagnosticar fallos.

---

## Ejecutando un Flujo

### Ejecución Completa

Para ejecutar todo el flujo:

1. Asegúrate de que el flujo esté guardado
2. Haz clic en el botón **Ejecutar** (▶️) en la barra superior
3. Espera la finalización — los nodos mostrarán indicadores de estado en tiempo real

[Flujo en ejecución](../../assets/images/executando.mp4)
*Imagen: Canvas mostrando nodos con indicadores de estado durante la ejecución*

### Estado de Ejecución

| Estado | Indicador | Descripción |
|--------|-----------|-------------|
| **Ejecutando** | 🔵 Azul/Pulsante | El nodo está siendo procesado |
| **Éxito** | 🟢 Verde | El nodo se ejecutó con éxito |
| **Fallo** | 🔴 Rojo | El nodo encontró un error |
| **Omitido** | ⚪ Gris | El nodo no fue ejecutado (rama inactiva) |

---

## Analizando Resultados

Después de la ejecución, haz clic en cualquier nodo para ver sus resultados en el panel de propiedades:

### Pestaña de Resultados

| Sección | Descripción |
|---------|-------------|
| **Estado** | Éxito o fallo, con duración |
| **Logs** | Mensajes registrados durante la ejecución del nodo |
| **Outputs** | Datos producidos por el nodo (JSON navegable) |
| **Error** | Mensaje de error detallado (cuando corresponda) |
| **Screenshots** | Capturas de pantalla (nodos web con evidencia activada) |

### Outputs

Los outputs de cada nodo quedan accesibles para nodos siguientes mediante expresiones. Por ejemplo, después de ejecutar un **HTTP Request**, los outputs serán:

```json
{
  "status": 200,
  "body": "...",
  "json": {
    "id": 1,
    "name": "João",
    "email": "joao@exemplo.com"
  }
}
```

Luego puedes acceder a estos datos en nodos siguientes:
```
{{ steps["http-request"].outputs.json.name }}  →  "João"
{{ steps["http-request"].outputs.status }}     →  200
```

### Screenshots (Evidencias)

Para nodos web (Web Flow y Smart Locators), las capturas de pantalla aparecen como miniaturas clicables. Haz clic para verlas a tamaño completo.

---

## Depurando Fallos

### Identificando el Nodo con Problema

1. Busca nodos con borde **rojo** (🔴) en el canvas
2. Haz clic en el nodo con fallo
3. Verifica el **mensaje de error** y los **logs**

### Mensajes de Error Comunes

#### Web Flow / Smart Locators

| Error | Causa | Solución |
|-------|-------|----------|
| `Timeout waiting for selector` | Elemento no encontrado en la página | Verifica el selector/localizador; aumenta el timeout |
| `Element not visible` | El elemento existe pero no está visible | Agrega un paso `wait` o `scroll` antes |
| `Navigation timeout` | La página no cargó a tiempo | Verifica la URL y la conectividad |
| `Element is not attached to DOM` | Elemento eliminado antes de la acción | Agrega un `wait` para estabilidad |

#### HTTP Request

| Error | Causa | Solución |
|-------|-------|----------|
| `ECONNREFUSED` | El servidor no está accesible | Verifica la URL y si el servidor está corriendo |
| `401 Unauthorized` | Autenticación inválida | Verifica el token/credenciales |
| `ETIMEOUT` | La solicitud superó el tiempo límite | Aumenta el timeout o verifica el servidor |

#### Base de Datos

| Error | Causa | Solución |
|-------|-------|----------|
| `Connection refused` | Base de datos no accesible | Verifica host, puerto y firewall |
| `Authentication failed` | Credenciales inválidas | Verifica usuario y contraseña |
| `Relation does not exist` | Tabla no encontrada | Verifica el nombre de la tabla y la base de datos |

#### SSH

| Error | Causa | Solución |
|-------|-------|----------|
| `Authentication failed` | Credenciales SSH inválidas | Verifica usuario, contraseña o clave privada |
| `Connection timeout` | Host no accesible | Verifica host, puerto y red |

### Usando Logs para Diagnóstico

Los **logs** de cada nodo proporcionan detalles sobre cada paso ejecutado. Para nodos web, los logs muestran:

```
Navigated to: https://exemplo.com/login
Filled: "usuario@exemplo.com" on [getByLabel("E-mail")]
Clicked: [getByRole("button", { name: "Entrar" })]
Assert passed: textContains "Bem-vindo" — true
```

Si un paso falló, el log mostrará exactamente cuál paso y por qué:

```
Navigated to: https://exemplo.com/login
Filled: "usuario@exemplo.com" on [getByLabel("E-mail")]
ERROR: Click failed after 3 attempts: [getByRole("button", { name: "Login" })] — element not found
```

---

## Consejos de Depuración

### 1. Desactiva el Modo Headless

Para pruebas web, desactiva el **modo headless** en el nodo para ver el navegador en acción:
- En el nodo Web Flow o Smart Locators, desmarca **Headless**
- Ejecuta el flujo y observa el navegador

### 2. Agrega Pasos de Wait

Si los elementos aparecen con retraso, agrega pasos de **wait** antes de interactuar:
- `wait` con modo `visible` espera que el elemento aparezca
- `wait` con modo `networkIdle` espera que todas las solicitudes de red terminen

### 3. Captura Screenshots para Diagnóstico

Activa las capturas de pantalla en modo **antes** para ver el estado de la página antes de cada acción. Esto ayuda a identificar si la página estaba en el estado esperado.

### 4. Verifica las Expresiones

Si un nodo falla con un valor inesperado, agrega un nodo **Log** antes de él para inspeccionar los valores:

```
Valor del token: {{ steps.login.outputs.json.token }}
Estado: {{ steps["http-request"].outputs.status }}
```

### 5. Usa el Toggle "Continuar en Fallo"

Para diagnosticar múltiples fallos a la vez, activa **Continuar en Fallo** en los nodos de verificación. Esto permite que el flujo continúe y puedas ver todos los fallos en una única ejecución.

---

## Reporte de Ejecución

Después de cada ejecución, QANode genera automáticamente un **reporte PDF** con:

- Resumen de la ejecución (estado, duración, fecha)
- Detalles de cada paso (estado, logs, outputs, duración)
- Capturas de pantalla tomadas
- Errores encontrados

El reporte está disponible en la lista de ejecuciones del proyecto y puede descargarse o enviarse por correo electrónico.

---

## Próximos Pasos

- [Referencia de Nodos](../nodos/control-flujo/if.md) — Detalles completos de cada tipo de nodo
- [Expresiones](../expresiones/expresiones.md) — Domina el sistema de datos dinámicos
