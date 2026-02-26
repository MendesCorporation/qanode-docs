# Nodos Locales en Version Desktop

En QANode Desktop puedes crear nodos personalizados de forma local, sin levantar un servidor HTTP separado. El runtime local detecta los archivos y publica los nodos en el proveedor local interno.

---

## Como Funciona (ciclo completo)

1. El runtime local escanea `%USERPROFILE%\Documents\QANode\custom nodes\`
2. Carga archivos `*.node.js`, `*.node.mjs`, `*.node.cjs`
3. Valida exports obligatorios (`manifest`, `execute`, `manifest.type`, `manifest.name`)
4. Publica nodos validos en `/manifest`
5. El worker consulta `/manifest` y valida `required` en `inputSchema`
6. El worker llama `/execute` con `nodeType`, `inputs`, `runId`, `nodeId`
7. El resultado vuelve al flujo con `logs`, `outputs` y `artifacts`

---

## Carpeta y Estructura Recomendada

```text
%USERPROFILE%\Documents\QANode\custom nodes\
```

```text
custom nodes/
├── mi-nodo/
│   ├── mi-nodo.node.js
│   ├── helpers.js
│   └── package.json
└── otro-nodo/
    ├── otro-nodo.node.mjs
    └── package.json
```

> Solo los archivos con `.node.` en el nombre se cargan como nodo.

---

## Reglas de Modulo por Extension

- `.node.js`: usa CommonJS (`module.exports`) por defecto
- `.node.mjs`: usa ESM (`export const`, `export async function`)
- `.node.cjs`: usa CommonJS

Error comun:

- `Unexpected token 'export'` en `.node.js`: el archivo esta en ESM sin configuracion de modulo correcta.

---

## Contrato del Archivo del Nodo

Cada archivo debe exportar:

1. `manifest`
2. `execute`

Si falta algun obligatorio, el archivo se ignora.

### Manifest: campos obligatorios y opcionales

| Campo | Tipo | Obligatorio | Uso en QANode |
|---|---|---|---|
| `type` | `string` | Si | Identificador unico del nodo |
| `name` | `string` | Si | Nombre visible en la paleta |
| `category` | `string` | No | Grupo en la paleta |
| `timeoutMs` | `number` | No | Timeout del nodo en ms; invalido/ausente usa valor por defecto |
| `inputSchema` | `object` | No | Campos de entrada en panel de propiedades |
| `outputSchema` | `object` | No | Campos de salida para autocompletado |
| `...extra` | `any` | No | Metadatos extra sin comportamiento por defecto |

### inputSchema: formato de campo

| Propiedad | Tipo | Obligatorio | Nota |
|---|---|---|---|
| `type` | `string` | Recomendado | `string`, `number`, `boolean`, `object`, `array`, `any` |
| `required` | `boolean` | No | Si es true, el worker bloquea ejecucion sin valor |
| `default` | `any` | No | Valor por defecto |
| `description` | `string` | No | Ayuda en UI |
| `enum` | `array` | No | Valores permitidos |
| `items` | `object` | No | Schema de item para arrays |

### outputSchema: formato

| Propiedad | Tipo | Obligatorio |
|---|---|---|
| `type` | `string` | Recomendado |

---

## Execute: parametros de entrada

La funcion `execute` recibe:

| Parametro | Tipo | Obligatorio | Origen |
|---|---|---|---|
| `nodeType` | `string` | No | Tipo solicitado |
| `inputs` | `object` | Si (practico) | Datos del nodo en el flujo |
| `runId` | `string` | No | ID de ejecucion |
| `nodeId` | `string` | No | ID del nodo en el flujo |

Payload de ejemplo:

```json
{
  "nodeType": "mi-nodo",
  "inputs": {
    "campo1": "valor",
    "campo2": 10
  },
  "runId": "run_123",
  "nodeId": "node_abc"
}
```

---

## Execute: retorno (obligatorio vs recomendado)

### Retorno completo recomendado

| Campo | Tipo | Obligatorio | Nota |
|---|---|---|---|
| `status` | `string` | Obligatorio | `success` o `failed` |
| `logs` | `string[]` | Recomendado | Logs de diagnostico |
| `outputs` | `object` | Recomendado | Datos de salida del nodo |
| `artifacts` | `array` | Recomendado | Archivos (screenshot/pdf/file/etc.) |
| `error` | `object` | Cuando failed | Ej: `{ "message": "..." }` |

### Comportamiento real del runtime local

- Si retorna objeto con `status`, se usa tal cual.
- Si retorna objeto sin `status`, el runtime puede envolver como `status: "success"` y usarlo como `outputs`.
- Si `execute` lanza error, runtime retorna `status: "failed"` con `error.message`.

---

## Ejemplo Correcto `.node.js` (CommonJS)

```javascript
// mi-nodo/mi-nodo.node.js
module.exports = {
  manifest: {
    type: "mi-nodo",
    name: "Mi Nodo",
    category: "Custom",
    timeoutMs: 600000,
    inputSchema: {
      texto: { type: "string", required: true, description: "Texto de entrada" },
      repetir: { type: "number", default: 1 }
    },
    outputSchema: {
      resultado: { type: "string" },
      processedAt: { type: "string" }
    }
  },
  async execute({ inputs = {}, runId, nodeId }) {
    const texto = String(inputs.texto || "");
    const repetir = Number(inputs.repetir || 1);
    const resultado = texto.repeat(Math.max(1, repetir));

    return {
      status: "success",
      logs: [
        `runId=${runId || "n/a"} nodeId=${nodeId || "n/a"}`,
        `texto=${texto}`,
        `repetir=${repetir}`
      ],
      outputs: {
        resultado,
        processedAt: new Date().toISOString()
      },
      artifacts: []
    };
  }
};
```

---

## Ejemplo Correcto `.node.mjs` (ESM)

```javascript
// mi-nodo-mjs/mi-nodo-mjs.node.mjs
export const manifest = {
  type: "mi-nodo-mjs",
  name: "Mi Nodo MJS",
  category: "Custom",
  inputSchema: {
    texto: { type: "string", required: true }
  },
  outputSchema: {
    resultado: { type: "string" }
  }
};

