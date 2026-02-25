# Creando un Proveedor de Nodos Personalizados (Enterprise)

Esta guía muestra cómo crear un proveedor HTTP de nodos personalizados para QANode Enterprise. Un proveedor es un servidor HTTP que implementa el contrato de API y puede estar escrito en cualquier lenguaje.

---

## Convenciones

### Nomenclatura de Archivos

Los archivos de definición de nodo deben usar la extensión **`.node.`** seguida de la extensión del lenguaje:

| Lenguaje | Extensión |
|----------|-----------|
| JavaScript | `.node.js`, `.node.mjs`, `.node.cjs` |
| Python | `.node.py` |
| Java | `.node.java` |
| C# | `.node.cs` |
| Go | `.node.go` |

Esta convención distingue los **archivos de nodo** (que exportan `manifest` y `execute`) de los **archivos auxiliares** (utils, helpers, etc.) que **no** deben usar `.node.` en el nombre.

### Organización: Una Carpeta por Nodo

Cada nodo personalizado debe tener su propia carpeta que contenga:

1. El archivo de definición del nodo (`.node.js` o equivalente)
2. Archivos auxiliares (helpers, utils — **sin** `.node.` en el nombre)
3. `package.json` o equivalente para las dependencias propias del nodo

```
meu-provedor/
├── server.js                    # Servidor HTTP (auto-loader)
├── package.json                 # Dependencias del servidor
├── validar-cpf/
│   ├── validar-cpf.node.js      # Definición del nodo
│   ├── cpf-utils.js             # Auxiliar (NO usa .node.)
│   └── package.json             # Dependencias del nodo
├── validar-cnpj/
│   ├── validar-cnpj.node.js     # Definición del nodo
│   ├── cnpj-utils.js            # Auxiliar
│   └── package.json
└── gerar-dados/
    ├── gerar-dados.node.js      # Definición del nodo
    ├── generators.js            # Auxiliar
    └── package.json
```

> **Importante:** Solo los archivos con `.node.` en el nombre son reconocidos como nodos. Otros archivos `.js` en la misma carpeta son ignorados por el loader y pueden usarse libremente como módulos auxiliares.

---

## Paso 1: Crea la Definición del Nodo

Cada archivo `.node.js` exporta `manifest` y `execute`:

```javascript
// validar-cpf/validar-cpf.node.js

import { validateCPF } from './cpf-utils.js';

export const manifest = {
  type: 'validar-cpf',
  name: 'Validar CPF',
  category: 'Validação',
  // timeoutMs: 300000, // opcional — predeterminado: 30 min
  inputSchema: {
    cpf: {
      type: 'string',
      required: true,
      description: 'CPF a ser validado (com ou sem formatação)'
    }
  },
  outputSchema: {
    valid: { type: 'boolean' },
    formatted: { type: 'string' },
    message: { type: 'string' }
  }
};

export async function execute({ inputs, runId, nodeId }) {
  const cpf = (inputs.cpf || '').replace(/\D/g, '');
  const valid = validateCPF(cpf);
  const formatted = cpf.replace(/(\d{3})(\d{3})(\d{3})(\d{2})/, '$1.$2.$3-$4');

  return {
    status: 'success',
    logs: [`CPF recebido: ${inputs.cpf}`, `Válido: ${valid}`],
    outputs: {
      valid,
      formatted,
      message: valid ? 'CPF válido' : 'CPF inválido'
    },
    artifacts: []
  };
}
```

Archivo auxiliar (sin `.node.` en el nombre):

```javascript
// validar-cpf/cpf-utils.js

export function validateCPF(cpf) {
  if (cpf.length !== 11 || /^(\d)\1+$/.test(cpf)) return false;
  let sum = 0, rest;
  for (let i = 1; i <= 9; i++) sum += parseInt(cpf[i - 1]) * (11 - i);
  rest = (sum * 10) % 11;
  if (rest === 10 || rest === 11) rest = 0;
  if (rest !== parseInt(cpf[9])) return false;
  sum = 0;
  for (let i = 1; i <= 10; i++) sum += parseInt(cpf[i - 1]) * (12 - i);
  rest = (sum * 10) % 11;
  if (rest === 10 || rest === 11) rest = 0;
  return rest === parseInt(cpf[10]);
}
```

