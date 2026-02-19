# Nós Locais na Versão Desktop

Na versão desktop do QANode, você pode criar nós customizados **localmente** como arquivos JavaScript, sem precisar de um servidor HTTP separado. O QANode detecta automaticamente os arquivos e disponibiliza os nós na paleta.

---

## Como Funciona

A versão desktop inclui um **provedor local embutido** que:

1. Monitora uma pasta no seu computador
2. Detecta arquivos `.node.js`, `.node.mjs` ou `.node.cjs`
3. Carrega automaticamente os nós definidos nesses arquivos
4. Recarrega quando detecta alterações (**hot-reload**)

---

## Pasta de Nós Customizados

Os nós locais ficam na pasta:

```
%USERPROFILE%\Documents\QANode\custom nodes\
```

No Windows, geralmente:
```
C:\Users\SeuUsuario\Documents\QANode\custom nodes\
```

A organização recomendada é **uma pasta por nó**, cada uma contendo o arquivo `.node.js`, arquivos auxiliares e `package.json` para dependências:

```
custom nodes/
├── validar-cpf/
│   ├── validar-cpf.node.js    # Definição do nó
│   ├── cpf-utils.js           # Auxiliar (NÃO usa .node.)
│   └── package.json           # Dependências do nó
├── formatar-data/
│   ├── formatar-data.node.js
│   └── package.json
└── sample-hash-node/          # Nó de exemplo (criado automaticamente)
    ├── sample-hash.node.js
    └── package.json
```

> **Importante:** Somente arquivos com `.node.` no nome são reconhecidos como nós. Outros arquivos `.js` na mesma pasta são ignorados e podem ser usados como módulos auxiliares.

---

## Formato do Arquivo

Cada arquivo `.node.js` deve exportar dois elementos:

1. **`manifest`** — Definição do nó (tipo, nome, inputs, outputs)
2. **`execute`** — Função de execução

### Estrutura Básica

```javascript
// meu-no/meu-no.node.js

export const manifest = {
  type: 'meu-no-customizado',
  name: 'Meu Nó Customizado',
  category: 'Meus Nós',
  timeoutMs: 600000, // 10 minutos (opcional, padrão: 30 minutos)
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

## Exemplo Completo: Hash Generator

Este é o nó de exemplo que o QANode cria automaticamente na primeira execução:

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

O QANode monitora a pasta de nós customizados e **recarrega automaticamente** quando detecta alterações:

- **Intervalo de verificação**: 1.5 segundos
- **Detecção**: Baseada em data de modificação e tamanho do arquivo
- **Recarga**: Automática, sem necessidade de reiniciar o QANode

### Fluxo de Desenvolvimento

1. Crie ou edite um arquivo `.node.js` na pasta de nós
2. Salve o arquivo
3. Em ~1.5 segundos, o QANode detecta a mudança
4. O nó atualizado aparece na paleta do editor

> **Dica:** Abra o console do QANode (DevTools) para ver logs de recarga:
> ```
> [CustomProvider] Reloaded: meu-no.node.js (2 nodes)
> ```

---

## Usando Dependências NPM

Nós locais podem usar módulos Node.js nativos (`crypto`, `path`, `fs`, etc.) diretamente. Para dependências npm externas, cada nó tem seu próprio `package.json`:

```
custom nodes/
└── formatar-data/
    ├── formatar-data.node.js   # Definição do nó
    ├── date-helpers.js         # Auxiliar (sem .node.)
    └── package.json            # Dependências do nó
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

## Diferenças: Nós Locais vs Provedor HTTP

| Aspecto | Nós Locais | Provedor HTTP |
|---------|------------|---------------|
| **Setup** | Criar arquivo na pasta | Criar e rodar servidor |
| **Linguagem** | Apenas JavaScript | Qualquer linguagem |
| **Hot-Reload** | Automático | Automático |
| **Disponibilidade** | Apenas versão desktop | Apenas Enterprise |
| **Deploy** | Local apenas | Pode ser remoto |
| **Dependências** | npm (local) | Qualquer gerenciador |

---

## Dicas

- Use a convenção `*.node.js` — o QANode só detecta arquivos com essa extensão. Arquivos auxiliares **não** devem usar `.node.` no nome
- **Uma pasta por nó** — cada nó com seu `.node.js`, auxiliares e `package.json`
- O campo `category` no manifesto controla em qual grupo o nó aparece na paleta
- Use `console.log()` no execute para depuração — as saídas aparecem no console do Electron
- Comece com o nó de exemplo (sample-hash) como template
- Para operações longas, defina `timeoutMs` no manifesto (padrão: 5 min, ex: `timeoutMs: 1800000` para 30 min)
