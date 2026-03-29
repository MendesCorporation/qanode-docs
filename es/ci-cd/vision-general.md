# Integración CI/CD — Visión General

> **Disponible en:** QANode Enterprise

La integración CI/CD de QANode permite que su empresa ejecute escenarios y suites desde el pipeline, autentique con tokens de integración y descargue el informe consolidado de cada ejecución sin depender de una sesión de navegador.

En lugar de tratar QANode solo como una interfaz visual, su empresa puede usarlo como parte oficial de la validación de builds, pull requests, releases o despliegues.

---

## Qué ofrece la integración

- **Tokens de integración** para autenticación de pipelines y automatizaciones
- **CLI oficial (`@qanode/cli`)** para GitHub Actions, Azure DevOps, GitLab, Jenkins y scripts locales
- **Rutas `/api/ci` dedicadas** para escenarios, suites, consulta de ejecuciones y descarga de informes
- **Ejecución por ID o por nombre** con soporte de proyecto cuando sea necesario
- **Overrides por ejecución** para variables y credenciales
- **Salida simplificada para pipeline** con `success`, `failed` o `cancelled`
- **Informe PDF consolidado** para adjuntar al job

---

## Cómo funciona a alto nivel

El flujo operativo más común es:

1. El equipo crea un **token de integración** en **Configuración → Access Tokens**
2. El pipeline guarda ese token como **secret**
3. El job ejecuta la CLI de QANode con `QANODE_URL` y `QANODE_TOKEN`
4. QANode crea una ejecución real en la instancia
5. El pipeline espera el resultado final y decide si continúa o falla
6. Opcionalmente, el job descarga el **informe PDF consolidado**

---

## Conceptos principales

### URL pública de QANode

`QANODE_URL` debe apuntar a la **URL pública** que su equipo ya usa para abrir QANode en el navegador.

En la mayoría de los entornos corporativos esto significa:

- el mismo dominio público del frontend
- con proxy o balanceador encaminando `/api` hacia la API de QANode

Ejemplos:

```bash
https://qanode.empresa.com
https://qa.empresa.com/qanode
```

En desarrollo local, `http://localhost:3000` también puede funcionar cuando el frontend proxy `/api`.

### Token de integración

Es un token creado específicamente para automatización. Reemplaza el login humano en el pipeline y respeta los permisos del usuario o rol asociado.

### Escenario

Es una automatización individual de QANode, compuesta por el flujo de pasos que valida un comportamiento específico de la aplicación. En CI/CD, puedes ejecutar un escenario aislado cuando quieras validar un camino crítico sin correr la suite completa.

### Suite

La suite agrupa varios escenarios y es el recurso más común para regresión completa, smoke tests y gates de release.

### Override por ejecución

Los overrides permiten cambiar valores de variables o campos de credenciales **solo para esa ejecución del pipeline**, sin sobrescribir el valor persistido en QANode.

---

## Permisos involucrados

Los permisos principales de CI/CD son:

| Permiso | Lo que permite |
|---------|----------------|
| `settings.integration_token` | Crear y revocar sus propios tokens de integración |
| `settings.integration_token_all` | Ver todos los tokens, revocar tokens de otras personas y definir la política global de expiración |
| `flow.run` | Ejecutar escenarios mediante CLI/API |
| `suite.run` | Ejecutar suites mediante CLI/API |
| `run.view` / `run.view.all` | Consultar ejecuciones y descargar el informe |

Los permisos de tokens se configuran en **Configuración → Roles**.

---

## Lo que la integración no hace automáticamente

Hay algunos comportamientos importantes para evitar expectativas incorrectas:

- la CLI **no descubre el token por sí sola**; el pipeline debe proporcionarlo
- la CLI **no necesita sesión de navegador**
- la ejecución disparada por el pipeline es una **run real** de QANode
- los overrides de variables y credenciales **no cambian permanentemente** los valores guardados
- el pipeline debe reutilizar el **mismo ID de la run** al descargar el informe, en lugar de disparar una segunda ejecución

---

## Navegación de esta sección

| Página | Lo que aprenderá |
|--------|------------------|
| [Tokens de Integración](./tokens-integracion.md) | Cómo generar, revocar y gobernar tokens de pipeline |
| [CLI y API del CI/CD](./cli-api.md) | Cómo autenticar, disparar escenarios/suites y usar overrides |
| [Ejemplos de Pipeline](./ejemplos-pipelines.md) | Cómo integrar con GitHub Actions y Azure DevOps |
| [Overrides por Ejecución](./overrides.md) | Cómo sobrescribir variables y credenciales de forma segura |

---

## Próximos Pasos

- [Tokens de Integración](./tokens-integracion.md) — Aprenda a generar, revocar y gobernar las claves usadas por el pipeline
