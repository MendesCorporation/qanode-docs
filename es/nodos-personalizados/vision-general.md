# Nodos Personalizados — Visión General

QANode permite extender la plataforma con **nodos personalizados** creados en cualquier lenguaje de programación. A través del sistema de **proveedores HTTP**, puedes crear nodos que ejecuten cualquier lógica — desde integraciones con herramientas internas hasta automatizaciones específicas de tu negocio.

---

## Cómo Funciona

El sistema de nodos personalizados se basa en un contrato HTTP simple:

```
┌────────────────┐         ┌────────────────────┐
│   QANode       │  HTTP   │  Tu Proveedor      │
│   (Worker)     │ ◄─────► │  (Cualquier Lenguaje)│
│                │         │                    │
│ 1. GET /manifest        │  ← Define los nodos │
│ 2. POST /execute        │  ← Ejecuta un nodo  │
└────────────────┘         └────────────────────┘
```

1. QANode llama a `GET /manifest` en tu proveedor para **descubrir** qué nodos están disponibles
2. Los nodos aparecen en la **paleta** del editor de flujos con ícono rosa
3. Cuando se ejecuta un flujo, QANode llama a `POST /execute` en el proveedor para **ejecutar** cada nodo

---

## Formas de Crear Nodos Personalizados

### 1. Proveedor HTTP (Cualquier Lenguaje) - Enterprise

Crea un servidor HTTP que implemente el contrato de API. Puede estar escrito en:

- **Node.js** (Express, Fastify, Koa)
- **Python** (Flask, FastAPI, Django)
- **Java** (Spring Boot)
- **C#** (.NET)
- **Go** (net/http, Gin)
- **Cualquier lenguaje** con soporte HTTP

> Consulta [Creando un Proveedor](creando-proveedor-enterprise.md) para la guía paso a paso.

### 2. Nodos Locales en Version Desktop - Community Edition

En desktop puedes crear archivos `.node.js`, `.node.mjs` o `.node.cjs` en la carpeta de nodos personalizados. QANode los detecta automaticamente y expone los nodos sin necesidad de un servidor HTTP separado.

> Consulta [Nodos Locales Desktop](nodos-locales-escritorio.md) para más detalles.

---

## Conceptos

### Proveedor

Un **proveedor** es un servidor HTTP registrado en QANode que proporciona nodos personalizados. Cada proveedor puede ofrecer **múltiples nodos**.

### Manifiesto

El **manifiesto** es la respuesta de la ruta `GET /manifest`, que describe todos los nodos disponibles en el proveedor, incluyendo:
- Nombre y tipo de cada nodo
- Categoría para organización en la paleta
- Esquema de entrada (campos de configuración)
- Esquema de salida (datos producidos)

### Ejecución

La **ejecución** se realiza mediante `POST /execute`, donde QANode envía los datos de entrada del nodo y el proveedor retorna el resultado. Cada nodo tiene un timeout predeterminado de **30 minutos**, configurable mediante `timeoutMs` en el manifiesto para operaciones más largas.

---

## Registrando un Proveedor

1. Ve a **Configuraciones** → **Proveedores Personalizados**
2. Haz clic en **+ Agregar Proveedor**
3. Completa:
   - **Nombre**: Nombre descriptivo del proveedor
   - **URL Base**: URL del servidor (ej: `http://localhost:4010`)
   - **Token** (opcional): Token de autenticación Bearer
4. Haz clic en **Probar Conexión** para verificar
5. Guarda el proveedor

<!-- ![Registrando un proveedor](../../assets/images/registrar-provedor.png) -->
*Imagen: Pantalla de configuraciones mostrando la lista de proveedores con el botón de agregar*

Tras el registro, los nodos del proveedor aparecerán automáticamente en la paleta del editor de flujos bajo la categoría **Nodos Personalizados** (o la categoría definida en el manifiesto).

---

## Flujo Completo

```
1. Creas un servidor HTTP (proveedor)
2. Implementas GET /manifest y POST /execute
3. Registras el proveedor en QANode
4. Pruebas la conexión
5. Abres el editor de flujos → los nodos aparecen en la paleta
6. Arrastras nodos personalizados al canvas
7. Configuras los campos (definidos en inputSchema)
8. Ejecutas el flujo → QANode llama a POST /execute en el proveedor
9. El resultado retorna y está disponible mediante expresiones {{ }}
```

---

## Próximos Pasos

- [Creando un Proveedor](creando-proveedor-enterprise.md) — Guía paso a paso
- [Contrato de la API](contrato-api.md) — Especificación completa de los endpoints
- [Ejemplos](ejemplos-enterprise.md) — Código de ejemplo en varios lenguajes
- [Nodos Locales Desktop](nodos-locales-escritorio.md) — Nodos locales en la versión desktop