---

## Paso 2: Crea el Servidor (Auto-Loader)

El servidor HTTP escanea automáticamente las carpetas de nodos, detectando archivos `.node.js`:

```javascript
// server.js
import express from 'express';
import fs from 'fs';
import path from 'path';
import { pathToFileURL } from 'url';

const app = express();
app.use(express.json());

const nodes = [];
const executors = {};

// Auto-loader: busca archivos *.node.js recursivamente
function findNodeFiles(dir) {
  const results = [];
  for (const entry of fs.readdirSync(dir, { withFileTypes: true })) {
    const fullPath = path.join(dir, entry.name);
    if (entry.isDirectory()) {
      results.push(...findNodeFiles(fullPath));
    } else if (entry.name.endsWith('.node.js') || entry.name.endsWith('.node.mjs')) {
      results.push(fullPath);
    }
  }
  return results;
}

// Carga todos los nodos
const nodeFiles = findNodeFiles(path.resolve('.'));
for (const filePath of nodeFiles) {
  try {
    const mod = await import(pathToFileURL(filePath).href);
    const manifest = mod.manifest || mod.default?.manifest;
    const execute = mod.execute || mod.default?.execute;
    if (manifest?.type && typeof execute === 'function') {
      nodes.push(manifest);
      executors[manifest.type] = execute;
      console.log(`Loaded: ${manifest.name} (${manifest.type}) from ${path.basename(filePath)}`);
    }
  } catch (err) {
    console.warn(`Failed to load ${filePath}: ${err.message}`);
  }
}

// GET /manifest — lista los nodos disponibles
app.get('/manifest', (req, res) => {
  res.json({ nodes });
});

// POST /execute — ejecuta un nodo
app.post('/execute', async (req, res) => {
  const { nodeType, inputs, runId, nodeId } = req.body;
  const executor = executors[nodeType];
  if (!executor) {
    return res.json({
      status: 'failed',
      logs: [],
      outputs: {},
      error: { message: `Nó não encontrado: ${nodeType}` }
    });
  }
  try {
    const result = await executor({ inputs, runId, nodeId });
    res.json(result);
  } catch (err) {
    res.json({
      status: 'failed',
      logs: [err.stack || err.message],
      outputs: {},
      error: { message: err.message }
    });
  }
});

// GET /health — verificación de salud (opcional)
app.get('/health', (req, res) => {
  res.json({ ok: true, nodeCount: nodes.length });
});

app.listen(4010, () => {
  console.log(`Provedor rodando em http://localhost:4010 (${nodes.length} nós carregados)`);
});
```

---

## Paso 3: Prueba Localmente

```bash
# Instala las dependencias de cada nodo
cd validar-cpf && npm install && cd ..
cd validar-cnpj && npm install && cd ..

# Inicia el servidor
node server.js

# Prueba el manifiesto
curl http://localhost:4010/manifest

# Prueba la ejecución
curl -X POST http://localhost:4010/execute \
  -H "Content-Type: application/json" \
  -d '{"nodeType": "validar-cpf", "inputs": {"cpf": "123.456.789-09"}}'
```

---

## Paso 4: Registra en QANode

1. En QANode, ve a **Configuraciones** → **Proveedores Personalizados**
2. Haz clic en **+ Agregar Proveedor**
3. Completa:
   - **Nombre**: `Validações`
   - **URL Base**: `http://localhost:4010`
   - **Token** (opcional): Token de autenticación Bearer
4. Haz clic en **Probar Conexión** → debe retornar éxito con el conteo de nodos
5. Guarda

---

## Paso 5: Usa en un Flujo