export async function execute({ inputs = {} }) {
  const resultado = String(inputs.texto || "").toUpperCase();
  return {
    status: "success",
    logs: [`resultado=${resultado}`],
    outputs: { resultado },
    artifacts: []
  };
}
```

---

## Artifacts (formato)

Cada item en `artifacts` puede ser:

```json
{
  "type": "screenshot",
  "name": "evidencia.png",
  "base64": "iVBORw0KGgoAAA..."
}
```

Campos comunes:

- `type`: `screenshot`, `pdf`, `video`, `file`
- `name`: nombre del archivo
- `base64`: contenido en base64
- `path`: opcional cuando ya existe ruta externa

---

## Como Verificar Errores de Importacion

### 1) Prueba por provider local

1. Abre **Configuraciones > Providers**
2. Prueba `Desktop Local JS Provider`
3. Si el nodo no aparece, no cargo

### 2) Log detallado del desktop

Archivo:

```text
%APPDATA%\@qanode\desktop\desktop-main.log
```

Busca:

- `Failed loading "...archivo...": ...`
- `Cannot find package '...'`
- `Unexpected token 'export'`

### 3) Verificacion por PowerShell

```powershell
$providers = Invoke-RestMethod -Uri "http://127.0.0.1:30000/api/providers"
$p = $providers | Where-Object { $_.name -eq "Desktop Local JS Provider" }
Invoke-RestMethod -Uri ($p.baseUrl + "/manifest")
Invoke-RestMethod -Method Post -Uri ($p.baseUrl + "/reload")
Invoke-RestMethod -Uri ($p.baseUrl + "/health")
```

### 4) Prueba de ejecucion directa

```powershell
Invoke-RestMethod -Method Post `
  -Uri ($p.baseUrl + "/execute") `
  -ContentType "application/json" `
  -Body '{"nodeType":"mi-nodo","inputs":{"texto":"ok"}}'
```

---

## Errores Mas Comunes

- `Cannot find package 'x' imported from ...`
  - Falta instalar dependencia en la carpeta del nodo

- `Unexpected token 'export'` en `.node.js`
  - Mezcla incorrecta CJS/ESM para la extension

- Nodo ausente en `/manifest`
  - Falta `manifest`
  - Falta `execute` o no es funcion
  - Falta/invalido `manifest.type`
  - Falta/invalido `manifest.name`

---

## Checklist Final de Calidad

- Extension correcta (`.node.js`, `.node.mjs`, `.node.cjs`)
- `manifest.type` unico
- `manifest.name` claro
- `inputSchema` y `outputSchema` definidos
- `execute` con manejo de errores
- Retorno con `status`, `logs`, `outputs`, `artifacts`
- Dependencias instaladas en carpeta del nodo
