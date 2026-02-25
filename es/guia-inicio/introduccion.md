# Introducción a QANode

**QANode** es una plataforma de automatización de pruebas que permite crear, ejecutar y gestionar pruebas automatizadas a través de un **editor visual de flujos** basado en nodos. Con una interfaz intuitiva de arrastrar y conectar, puedes construir escenarios de prueba complejos sin escribir código — QANode, a través de su extensibilidad con nodos personalizados, está preparado para ejecutar cualquier tipo de prueba.

---

## ¿Qué es QANode?

QANode fue diseñado para equipos de QA, desarrolladores e ingenieros de pruebas que necesitan una herramienta moderna y visual para la automatización de pruebas. La plataforma cubre las principales áreas de prueba:

### Automatización Web
Prueba interfaces de usuario usando **Playwright**, el framework de automatización más moderno del mercado. QANode ofrece dos nodos especializados:

- **Web Flow** — Automatización basada en selectores CSS, XPath, data-testid y texto
- **Smart Locators** — Automatización con localizadores semánticos (`getByRole`, `getByLabel`, `getByPlaceholder`, etc.)

Ambos soportan Chromium, modo headless, captura de capturas de pantalla y reutilización de sesión.

### Pruebas de API
Ejecuta solicitudes HTTP con soporte completo para:
- Métodos GET, POST, PUT, PATCH y DELETE
- Autenticación Bearer, Basic y API Key
- Headers personalizados y cuerpo de la solicitud
- Integración con credenciales guardadas

### Base de Datos
Conéctate y valida datos en:
- **PostgreSQL**, **MySQL**, **MariaDB**, **Oracle** — con query builder visual o SQL directo
- **MongoDB** — con operaciones find, insert, update, delete y aggregation pipeline

### Infraestructura
Ejecuta comandos remotos vía **SSH** con soporte para múltiples pasos, autenticación por contraseña o clave privada, y captura de salida.

### Extensibilidad
Crea **nodos personalizados** en cualquier lenguaje de programación (Node.js, Python, Java, C#, Go, etc.) a través del sistema de proveedores HTTP. (La Community Edition solo soporta JavaScript)

---

## Características Principales

| Recurso | Descripción |
|---------|-------------|
| **Editor Visual** | Canvas interactivo con arrastrar y conectar nodos |
| **18+ Nodos Nativos** | Control de flujo, web, API, base de datos, infraestructura y utilidades |
| **Nodos Personalizados** | Extensible con proveedores HTTP en cualquier lenguaje |
| **Proyectos y Suites** | Organiza pruebas en proyectos y agrúpalas en suites |
| **Programación** | Ejecuta pruebas automáticamente con expresiones cron |
| **Dashboard** | Paneles personalizables con gráficos y métricas en tiempo real |
| **Reportes PDF** | Generación automática con plantillas configurables |
| **Variables y Credenciales** | Gestión centralizada y segura |
| **Control de Acceso** | Usuarios, roles y permisos granulares |
| **Auditoría** | Registro completo de todas las acciones del sistema |
| **Grabador Chrome** | Extensión que graba interacciones y genera flujos |
| **Versión Desktop** | Aplicación standalone con base de datos integrada |
| **Expresiones** | Sistema de interpolación `{{ }}` con acceso a outputs y variables |
| **Tiempo Real** | Actualizaciones en vivo del estado de ejecución |

---

## Arquitectura

QANode está compuesto por cuatro componentes principales:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Frontend    │    │  API Server  │    │   Worker     │
│  (React)     │◄──►│  (Fastify)   │◄──►│  (Executor)  │
│  Vite + TS   │    │  REST API    │    │  Playwright  │
└─────────────┘    └──────┬───────┘    └─────────────┘
                          │
                   ┌──────▼───────┐
                   │  PostgreSQL   │
                   │  (Prisma ORM) │
                   └──────────────┘
```

- **Frontend** — Interfaz de usuario en React con TypeScript, construida con Vite
- **API Server** — Servidor REST en Fastify con autenticación JWT
- **Worker** — Motor de ejecución que procesa los flujos y controla Playwright
- **PostgreSQL** — Base de datos relacional para almacenamiento de datos (vía Prisma ORM)

### Versión Desktop

La versión desktop empaqueta todos los componentes en una única aplicación **Electron**, incluyendo un PostgreSQL integrado. Solo instala y usa — sin necesidad de configuración de infraestructura.

---

## Community Edition vs Enterprise

Esta documentación cubre la **Community Edition**, que es gratuita. La siguiente tabla resume las diferencias:

| Recurso | Community | Enterprise |
|---------|-----------|------------|
| Editor de Flujos | ✅ | ✅ |
| Todos los Nodos Nativos | ✅ | ✅ |
| Nodos Personalizados | ✅ | ✅ |
| Proyectos y Suites | ✅ | ✅ |
| Programación | ✅ | ✅ |
| Variables y Credenciales | ✅ | ✅ |
| Reportes PDF | ✅ | ✅ |
| Grabador de pasos Chrome | ✅ | ✅ |
| Versión Web | ❌ | ✅ |
| Dashboard Personalizable | ❌ | ✅  |
| Multi-usuario | ❌ | ✅  |
| MFA (Autenticación 2FA) | ❌ | ✅ |
| Logs de auditoría | ❌ | ✅ |
| Generador de reportes | ❌ | ✅ |
| Programación de alarmas | ❌ | ✅ |
| Webhooks | ❌ | ✅ |
| Soporte Dedicado | ❌ | ✅ |

---

## Próximos Pasos

- [Instalación](instalacion.md) — Configura QANode en tu entorno
- [Inicio Rápido](inicio-rapido.md) — Crea tu primera prueba en minutos
- [Conceptos Fundamentales](conceptos.md) — Comprende la estructura de la plataforma
