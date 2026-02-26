# Custom Nodes — Overview

QANode allows you to extend the platform with **custom nodes** created in any programming language. Through the **HTTP provider** system, you can create nodes that execute any logic — from integrations with internal tools to automations specific to your business.

---

## How It Works

The custom node system is based on a simple HTTP contract:

```
┌────────────────┐         ┌────────────────────┐
│   QANode       │  HTTP   │  Your Provider     │
│   (Worker)     │ ◄─────► │  (Any Language)    │
│                │         │                    │
│ 1. GET /manifest        │  ← Defines nodes    │
│ 2. POST /execute        │  ← Executes a node  │
└────────────────┘         └────────────────────┘
```

1. QANode calls `GET /manifest` on your provider to **discover** which nodes are available
2. The nodes appear in the flow editor **palette** with a pink icon
3. When a flow is executed, QANode calls `POST /execute` on the provider to **execute** each node

---

## Ways to Create Custom Nodes

### 1. HTTP Provider (Any Language) - Enterprise

Create an HTTP server that implements the API contract. It can be written in:

- **Node.js** (Express, Fastify, Koa)
- **Python** (Flask, FastAPI, Django)
- **Java** (Spring Boot)
- **C#** (.NET)
- **Go** (net/http, Gin)
- **Any language** with HTTP support

> See [Creating a Provider](creating-enterprise-provider.md) for the step-by-step guide.

### 2. Local Nodes in Desktop Version - Community Edition

In desktop, you can create `.node.js`, `.node.mjs`, or `.node.cjs` files in the custom nodes folder. QANode automatically detects and exposes these nodes without a separate HTTP server.

> See [Local Desktop Nodes](local-desktop-nodes.md) for details.

---

## Concepts

### Provider

A **provider** is an HTTP server registered in QANode that supplies custom nodes. Each provider can offer **multiple nodes**.

### Manifest

The **manifest** is the response from the `GET /manifest` route, which describes all available nodes in the provider, including:
- Name and type of each node
- Category for organization in the palette
- Input schema (configuration fields)
- Output schema (data produced)

### Execution

**Execution** is done via `POST /execute`, where QANode sends the node's input data and the provider returns the result. Each node has a default timeout of **30 minutes**, configurable via `timeoutMs` in the manifest for longer operations.

---

## Registering a Provider

1. Go to **Settings** → **Custom Providers**
2. Click **+ Add Provider**
3. Fill in:
   - **Name**: Descriptive name for the provider
   - **Base URL**: Server URL (e.g., `http://localhost:4010`)
   - **Token** (optional): Bearer authentication token
4. Click **Test Connection** to verify
5. Save the provider

<!-- ![Registering a provider](../../assets/images/registrar-provedor.png) -->
*Image: Settings screen showing the provider list with an add button*

After registration, the provider's nodes will automatically appear in the flow editor palette under the **Custom Nodes** category (or the category defined in the manifest).

---

## Complete Flow

```
1. You create an HTTP server (provider)
2. Implement GET /manifest and POST /execute
3. Register the provider in QANode
4. Test the connection
5. Open the flow editor → nodes appear in the palette
6. Drag custom nodes onto the canvas
7. Configure the fields (defined in inputSchema)
8. Execute the flow → QANode calls POST /execute on the provider
9. Result is returned and available via expressions {{ }}
```

---

## Next Steps

- [Creating a Provider](creating-enterprise-provider.md) — Step-by-step guide
- [API Contract](api-contract.md) — Complete endpoint specification
- [Examples](enterprise-examples.md) — Sample code in various languages
- [Local Desktop Nodes](local-desktop-nodes.md) — Local nodes in the desktop version