1. Abre el editor de flujos
2. En la paleta, localiza la categoría **Validação** (o **Custom Nodes**)
3. Arrastra el nodo **Validar CPF** al canvas
4. Configura el campo **CPF**: `{{ steps.extract.outputs.extracts.cpf }}`
5. Conéctalo al flujo y ejecútalo

---

## Ejemplo con Timeout Personalizado

Para nodos que ejecutan operaciones largas, define `timeoutMs` en el manifiesto:

```javascript
// processar-lote/processar-lote.node.js

export const manifest = {
  type: 'processar-lote',
  name: 'Processar Lote',
  category: 'Processamento',
  timeoutMs: 1800000, // 30 minutos
  inputSchema: {
    dados: { type: 'string', required: true, description: 'Caminho ou JSON dos dados' },
    formato: { type: 'string', enum: ['csv', 'json', 'xml'], default: 'json' }
  },
  outputSchema: {
    processados: { type: 'number' },
    erros: { type: 'number' },
    resultado: { type: 'array' }
  }
};

export async function execute({ inputs }) {
  // Operación larga...
  const resultado = await processarDados(inputs.dados, inputs.formato);

  return {
    status: 'success',
    logs: [`Processados: ${resultado.length} registros`],
    outputs: {
      processados: resultado.length,
      erros: 0,
      resultado
    },
    artifacts: []
  };
}
```

Si `timeoutMs` no está definido, el valor predeterminado es **30 minutos** (300.000ms).

---

## Ejemplo en Otros Lenguajes

La convención `.node.` se aplica en cualquier lenguaje. Lo importante es que el auto-loader de tu servidor sepa identificar los archivos de nodo:

### Python

```
meu-provedor-python/
├── server.py                    # Servidor FastAPI
├── requirements.txt
├── validar-cpf/
│   ├── validar_cpf.node.py      # Definición del nodo
│   ├── utils.py                 # Auxiliar (sin .node.)
│   └── requirements.txt
└── gerar-dados/
    ├── gerar_dados.node.py
    └── requirements.txt
```

```python
# validar-cpf/validar_cpf.node.py

from .utils import validate_cpf_digits

manifest = {
    "type": "validar-cpf",
    "name": "Validar CPF",
    "category": "Validação",
    "inputSchema": {
        "cpf": {"type": "string", "required": True}
    },
    "outputSchema": {
        "valid": {"type": "boolean"}
    }
}

def execute(inputs, run_id=None, node_id=None):
    cpf = inputs.get("cpf", "").replace(".", "").replace("-", "")
    valid = validate_cpf_digits(cpf)
    return {
        "status": "success",
        "logs": [f"CPF: {cpf}, Válido: {valid}"],
        "outputs": {"valid": valid},
        "artifacts": []
    }
```

### Java / C# / Go

Sigue la misma estructura: cada nodo en su carpeta con el archivo `.node.{ext}` y los auxiliares sin `.node.` en el nombre. El servidor HTTP del lenguaje elegido implementa el auto-loader que busca los archivos `.node.*`.

---

## Buenas Prácticas

1. **Una carpeta por nodo** — mantiene las dependencias y auxiliares aislados
2. **Nomenclatura `.node.`** — solo para archivos de definición de nodo, nunca para auxiliares
3. **`package.json` por nodo** — cada nodo gestiona sus propias dependencias
4. **Logs descriptivos** — incluye logs que ayuden a diagnosticar problemas
5. **`timeoutMs` en el manifiesto** — defínelo para nodos que superen los 30 minutos
6. **Output Schema** — define `outputSchema` para que QANode muestre los campos en el autocompletado
7. **Docker** — considera containerizar el proveedor para facilitar el despliegue

---

## Próximos Pasos

- [Contrato de la API](contrato-api.md) — Especificación completa de los endpoints
- [Ejemplos](ejemplos-enterprise.md) — Más ejemplos en varios lenguajes
- [Nodos Locales Desktop](nodos-locales-escritorio.md) — Alternativa sin servidor HTTP
