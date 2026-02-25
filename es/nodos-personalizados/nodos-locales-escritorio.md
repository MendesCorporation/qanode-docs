# Nodos Locales en la Versión Desktop

En la versión desktop de QANode, puedes crear nodos personalizados **localmente** como archivos JavaScript, sin necesitar un servidor HTTP separado. QANode detecta automáticamente los archivos y pone los nodos a disposición en la paleta.

---

## Cómo Funciona

La versión desktop incluye un **proveedor local integrado** que:

1. Monitorea una carpeta en tu computadora
2. Detecta archivos `.node.js`, `.node.mjs` o `.node.cjs`
3. Carga automáticamente los nodos definidos en esos archivos
4. Recarga cuando detecta cambios (**hot-reload**)

---

## Carpeta de Nodos Personalizados

Los nodos locales se encuentran en la carpeta:

```
%USERPROFILE%\Documents\QANode\custom nodes\
```

En Windows, generalmente:
```
C:\Users\TuUsuario\Documents\QANode\custom nodes\
```

La organización recomendada es **una carpeta por nodo**, cada una con el archivo `.node.js`, archivos auxiliares y `package.json` para las dependencias:

```
custom nodes/
├── validar-cpf/
│   ├── validar-cpf.node.js    # Definición del nodo
│   ├── cpf-utils.js           # Auxiliar (NO usa .node.)
│   └── package.json           # Dependencias del nodo
├── formatar-data/
│   ├── formatar-data.node.js
│   └── package.json
└── sample-hash-node/          # Nodo de ejemplo (creado automáticamente)
    ├── sample-hash.node.js
    └── package.json
```

> **Importante:** Solo los archivos con `.node.` en el nombre son reconocidos como nodos. Otros archivos `.js` en la misma carpeta son ignorados y pueden usarse como módulos auxiliares.

---

## Formato del Archivo

Cada archivo `.node.js` debe exportar dos elementos:

1. **`manifest`** — Definición del nodo (tipo, nombre, inputs, outputs)
2. **`execute`** — Función de ejecución

### Estructura Básica

```javascript
// meu-no/meu-no.node.js

export const manifest = {
  type: 'meu-no-customizado',
  name: 'Meu Nó Customizado',
  category: 'Meus Nós',
  timeoutMs: 600000, // 10 minutos (opcional, predeterminado: 30 minutos)
  inputSchema: {
    campo1: { type: 'string', required: true, description: 'Descrição do campo' },
    campo2: { type: 'number', default: 10 }
  },
  outputSchema: {
    resultado: { type: 'string' },
    processedAt: { type: 'string' }
  }
};

export async function execute({ inputs, runId, nodeId }) {
  const resultado = `Processado: ${inputs.campo1} (x${inputs.campo2})`;

  return {
    status: 'success',
    logs: [
      `Recebido: campo1="${inputs.campo1}", campo2=${inputs.campo2}`,
      `Resultado: ${resultado}`
    ],
    outputs: {
      resultado,
      processedAt: new Date().toISOString()
    },
    artifacts: []
  };
}
```

---

## Ejemplo Completo: Hash Generator

Este es el nodo de ejemplo que QANode crea automáticamente en la primera ejecución:

```javascript
// sample-hash-node/sample-hash.node.js

import crypto from 'crypto';

export const manifest = {
  type: 'sample-hash',
  name: 'Sample Hash Generator',
  category: 'Custom Nodes',
  inputSchema: {
    text: {
      type: 'string',
      required: true,
      description: 'Text to hash'
    },
    algorithm: {
      type: 'string',
      enum: ['md5', 'sha256', 'sha512'],
      default: 'sha256',
      description: 'Hash algorithm'
    }
  },
  outputSchema: {
    hash: { type: 'string' },
    algorithm: { type: 'string' },
    timestamp: { type: 'string' }
  }
};

export async function execute({ inputs }) {
  const { text, algorithm = 'sha256' } = inputs;
  const hash = crypto.createHash(algorithm).update(text).digest('hex');

  return {
    status: 'success',
    logs: [
      `Input: "${text.substring(0, 50)}"`,
      `Algorithm: ${algorithm}`,
      `Hash: ${hash}`
    ],
    outputs: {
      hash,
      algorithm,
      timestamp: new Date().toISOString()
    },
    artifacts: []
  };
}
```

---

## Hot-Reload

QANode monitorea la carpeta de nodos personalizados y **recarga automáticamente** cuando detecta cambios:

- **Intervalo de verificación**: 1.5 segundos
- **Detección**: Basada en fecha de modificación y tamaño del archivo
- **Recarga**: Automática, sin necesidad de reiniciar QANode

### Flujo de Desarrollo

1. Crea o edita un archivo `.node.js` en la carpeta de nodos
2. Guarda el archivo
3. En ~1.5 segundos, QANode detecta el cambio
4. El nodo actualizado aparece en la paleta del editor

> **Consejo:** Abre la consola de QANode (DevTools) para ver los logs de recarga:
> ```
> [CustomProvider] Reloaded: meu-no.node.js (2 nodes)
> ```

---

## Usando Dependencias NPM

Los nodos locales pueden usar módulos nativos de Node.js (`crypto`, `path`, `fs`, etc.) directamente. Para dependencias npm externas, cada nodo tiene su propio `package.json`:

```
custom nodes/
└── formatar-data/
    ├── formatar-data.node.js   # Definición del nodo
    ├── date-helpers.js         # Auxiliar (sin .node.)
    └── package.json            # Dependencias del nodo
```

```bash
cd "%USERPROFILE%\Documents\QANode\custom nodes\formatar-data"
npm init -y
npm install dayjs
```

```javascript
// formatar-data/formatar-data.node.js
import dayjs from 'dayjs';

export const manifest = {
  type: 'formatar-data',
  name: 'Formatar Data',
  inputSchema: {
    data: { type: 'string', required: true },
    formato: { type: 'string', default: 'DD/MM/YYYY' }
  },
  outputSchema: {
    formatted: { type: 'string' }
  }
};

export async function execute({ inputs }) {
  const formatted = dayjs(inputs.data).format(inputs.formato);
  return {
    status: 'success',
    logs: [`${inputs.data} → ${formatted}`],
    outputs: { formatted },
    artifacts: []
  };
}
```

---

## Diferencias: Nodos Locales vs Proveedor HTTP

| Aspecto | Nodos Locales | Proveedor HTTP |
|---------|---------------|----------------|
| **Configuración** | Crear archivo en la carpeta | Crear y ejecutar un servidor |
| **Lenguaje** | Solo JavaScript | Cualquier lenguaje |
| **Hot-Reload** | Automático | Automático |
| **Disponibilidad** | Solo versión desktop | Solo Enterprise |
| **Despliegue** | Solo local | Puede ser remoto |
| **Dependencias** | npm (local) | Cualquier gestor |

---

## Consejos

- Usa la convención `*.node.js` — QANode solo detecta archivos con esta extensión. Los archivos auxiliares **no** deben usar `.node.` en el nombre
- **Una carpeta por nodo** — cada nodo con su `.node.js`, auxiliares y `package.json`
- El campo `category` en el manifiesto controla en qué grupo aparece el nodo en la paleta
- Usa `console.log()` en execute para depuración — las salidas aparecen en la consola de Electron
- Comienza con el nodo de ejemplo (sample-hash) como plantilla
- Para operaciones largas, define `timeoutMs` en el manifiesto (predeterminado: 30 min, ej: `timeoutMs: 1800000` para 30 min)
