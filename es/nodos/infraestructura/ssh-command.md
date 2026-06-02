# Nodo SSH Command

El nodo **SSH Command** permite ejecutar comandos en servidores remotos vía SSH. Soporta múltiples pasos secuenciales, autenticación por contraseña o clave privada, envío/descarga de archivos y captura de salida.

---

## Visión General

[Visión General](../../assets/images/nos-ssh-visao-geral.mp4)

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `ssh-command` |
| **Categoría** | Infraestructura |
| **Color** | 🔵 Azul Claro (#0ea5e9) |
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
| **Contraseña** | `string` | Autenticación por contraseña |
| **Clave Privada** | `string` | Clave privada SSH |
| **Passphrase** | `string` | Frase de la clave, si está protegida |

### Métodos de Autenticación

| Método | Campos necesarios |
|--------|-------------------|
| **Contraseña** | Usuario + Contraseña |
| **Clave Privada** | Usuario + Clave Privada + Passphrase opcional |

---

## Configuración

### Timeout Global

| Campo | Tipo | Predeterminado | Descripción |
|-------|------|----------------|-------------|
| **Timeout (ms)** | `number` | 30000 | Tiempo máximo para la ejecución total |

### Pasos SSH

El nodo soporta múltiples pasos ejecutados secuencialmente en la misma conexión SSH. Use **+ Agregar paso** para incluir comandos, envíos de archivo y descargas.

| Tipo de paso | Descripción |
|--------------|-------------|
| **Comando** | Ejecuta un comando en el shell remoto |
| **Enviar archivo** | Envía un `fileRef` al servidor remoto vía SFTP |
| **Descargar archivo** | Descarga un archivo remoto vía SFTP, con fallback para SCP |

Todos los pasos se ejecutan en la misma sesión SSH. Esto significa que comandos como `cd /mi/carpeta` afectan los pasos siguientes cuando usan **Carpeta actual**.

### Paso Comando

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Rótulo** | `string` | Nombre descriptivo del paso |
| **Comando** | `string` | Comando a ejecutar (soporta `{{ }}`) |
| **Esperar Match** | `boolean` | Esperar texto en la salida |
| **Cadena de Match** | `string` | Texto a esperar |
| **Usar Regex** | `boolean` | Interpretar el match como regex |
| **Timeout del Match (ms)** | `number` | Tiempo máximo para el match |

### Paso Enviar Archivo

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Rótulo** | `string` | Nombre descriptivo del paso |
| **Archivo** | `fileRef` | Archivo a enviar |
| **Destino** | selección | **Carpeta actual** o **Ruta remota** |
| **Ruta remota** | `string` | Ruta absoluta o relativa en el servidor, cuando el destino es ruta remota |
| **Mantener nombre del archivo** | `boolean` | Usa el nombre original del archivo |
| **Nombre del archivo** | `string` | Nuevo nombre en el servidor cuando **Mantener nombre del archivo** está desactivado |
| **Crear carpetas remotas** | `boolean` | Crea directorios faltantes antes del envío |

Con **Destino = Carpeta actual**, QANode envía el archivo al directorio actual del shell SSH. Esto permite:

```
Paso 1: cd /opt/mi-app/uploads
Paso 2: Enviar archivo → Destino: Carpeta actual
```

Con **Destino = Ruta remota**, informe la ruta directamente. Si termina con `/`, el archivo se guarda dentro de esa carpeta.

### Paso Descargar Archivo

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Rótulo** | `string` | Nombre descriptivo del paso |
| **Origen** | selección | **Carpeta actual** o **Ruta remota** |
| **Nombre del archivo** | `string` | Nombre del archivo cuando el origen es carpeta actual |
| **Ruta remota** | `string` | Ruta del archivo en el servidor, cuando el origen es ruta remota |
| **Nombre de la variable** | `string` | Clave usada en `outputs.files` |
| **Nombre del archivo de salida** | `string` | Nombre guardado en QANode. Si está vacío, usa el nombre remoto |

Si se omite la extensión en la ruta remota, QANode intenta resolver el archivo por nombre en la carpeta remota. Por ejemplo, `/tmp/datos` puede encontrar `/tmp/datos.csv` cuando ese archivo existe en el servidor.

El MIME type se infiere automáticamente por el contenido y por el nombre del archivo.

### Esperar Match

El recurso **Esperar Match** permite que un paso de comando espere hasta que un texto específico aparezca en la salida. Útil para:

- esperar que servicios inicien;
- esperar prompts interactivos;
- confirmar que una operación terminó.

Ejemplo:

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
| `exitCode` | `number` | Código de salida (0 = éxito) |
| `commands` | `array` | Resultado de cada paso de comando |
| `files` | `object` | Archivos descargados, cuando existe paso **Descargar archivo** |
| `fileRef` | `fileRef` | Atajo para el último archivo descargado |
| `fileInfo` | `object` | Metadatos de los archivos descargados, como nombre, MIME type, tamaño, ruta remota y método de transferencia |

### Accediendo a los Outputs

```
// Salida completa
{{ steps.ssh.outputs.stdout }}

// Código de salida
{{ steps.ssh.outputs.exitCode }}

// Resultado de un comando específico
{{ steps.ssh.outputs.commands[0].stdout }}
{{ steps.ssh.outputs.commands[0].exitCode }}

// Archivo descargado por nombre de variable
{{ steps.ssh.outputs.files.reporte }}

// Atajo al último archivo descargado
{{ steps.ssh.outputs.fileRef }}
```

---

## Ejemplos Prácticos

### Verificar estado de servicio

```
Paso 1:
  Rótulo: Verificar Nginx
  Comando: systemctl status nginx
```

### Deploy de aplicación

```
Paso 1:
  Rótulo: Ir al directorio
  Comando: cd /opt/mi-app

Paso 2:
  Rótulo: Pull del repositorio
  Comando: git pull origin main

Paso 3:
  Rótulo: Instalar dependencias
  Comando: npm install --production

Paso 4:
  Rótulo: Reiniciar servicio
  Comando: pm2 restart mi-app
  Esperar Match: true
  Cadena de Match: "online"
```

### Enviar y descargar archivo en la carpeta actual

```
Paso 1:
  Rótulo: Entrar en la carpeta
  Comando: cd /root/pruebas

Paso 2:
  Rótulo: Enviar datos
  Tipo: Enviar archivo
  Archivo: {{ steps["file-generate"].outputs.fileRef }}
  Destino: Carpeta actual
  Mantener nombre del archivo: true

Paso 3:
  Rótulo: Descargar resultado
  Tipo: Descargar archivo
  Origen: Carpeta actual
  Nombre del archivo: resultado.csv
  Nombre de la variable: resultado
```

Después de la descarga:

```
{{ steps.ssh.outputs.files.resultado }}
{{ steps.ssh.outputs.fileInfo.resultado.name }}
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
    │ false → [Log: "Deploy falló: {{ steps.ssh.outputs.stderr }}"]
              → [Stop and Fail]
```

---

## Consejos

- **Use credenciales guardadas** para evitar exponer contraseñas/claves en los flujos.
- **Verifique `exitCode`**: `0` indica éxito, otros valores indican error.
- Use expresiones en comandos: `echo "Deploy en {{ variables.ENVIRONMENT }}"`.
- **Esperar Match** es esencial para comandos demorados como reinicios y builds.
- Todos los pasos se ejecutan en la **misma sesión SSH**, por lo que el directorio de trabajo se mantiene entre pasos.
- Use **Carpeta actual** cuando un comando anterior ya posicionó el shell en el directorio correcto.
- Use **Ruta remota** cuando quiera ser explícito e independiente del `cd` anterior.
- Para descargar archivos, prefiera un **Nombre de variable** claro, como `reporte`, `datos` o `evidencia`.
- Los comandos con `{{ }}` se evalúan antes de la ejecución.
