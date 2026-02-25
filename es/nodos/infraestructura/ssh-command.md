# Nodo SSH Command

El nodo **SSH Command** permite ejecutar comandos en servidores remotos a través del protocolo SSH. Admite múltiples pasos (comandos secuenciales), autenticación por contraseña o clave privada, y captura de la salida.

---

## Descripción General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `ssh-command` |
| **Categoría** | Infraestructura |
| **Color** | Azul Claro (#0ea5e9) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Conexión

### Usar Credenciales Guardadas (Recomendado)

Seleccione una credencial de tipo **SSH** en el campo **Credencial**.

### Configuración Manual

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Host** | `string` | Dirección del servidor |
| **Puerto** | `number` | Puerto SSH (predeterminado: 22) |
| **Usuario** | `string` | Nombre de usuario |
| **Contraseña** | `string` | Contraseña (autenticación por contraseña) |
| **Clave Privada** | `string` | Clave privada SSH (autenticación por clave) |
| **Passphrase** | `string` | Frase de contraseña de la clave (si está protegida) |

### Métodos de Autenticación

| Método | Campos Requeridos |
|--------|------------------|
| **Contraseña** | Usuario + Contraseña |
| **Clave Privada** | Usuario + Clave Privada + Passphrase (opcional) |

---

## Configuración

### Timeout Global

| Campo | Tipo | Predeterminado | Descripción |
|-------|------|----------------|-------------|
| **Timeout (ms)** | `number` | 30000 | Tiempo máximo para la ejecución total |

### Pasos (SSH Steps)

El nodo admite múltiples pasos ejecutados secuencialmente en la misma conexión SSH:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Label** | `string` | Nombre descriptivo del paso |
| **Comando** | `string` | Comando a ejecutar (admite `{{ }}`) |
| **Esperar Match** | `boolean` | Esperar texto en la salida |
| **Cadena de Match** | `string` | Texto a esperar |
| **Usar Regex** | `boolean` | Interpretar el match como expresión regular |
| **Timeout del Match (ms)** | `number` | Tiempo máximo para el match |

### Esperar Match

La función **Esperar Match** permite que el paso espere hasta que un texto específico aparezca en la salida del comando. Útil para:
- Esperar que los servicios inicien
- Esperar prompts interactivos
- Confirmar que una operación se completó

**Ejemplo:**
```
Comando: systemctl restart nginx
Esperar Match: true
Cadena de Match: "Active: active (running)"
Timeout: 10000
```

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `stdout` | `string` | Salida estándar completa |
| `stderr` | `string` | Salida de error |
| `exitCode` | `number` | Código de salida (0 = éxito) |
| `steps` | `array` | Resultado de cada paso individual |

### Acceder a los Outputs

```
// Salida completa
{{ steps.ssh.outputs.stdout }}

// Código de salida
{{ steps.ssh.outputs.exitCode }}

// Resultado de un paso específico
{{ steps.ssh.outputs.steps[0].stdout }}
{{ steps.ssh.outputs.steps[0].exitCode }}
```

---

## Ejemplos Prácticos

### Verificar estado de un servicio

```
Paso 1:
  Label: Verificar Nginx
  Comando: systemctl status nginx
```

### Despliegue de aplicación

```
Paso 1:
  Label: Ir al directorio
  Comando: cd /opt/minha-app

Paso 2:
  Label: Pull del repositorio
  Comando: git pull origin main

Paso 3:
  Label: Instalar dependencias
  Comando: npm install --production

Paso 4:
  Label: Reiniciar servicio
  Comando: pm2 restart minha-app
  Esperar Match: true
  Cadena de Match: "online"
```

### Recopilar información del servidor

```
Paso 1:
  Label: Uso de disco
  Comando: df -h /

Paso 2:
  Label: Memoria
  Comando: free -m

Paso 3:
  Label: Procesos
  Comando: ps aux --sort=-%mem | head -10
```

---

## Flujo con Validación

```
[SSH Command: "deploy"]
    │
    ▼
[If: {{ steps.ssh.outputs.exitCode }} === 0]
    │ true → [HTTP Request: GET /health-check]
    │            │
    │            ▼
    │         [Assert: status === 200]
    │
    │ false → [Log: "Deploy falhou: {{ steps.ssh.outputs.stderr }}"]
              → [Stop and Fail]
```

---

## Consejos

- **Use credenciales guardadas** para evitar exponer contraseñas/claves en los flujos
- **Verifique el exitCode** — `0` indica éxito, otros valores indican un error
- Use **expresiones** en los comandos: `echo "Deploy em {{ variables.ENVIRONMENT }}"`
- **Esperar Match** es esencial para comandos de larga duración (reinicios, compilaciones)
- Todos los pasos se ejecutan en la **misma sesión SSH** — el directorio de trabajo se mantiene entre pasos
- Los comandos con `{{ }}` son evaluados antes de la ejecución
