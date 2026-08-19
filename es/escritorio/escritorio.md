# Version de Escritorio

QANode Desktop es una aplicacion **Community Edition** sin restricciones en las funciones principales de flujo.

---

## Ventajas del Escritorio

| Caracteristica | Descripcion |
|---|---|
| **Instalacion simple** | Un instalador unico, sin dependencias externas |
| **Base de datos embebida** | PostgreSQL integrado, sin configuracion manual |
| **Nodos locales** | Crear nodos personalizados como archivos locales (hot-reload) |
| **Todo en uno** | Frontend, API, Worker y base de datos en una sola app |
| **Portable** | Funciona offline, sin servidor |

---

## Instalacion

Consulta [Instalacion](../guia-inicio/instalacion.md) para instrucciones completas.

### Requisitos Minimos

- **Windows** 10/11 (64 bits), **macOS** 11+ (Intel o Apple Silicon) o **Linux** 64 bits con AppImage
- **RAM**: 4 GB minimo (8 GB recomendado)
- **Disco**: 2 GB para instalacion + espacio para datos y evidencias

---

## Primera Ejecucion

En el primer inicio, QANode:

1. Muestra progreso en splash screen
2. Inicializa PostgreSQL embebido
3. Ejecuta migraciones (crea tablas)
4. Inicia servidor API
5. Abre la interfaz principal

> La primera inicializacion puede tardar algunos segundos extra mientras termina la preparacion de la base de datos.

---

## Estructura de Datos

QANode guarda datos en carpetas de usuario:

```text
Windows: %APPDATA%\@qanode\desktop\
macOS:   ~/Library/Application Support/@qanode/desktop/
Linux:   ~/.config/@qanode/desktop/
```

Dentro de la carpeta de datos:

```text
postgres/          # Datos de PostgreSQL embebido
storage/           # Artefactos, screenshots y PDFs
desktop-main.log   # Log de inicio y servicios locales
```

```text
%USERPROFILE%\Documents\QANode\
└── custom nodes/         # Nodos personalizados locales
    └── sample-hash-node/ # Nodo de ejemplo
```

> Importante: no modifique manualmente los archivos dentro de `postgres/`. Haga una copia de seguridad de la carpeta de datos completa antes de cualquier procedimiento de recuperación.

---

## Nodos Personalizados Locales

Desktop soporta nodos locales: archivos JavaScript detectados y cargados automaticamente.

### Carpeta de Nodos

```text
%USERPROFILE%\Documents\QANode\custom nodes\
```

### Crear un Nodo Local

1. Crea un archivo con extension `.node.js`, `.node.mjs` o `.node.cjs`
2. Exporta `manifest` y `execute`
3. QANode detecta cambios automaticamente en ~1.5 segundos

### Ejemplo: `.node.js` (CommonJS)

```javascript
// mi-nodo.node.js
module.exports = {
  manifest: {
    type: 'mi-nodo',
    name: 'Mi Nodo',
    category: 'Mis Nodos',
    inputSchema: {
      texto: { type: 'string', required: true },
    },
    outputSchema: {
      resultado: { type: 'string' },
    },
  },
  async execute({ inputs }) {
    return {
      status: 'success',
      outputs: { resultado: String(inputs.texto || '').toUpperCase() },
      logs: [`Procesado: ${inputs.texto}`],
      artifacts: [],
    };
  },
};
```

### Ejemplo: `.node.mjs` (ESM)

```javascript
// mi-nodo.node.mjs
export const manifest = {
  type: 'mi-nodo-mjs',
  name: 'Mi Nodo MJS',
  category: 'Mis Nodos',
  inputSchema: {
    texto: { type: 'string', required: true },
  },
  outputSchema: {
    resultado: { type: 'string' },
  },
};

export async function execute({ inputs }) {
  return {
    status: 'success',
    outputs: { resultado: String(inputs.texto || '').toUpperCase() },
    logs: [`Procesado: ${inputs.texto}`],
    artifacts: [],
  };
}
```

> Detalles completos: [Nodos Locales Desktop](../nodos-personalizados/nodos-locales-escritorio.md).

---

## Hot Reload

El provider local monitorea la carpeta de nodos y recarga automaticamente:

- **Intervalo**: 1.5 segundos
- **Deteccion**: fecha de modificacion + tamano de archivo
- **Recarga**: automatica, sin reiniciar la app

---

## Tema Nativo

QANode Desktop admite temas claro y oscuro integrados con la ventana nativa del sistema operativo:

- **Claro**: interfaz clara con barra de titulo clara
- **Oscuro**: interfaz oscura con barra de titulo oscura

---

## Escritorio vs Web

| Aspecto | Escritorio | Web (Enterprise) |
|---|---|---|
| Base de datos | PostgreSQL embebido | PostgreSQL externo |
| Instalacion | Instalador unico | Manual (Node.js + PostgreSQL) |
| Nodos locales | Soportado | Solo providers HTTP |
| Multiusuario | Uso individual | Uso en equipo |
| Red | Funciona offline | Requiere red |
| Actualizaciones | Actualizador integrado | Actualizacion del entorno de despliegue |

---

## Solucion de Problemas

### La app no inicia

- Verifica que no haya otra instancia ejecutandose
- Prueba ejecutar como administrador
- Revisa conflictos de puertos

### La base de datos no inicia

- Abra `desktop-main.log` en la carpeta de datos y busque `Fatal startup error`
- No elimine la carpeta `postgres/`: contiene proyectos, escenarios e historiales locales
- Haga una copia de la carpeta de datos completa antes de cualquier procedimiento de recuperación
- Envíe el fragmento correspondiente del log al soporte para identificar si el fallo ocurrió durante el inicio, la actualización del esquema u otro servicio local

### Los nodos locales no aparecen

- Verifica extension: `.node.js`, `.node.mjs`, `.node.cjs`
- Verifica exports: `manifest` y `execute`
- Para `.node.js`, usa CommonJS (`module.exports`) o configura `type: "module"` en `package.json`
- Abra `desktop-main.log` en la carpeta de datos de su sistema y busque `Failed loading`

### Verificacion rapida de errores de importacion

1. Abre **Configuraciones > Providers** y prueba `Desktop Local JS Provider`
2. Revise `desktop-main.log` en la carpeta de datos de su sistema
3. Busca mensajes como:
- `Failed loading ".../tu-nodo.node.mjs": Cannot find package 'x'`
- `Unexpected token 'export'` (normalmente `.node.js` con sintaxis ESM sin `type: "module"`)

---

## Consejos

- Usa `.node.js` con CommonJS para compatibilidad simple
- Usa `.node.mjs` cuando necesites ESM explicito
- Mantén una carpeta por nodo con su `package.json`
- Aprovecha hot-reload para iterar rapido
