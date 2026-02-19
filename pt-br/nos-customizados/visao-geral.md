# Nós Customizados — Visão Geral

O QANode permite estender a plataforma com **nós customizados** criados em qualquer linguagem de programação. Através do sistema de **provedores HTTP**, você pode criar nós que executam qualquer lógica — desde integrações com ferramentas internas até automações específicas do seu negócio.

---

## Como Funciona

O sistema de nós customizados é baseado em um contrato HTTP simples:

```
┌────────────────┐         ┌────────────────────┐
│   QANode       │  HTTP   │  Seu Provedor      │
│   (Worker)     │ ◄─────► │  (Qualquer Linguagem)│
│                │         │                    │
│ 1. GET /manifest        │  ← Define os nós    │
│ 2. POST /execute        │  ← Executa um nó    │
└────────────────┘         └────────────────────┘
```

1. O QANode chama `GET /manifest` no seu provedor para **descobrir** quais nós estão disponíveis
2. Os nós aparecem na **paleta** do editor de fluxos com ícone rosa
3. Quando um fluxo é executado, o QANode chama `POST /execute` no provedor para **executar** cada nó

---

## Formas de Criar Nós Customizados

### 1. Provedor HTTP (Qualquer Linguagem) - Enterprise

Crie um servidor HTTP que implementa o contrato de API. Pode ser escrito em:

- **Node.js** (Express, Fastify, Koa)
- **Python** (Flask, FastAPI, Django)
- **Java** (Spring Boot)
- **C#** (.NET)
- **Go** (net/http, Gin)
- **Qualquer linguagem** com suporte a HTTP

> Veja [Criando um Provedor](criando-provedor-enterprise.md) para o guia passo a passo.

### 2. Nós Locais na Versão Desktop - Community Edition

Na versão desktop, você pode criar arquivos `.node.js` na pasta de nós customizados. O QANode detecta automaticamente e disponibiliza os nós sem necessidade de servidor HTTP separado.

> Veja [Nós Locais Desktop](nos-locais-desktop.md) para detalhes.

---

## Conceitos

### Provedor

Um **provedor** é um servidor HTTP registrado no QANode que fornece nós customizados. Cada provedor pode oferecer **múltiplos nós**.

### Manifesto

O **manifesto** é a resposta da rota `GET /manifest`, que descreve todos os nós disponíveis no provedor, incluindo:
- Nome e tipo de cada nó
- Categoria para organização na paleta
- Schema de entrada (campos de configuração)
- Schema de saída (dados produzidos)

### Execução

A **execução** é feita via `POST /execute`, onde o QANode envia os dados de entrada do nó e o provedor retorna o resultado. Cada nó tem um timeout padrão de **30 minutos**, configurável via `timeoutMs` no manifesto para operações mais longas.

---

## Registrando um Provedor

1. Acesse **Configurações** → **Provedores Customizados**
2. Clique em **+ Adicionar Provedor**
3. Preencha:
   - **Nome**: Nome descritivo do provedor
   - **URL Base**: URL do servidor (ex: `http://localhost:4010`)
   - **Token** (opcional): Token de autenticação Bearer
4. Clique em **Testar Conexão** para verificar
5. Salve o provedor

<!-- ![Registrando um provedor](../../assets/images/registrar-provedor.png) -->
*Imagem: Tela de configurações mostrando a lista de provedores com botão de adicionar*

Após o registro, os nós do provedor aparecerão automaticamente na paleta do editor de fluxos na categoria **Nós Customizados** (ou na categoria definida no manifesto).

---

## Fluxo Completo

```
1. Você cria um servidor HTTP (provedor)
2. Implementa GET /manifest e POST /execute
3. Registra o provedor no QANode
4. Testa a conexão
5. Abre o editor de fluxos → nós aparecem na paleta
6. Arrasta nós customizados para o canvas
7. Configura os campos (definidos no inputSchema)
8. Executa o fluxo → QANode chama POST /execute no provedor
9. Resultado retorna e fica disponível via expressões {{ }}
```

---

## Próximos Passos

- [Criando um Provedor](criando-provedor-enterprise.md) — Guia passo a passo
- [Contrato da API](contrato-api.md) — Especificação completa dos endpoints
- [Exemplos](exemplos-enterprise.md) — Código de exemplo em várias linguagens
- [Nós Locais Desktop](nos-locais-desktop.md) — Nós locais na versão desktop
