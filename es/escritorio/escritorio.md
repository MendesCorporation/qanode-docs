# Versión de Escritorio

La versión de escritorio de QANode es una aplicación **Community Edition** que no tiene restricciones en su núcleo principal de flujos.

---

## Ventajas de la Versión de Escritorio

| Característica | Descripción |
|----------------|-------------|
| **Instalación Simple** | Un único instalador, sin dependencias externas |
| **Base de Datos Integrada** | PostgreSQL integrado, sin necesidad de configuración |
| **Nodos Locales** | Cree nodos personalizados como archivos locales (hot-reload) |
| **Todo Integrado** | Frontend, API, Worker y base de datos en una sola app |
| **Portátil** | Funciona sin conexión, sin servidor |

---

## Instalación

Consulte la guía de [Instalación](../guia-inicio/instalacion.md) para obtener instrucciones detalladas.

### Requisitos Mínimos

- **Windows** 10/11 (64-bit)
- **RAM**: 4 GB mínimo (8 GB recomendado)
- **Disco**: 500 MB para instalación + espacio para datos

---

## Primera Ejecución

En la primera ejecución, QANode:

1. Muestra una **pantalla de bienvenida** con progreso
2. Inicializa el **PostgreSQL integrado**
3. Ejecuta las **migraciones** (crea tablas)
4. Inicia el **servidor de la API**
5. Abre la **interfaz principal**

<!-- ![Pantalla de bienvenida](../../assets/images/splash-screen.png) -->
*Imagen: Pantalla de bienvenida mostrando el progreso de inicialización con barra de estado*

> La primera inicialización puede tomar algunos segundos adicionales mientras se prepara la base de datos.

---

## Estructura de Datos

QANode almacena datos en la carpeta del usuario:

```
%APPDATA%\QANode\
├── pg-data/              # Datos del PostgreSQL integrado
├── storage/              # Artefactos (capturas de pantalla, PDFs)
└── logs/                 # Registros de la aplicación
```

```
%USERPROFILE%\Documents\QANode\
└── custom nodes/         # Nodos personalizados locales
    └── sample-hash-node/ # Nodo de ejemplo
```

> **Importante:** No modifique manualmente los archivos en `pg-data/`.

---

## Nodos Personalizados Locales

La versión de escritorio incluye soporte para **nodos locales** — archivos JavaScript que son detectados y cargados automáticamente.

### Carpeta de Nodos

```
%USERPROFILE%\Documents\QANode\custom nodes\
```

### Creando un Nodo Local

1. Cree un archivo con extensión `.node.js` en la carpeta anterior
2. Exporte `manifest` y `execute`
3. QANode lo detecta automáticamente en ~1.5 segundos

```javascript
// meu-no.node.js
export const manifest = {
  type: 'meu-no',
  name: 'Meu Nó',
  category: 'Meus Nós',
  inputSchema: {
    texto: { type: 'string', required: true }
  },
  outputSchema: {
    resultado: { type: 'string' }
  }
};

export async function execute({ inputs }) {
  return {
    status: 'success',
    outputs: { resultado: inputs.texto.toUpperCase() },
    logs: [`Processado: ${inputs.texto}`],
    artifacts: []
  };
}
```

> Para más detalles, consulte [Nodos Locales de Escritorio](../nodos-personalizados/nodos-locales-escritorio.md).

---

## Hot-Reload

El proveedor local monitorea la carpeta de nodos personalizados y recarga automáticamente cuando detecta cambios:

- **Intervalo**: 1.5 segundos
- **Detección**: Fecha de modificación + tamaño del archivo
- **Recarga**: Automática, sin reiniciar la app

Ideal para el desarrollo rápido de nodos personalizados.

---

## Tema Nativo

QANode de escritorio admite temas claro y oscuro que se integran con la barra de título nativa de Windows:

- **Tema Claro** — Interfaz clara con barra de título blanca
- **Tema Oscuro** — Interfaz oscura con barra de título oscura

El cambio de tema puede realizarse desde la interfaz y se sincroniza con la barra de título del sistema operativo.

---

## Diferencias con la Versión Web

| Aspecto | Escritorio | Web (Enterprise) |
|---------|------------|------------------|
| **Base de Datos** | PostgreSQL integrado | PostgreSQL externo |
| **Instalación** | Instalador único | Manual (Node.js + PostgreSQL) |
| **Nodos Locales** | Admitidos (hot-reload) | Solo proveedores HTTP |
| **Multiusuario** | Uso individual | Equipo compartido |
| **Red** | Funciona sin conexión | Requiere red |
| **Actualizaciones** | Instalador manual | Git pull + rebuild |

---

## Solución de Problemas

### La app no inicia

- Verifique que no haya otra instancia en ejecución
- Intente ejecutar como administrador
- Verifique que el puerto 5432 no esté en uso (un PostgreSQL externo puede generar conflictos)

### La base de datos no inicializa

- La carpeta `%APPDATA%\QANode\pg-data` puede estar dañada
- Intente renombrar/eliminar la carpeta y reiniciar (los datos se perderán)

### Los nodos locales no aparecen

- Verifique que el archivo tenga extensión `.node.js`, `.node.mjs` o `.node.cjs`
- Verifique que exporte `manifest` y `execute`
- Abra las DevTools (Ctrl+Shift+I) para ver los registros de error

---

## Consejos

- Use la versión de escritorio para **desarrollo y pruebas individuales**
- Aproveche los **nodos locales** para prototipado rápido
- El **hot-reload** elimina la necesidad de reiniciar durante el desarrollo
