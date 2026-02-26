# Versão Desktop

A versão desktop do QANode é um aplicativo **Community Edition** sem restrição no core principal de fluxos.

---

## Vantagens da Versão Desktop

| Característica | Descrição |
|---------------|-----------|
| **Instalação Simples** | Um único instalador, sem dependências externas |
| **Banco Embutido** | PostgreSQL integrado, sem necessidade de configuração |
| **Nós Locais** | Crie nós customizados como arquivos locais (hot-reload) |
| **Tudo Integrado** | Frontend, API, Worker e banco em um único app |
| **Portátil** | Funciona offline, sem servidor |

---

## Instalação

Consulte o guia de [Instalação](../guia-inicio/instalacao.md) para instruções detalhadas.

### Requisitos Mínimos

- **Windows** 10/11 (64-bit)
- **RAM**: 4 GB mínimo (8 GB recomendado)
- **Disco**: 500 MB para instalação + espaço para dados

---

## Primeira Execução

Na primeira execução, o QANode:

1. Exibe uma tela de splash com progresso
2. Inicializa o PostgreSQL embutido
3. Executa as migrations (cria tabelas)
4. Inicia o servidor da API
5. Abre a interface principal

> A primeira inicialização pode levar alguns segundos adicionais enquanto o banco é preparado.

---

## Estrutura de Dados

O QANode armazena dados na pasta do usuário:

```text
%APPDATA%\QANode\
├── pg-data/              # Dados do PostgreSQL embutido
├── storage/              # Artefatos (screenshots, PDFs)
└── logs/                 # Logs da aplicação
```

```text
%USERPROFILE%\Documents\QANode\
└── custom nodes/         # Nós customizados locais
    └── sample-hash-node/ # Nó de exemplo
```

> Importante: não modifique manualmente os arquivos em `pg-data/`.

---

## Nós Customizados Locais

A versão desktop inclui suporte a nós locais: arquivos JavaScript detectados e carregados automaticamente.

### Pasta dos Nós

```text
%USERPROFILE%\Documents\QANode\custom nodes\
```

### Criando um Nó Local

1. Crie um arquivo com extensão `.node.js`, `.node.mjs` ou `.node.cjs`
2. Exporte `manifest` e `execute`
3. O QANode detecta automaticamente em ~1.5 segundos

### Exemplo: `.node.js` (CommonJS)

```javascript
// meu-no.node.js
module.exports = {
  manifest: {
    type: 'meu-no',
    name: 'Meu Nó',
    category: 'Meus Nós',
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
      logs: [`Processado: ${inputs.texto}`],
      artifacts: [],
    };
  },
};
```

### Exemplo: `.node.mjs` (ESM)

```javascript
// meu-no.node.mjs
export const manifest = {
  type: 'meu-no-mjs',
  name: 'Meu Nó MJS',
  category: 'Meus Nós',
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
    logs: [`Processado: ${inputs.texto}`],
    artifacts: [],
  };
}
```

> Para detalhes completos, veja [Nós Locais Desktop](../nos-customizados/nos-locais-desktop.md).

---

## Hot-Reload

O provedor local monitora a pasta de nós customizados e recarrega automaticamente quando detecta alterações:

- **Intervalo**: 1.5 segundos
- **Detecção**: data de modificação + tamanho do arquivo
- **Recarga**: automática, sem reiniciar o app

Ideal para desenvolvimento rápido de nós customizados.

---

## Tema Nativo

O QANode desktop suporta temas claro e escuro que se integram com a barra de título nativa do Windows:

- **Tema Claro**: interface clara com barra de título branca
- **Tema Escuro**: interface escura com barra de título escura

---

## Diferenças em Relação à Versão Web

| Aspecto | Desktop | Web (Enterprise) |
|---------|---------|-----|
| **Banco de Dados** | PostgreSQL embutido | PostgreSQL externo |
| **Instalação** | Instalador único | Manual (Node.js + PostgreSQL) |
| **Nós Locais** | Suportados (hot-reload) | Apenas provedores HTTP |
| **Multi-usuário** | Uso individual | Equipe compartilhada |
| **Rede** | Funciona offline | Requer rede |
| **Atualizações** | Instalador manual | Git pull + rebuild |

---

## Solução de Problemas

### O app não inicia

- Verifique se outra instância não está rodando
- Tente executar como administrador
- Verifique conflito de portas

### Banco de dados não inicializa

- A pasta `%APPDATA%\QANode\pg-data` pode estar corrompida
- Renomeie/remova a pasta e reinicie (os dados serão perdidos)

### Nós locais não aparecem

- Verifique extensão: `.node.js`, `.node.mjs` ou `.node.cjs`
- Verifique exportação de `manifest` e `execute`
- Para `.node.js`, use CommonJS (`module.exports`) ou configure `type: "module"` no `package.json`
- Abra `%APPDATA%\@qanode\desktop\desktop-main.log` e procure por `Failed loading`

### Como verificar erro de importação rapidamente

1. Abra **Configurações > Providers** e teste o provider `Desktop Local JS Provider`
2. Veja o log em `%APPDATA%\@qanode\desktop\desktop-main.log`
3. Procure mensagens como:
- `Failed loading ".../seu-node.node.mjs": Cannot find package 'x'`
- `Unexpected token 'export'` (geralmente `.node.js` em ESM sem `type: "module"`)

---

## Dicas

- Use `.node.js` com CommonJS para evitar conflito de módulo
- Use `.node.mjs` quando quiser ESM explícito
- Mantenha uma pasta por nó com `package.json` próprio
- Use o hot-reload para iterar rápido
