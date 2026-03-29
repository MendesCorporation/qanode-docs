# Overrides por Ejecución

> **Disponible en:** QANode Enterprise

Los overrides permiten cambiar valores de variables y campos de credenciales solo durante la ejecución actual del pipeline, sin modificar la configuración permanente guardada en QANode.

---

## Cuándo usar

Los overrides son útiles cuando su empresa necesita:

- apuntar a un entorno temporal de PR
- cambiar la URL base de una API
- usar un token efímero del pipeline
- cambiar una contraseña temporal de homologación
- ejecutar la misma suite contra varios entornos sin editar el flujo

---

## Override de variables

Use:

```bash
--var NOMBRE=VALOR
```

Ejemplos:

```bash
--var BASE_URL=https://preview.app
--var LOCALE=en-US
--var FEATURE_FLAG=true
```

Ejemplo completo:

```bash
npx @qanode/cli run scenario \
  --project-name "Checkout" \
  --scenario-name "Login API" \
  --var BASE_URL=https://preview.app \
  --wait
```

---

## Override de credenciales

Use:

```bash
--credential CREDENCIAL.CAMPO=VALOR
```

Ejemplos:

```bash
--credential api-main.token=$API_TOKEN
--credential smtp-main.password=$SMTP_PASSWORD
--credential pg-staging.host=db-preview.internal
```

Ejemplo completo:

```bash
npx @qanode/cli run scenario \
  --scenario-id SCENARIO_ID \
  --credential api-main.token=$API_TOKEN \
  --credential api-main.baseUrl=https://preview.api \
  --wait
```

---

## Comportamiento de los overrides

Los overrides:

- solo se aplican a la ejecución actual
- no sobrescriben el valor guardado en la base de datos
- no crean un nuevo registro de variable o credencial

En otras palabras, QANode usa el valor override únicamente en esa run.

---

## Relación con variables y credenciales guardadas

El valor persistido sigue siendo la referencia oficial del producto:

- [Variables](../variables/variables.md)
- [Credenciales](../credenciales/credenciales.md)

El override es una capa temporal por encima de esa configuración.

---

## Buenas prácticas

- Use overrides solo para valores que realmente cambian por ambiente o pipeline
- Guarde secretos en los **Secrets** de su proveedor de CI/CD
- Prefiera URLs completas, incluyendo `https://`, cuando el escenario navegue directamente a una página
- Si el objetivo se identifica por nombre, informe también el proyecto

---

## Lo que no cambia automáticamente

Hoy el override se aplica a la ejecución disparada por el pipeline. Por sí solo, no implica que otros flujos derivados hereden automáticamente el mismo contexto.

Por eso, trate el override como parte del contexto de ejecución de CI/CD, no como un cambio permanente del escenario.

---

## Próximos Pasos

- [Ejemplos de Pipeline](./ejemplos-pipelines.md) — Vea implementaciones completas para GitHub Actions y Azure DevOps
