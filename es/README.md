# QANode — Documentación

Bienvenido a la documentación oficial de **QANode Community Edition**, la plataforma de automatización de pruebas con editor visual de flujos.

---

## Primeros Pasos

| Página | Descripción |
|--------|-------------|
| [Introducción](guia-inicio/introduccion.md) | Qué es QANode y descripción general de la plataforma |
| [Instalación](guia-inicio/instalacion.md) | Cómo instalar y configurar QANode |
| [Inicio Rápido](guia-inicio/inicio-rapido.md) | Crea tu primera prueba en minutos |
| [Conceptos Fundamentales](guia-inicio/conceptos.md) | Proyectos, flujos, nodos, suites y ejecuciones |

---

## Editor de Flujos

| Página | Descripción |
|--------|-------------|
| [Descripción General del Editor](editor-flujos/vision-general.md) | Interfaz, canvas, paleta de nodos y panel de propiedades |
| [Trabajando con Nodos](editor-flujos/trabajando-con-nodos.md) | Agregar, conectar, configurar y eliminar nodos |
| [Ejecución y Depuración](editor-flujos/ejecucion-depuracion.md) | Ejecutar flujos, visualizar resultados y depurar fallos |

---

## Referencia de Nodos

### Control de Flujo
| Nodo | Descripción |
|------|-------------|
| [If](nodos/control-flujo/if.md) | Bifurcación condicional (verdadero/falso) |
| [Switch](nodos/control-flujo/switch.md) | Bifurcación múltiple por valor o condición |
| [Loop](nodos/control-flujo/loop.md) | Repetición por conteo, array o condición |
| [Merge](nodos/control-flujo/merge.md) | Unión de múltiples caminos de ejecución |

### Web
| Nodo | Descripción |
|------|-------------|
| [Web Flow](nodos/web/web-flow.md) | Automatización web con múltiples pasos y selectores CSS/XPath |
| [Smart Locators](nodos/web/smart-locators.md) | Automatización web con localizadores semánticos de Playwright |

### Mobile
| Nodo | Descripción |
|------|-------------|
| [Mobile Flow](nodos/mobile/mobile-flow.md) | Automatización de apps Android e iOS con Appium |
| [Inspector Mobile](nodos/mobile/inspector-mobile.md) | Grabación visual interactiva de pasos mobile |

### API
| Nodo | Descripción |
|------|-------------|
| [HTTP Request](nodos/api/http-request.md) | Solicitudes HTTP (GET, POST, PUT, PATCH, DELETE) |

### Base de Datos
| Nodo | Descripción |
|------|-------------|
| [PostgreSQL](nodos/base-datos/postgresql.md) | Consultas y operaciones en PostgreSQL |
| [MySQL](nodos/base-datos/mysql.md) | Consultas y operaciones en MySQL |
| [MariaDB](nodos/base-datos/mariadb.md) | Consultas y operaciones en MariaDB |
| [Oracle](nodos/base-datos/oracle.md) | Consultas y operaciones en Oracle |
| [MongoDB](nodos/base-datos/mongodb.md) | Operaciones en MongoDB (find, insert, update, etc.) |

### Infraestructura
| Nodo | Descripción |
|------|-------------|
| [SSH Command](nodos/infraestructura/ssh-command.md) | Ejecución de comandos SSH remotos |

### Utilidades
| Nodo | Descripción |
|------|-------------|
| [Set Variable](nodos/utilidades/set-variable.md) | Define variables en tiempo de ejecución |
| [Log](nodos/utilidades/log.md) | Registra mensajes en el log de ejecución |
| [Wait](nodos/utilidades/wait.md) | Espera un tiempo determinado o una condición |
| [Stop and Fail](nodos/utilidades/stop-and-fail.md) | Detiene el flujo con estado de fallo |
| [Custom JavaScript](nodos/utilidades/custom-javascript.md) | Ejecuta código JavaScript personalizado |
| [Email Inbox](nodos/utilidades/email-inbox.md) | Espera y extrae correos, OTPs y enlaces vía IMAP |

---

## Nodos Personalizados

| Página | Descripción |
|--------|-------------|
| [Descripción General](nodos-personalizados/vision-general.md) | Cómo funciona el sistema de proveedores |
| [Creando un Proveedor - Enterprise](nodos-personalizados/creando-proveedor-enterprise.md) | Guía paso a paso para crear un proveedor HTTP |
| [Contrato de la API](nodos-personalizados/contrato-api.md) | Endpoints requeridos y formato de datos |
| [Ejemplos](nodos-personalizados/ejemplos-enterprise.md) | Ejemplos en Node.js, Python, Java, C# y Go |
| [Escritorio: Nodos Locales](nodos-personalizados/nodos-locales-escritorio.md) | Creando nodos locales en la versión escritorio |
| [QANode.MD (IA)](nodos-personalizados/QANode.MD) | Guia de IA para crear nodos y diagnosticar problemas |

---

## Gestión

| Página | Descripción |
|--------|-------------|
| [Proyectos](proyectos/proyectos.md) | Creación y gestión de proyectos de prueba |
| [Suites de Prueba](suites/suites.md) | Agrupación de flujos y programación |
| [Variables](variables/variables.md) | Variables globales y secretas |
| [Credenciales](credenciales/credenciales.md) | Gestión segura de credenciales |

---

## CI/CD — Enterprise

| Página | Descripción |
|--------|-------------|
| [Visión General](ci-cd/vision-general.md) | Qué ofrece la integración, permisos y conceptos principales |
| [Tokens de Integración](ci-cd/tokens-integracion.md) | Generación, revocación, expiración y gobernanza de tokens |
| [CLI y API del CI/CD](ci-cd/cli-api.md) | Cómo usar `@qanode/cli`, las rutas `/api/ci` y los patrones operativos |
| [Ejemplos de Pipeline](ci-cd/ejemplos-pipelines.md) | Ejemplos listos para GitHub Actions y Azure DevOps |
| [Overrides por Ejecución](ci-cd/overrides.md) | Cómo sobrescribir variables y credenciales sin persistir cambios |

---

## Seguimiento de Defectos — Enterprise

| Página | Descripción |
|--------|-------------|
| [Descripción General](defectos/descripcion-general.md) | Funcionalidades del módulo, permisos y conceptos fundamentales |
| [Workflow Builder](defectos/workflow-builder.md) | Configurar estados, transiciones, campos y campos personalizados |
| [Ciclo de Vida del Defecto](defectos/ciclo-de-vida.md) | Abrir, asignar, hacer claim y tramitar defectos |
| [Sandbox de Investigación](defectos/sandbox.md) | Investigar fallos sin afectar ejecuciones oficiales |
| [Comentarios, Adjuntos e Historial](defectos/comentarios-adjuntos.md) | Colaborar y seguir el historial del defecto |

---

## Monitoreo e Informes

| Página | Descripción |
|--------|-------------|
| [Dashboard - Enterprise](dashboard/dashboard-enterprise.md) | Paneles, widgets y gráficos |
| [Informes - Enterprise](informes/informes-enterprise.md) | Generación de informes PDF y envío por correo electrónico |

---

## Referencia

| Página | Descripción |
|--------|-------------|
| [Expresiones](expresiones/expresiones.md) | Sistema de expresiones `{{ }}` e interpolación |
| [Administración - Enterprise](administracion/administracion-enterprise.md) | Usuarios, roles, permisos, SMTP y auditoría |
| [Versión Escritorio](escritorio/escritorio.md) | Instalación y funciones exclusivas de la versión escritorio |
| [Extensión Chrome](escritorio/extension-chrome.md) | Grabador de pruebas para el navegador |
