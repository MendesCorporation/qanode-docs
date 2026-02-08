# Versão Desktop

A versão desktop do QANode é um aplicativo **Community Edition** que não tem restrição no seu core principal de fluxos.

---

## Vantagens da Versão Desktop

| Característica | Descrição |
|---------------|-----------|
| **Instalação Simples** | Um único instalador, sem dependências externas |
| **Banco Embutido** | PostgreSQL integrado, sem necessidade de configuração |
| **Nós Locais** | Crie nós customizados como arquivos locais (hot-reload) |
| **Tudo Integrado** | Frontend, API, Worker e banco em um único app |
| **Portável** | Funciona offline, sem servidor |

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

1. Exibe uma **tela de splash** com progresso
2. Inicializa o **PostgreSQL embutido**
3. Executa as **migrations** (cria tabelas)
4. Inicia o **servidor da API**
5. Abre a **interface principal**

<!-- ![Splash screen](../../assets/images/splash-screen.png) -->
*Imagem: Tela de splash mostrando progresso da inicialização com barra de status*

> A primeira inicialização pode levar alguns segundos adicionais enquanto o banco é preparado.

---

## Estrutura de Dados

O QANode armazena dados na pasta do usuário:

```
%APPDATA%\QANode\
├── pg-data/              # Dados do PostgreSQL embutido
├── storage/              # Artefatos (screenshots, PDFs)
└── logs/                 # Logs da aplicação
```

```
%USERPROFILE%\Documents\QANode\
└── custom nodes/         # Nós customizados locais
    └── sample-hash-node/ # Nó de exemplo
```

> **Importante:** Não modifique manualmente os arquivos em `pg-data/`.

---

## Nós Customizados Locais

A versão desktop inclui suporte a **nós locais** — arquivos JavaScript que são detectados e carregados automaticamente.

### Pasta dos Nós

```
%USERPROFILE%\Documents\QANode\custom nodes\
```

### Criando um Nó Local

1. Crie um arquivo com extensão `.node.js` na pasta acima
2. Exporte `manifest` e `execute`
3. O QANode detecta automaticamente em ~1.5 segundos

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

> Para detalhes completos, veja [Nós Locais Desktop](../nos-customizados/nos-locais-desktop.md).

---

## Hot-Reload

O provedor local monitora a pasta de nós customizados e recarrega automaticamente quando detecta alterações:

- **Intervalo**: 1.5 segundos
- **Detecção**: Data de modificação + tamanho do arquivo
- **Recarga**: Automática, sem reiniciar o app

Ideal para desenvolvimento rápido de nós customizados.

---

## Tema Nativo

O QANode desktop suporta temas claro e escuro que se integram com a barra de título nativa do Windows:

- **Tema Claro** — Interface clara com barra de título branca
- **Tema Escuro** — Interface escura com barra de título escura

A troca de tema pode ser feita pela interface e sincroniza com a barra de título do sistema operacional.

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
- Verifique se a porta 5432 não está em uso (PostgreSQL externo pode conflitar)

### Banco de dados não inicializa

- A pasta `%APPDATA%\QANode\pg-data` pode estar corrompida
- Tente renomear/remover a pasta e reiniciar (os dados serão perdidos)

### Nós locais não aparecem

- Verifique se o arquivo tem extensão `.node.js`, `.node.mjs` ou `.node.cjs`
- Verifique se exporta `manifest` e `execute`
- Abra o DevTools (Ctrl+Shift+I) para ver logs de erro

---

## Dicas

- Use a versão desktop para **desenvolvimento e testes individuais**
- Aproveite os **nós locais** para prototipagem rápida
- O **hot-reload** elimina a necessidade de reiniciar durante o desenvolvimento
