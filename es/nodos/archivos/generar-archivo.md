# Nodo Generar Archivo

El nodo **Generar Archivo** crea archivos dentro de una ejecución de QANode. Puede generar TXT, JSON, CSV, Excel o PDF a partir de contenido informado en el panel o producido por expresiones de nodos anteriores.

---

## Visión General

[Visión General](../../assets/images/nos-generate-file.mp4)

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `file-generate` |
| **Categoría** | Archivos |
| **Color** | 🟤 Dorado (#C89F65) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Cuándo Usar

Usa este nodo cuando el flujo necesita:

- crear datos de prueba en CSV o Excel;
- generar JSON para enviar a una API;
- montar un archivo TXT auxiliar;
- crear un PDF simple para adjuntar, descargar o validar;
- producir un archivo que será usado por HTTP Request, Smart Web Flow, Mobile Flow, SSH, un componente o Custom JavaScript.

---

## Configuración

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Tipo de archivo** | selección | TXT, JSON, CSV, Excel o PDF |
| **Nombre del archivo** | texto | Nombre final del archivo, incluyendo la extensión |

Los demás campos cambian según el tipo elegido.

### TXT

| Campo | Descripción |
|-------|-------------|
| **Contenido** | Texto del archivo. Acepta expresiones `{{ }}` |

### JSON

| Campo | Descripción |
|-------|-------------|
| **JSON** | Objeto o array JSON válido |

### CSV

| Campo | Descripción |
|-------|-------------|
| **Modo** | Líneas manuales o a partir de lista |
| **Columnas** | Nombres de columnas |
| **Líneas** | Valores de cada línea cuando el modo es manual |
| **Lista** | Array de otro nodo cuando el modo es a partir de lista |
| **Mapeo** | Campo del item usado en cada columna |

### Excel

| Campo | Descripción |
|-------|-------------|
| **Nombre de hoja** | Nombre de la hoja dentro del archivo |
| **Modo** | Líneas manuales o a partir de lista |
| **Columnas / Líneas / Lista** | Misma lógica del CSV |

### PDF

| Campo | Descripción |
|-------|-------------|
| **Título** | Título opcional mostrado en el PDF |
| **Contenido** | Texto o JSON que se renderizará en el PDF |

---

## Tabla Manual y A Partir de Lista

Para CSV y Excel hay dos formas principales de montar la tabla.

### Líneas manuales

Úsalo cuando los datos son pequeños o fijos.

```
Columnas: id, nombre, email
Línea 1: 1, Maria, maria@empresa.com
Línea 2: 2, Juan, juan@empresa.com
```

### A partir de lista

Úsalo cuando los datos ya vienen de una API, base de datos, componente o nodo anterior.

```
Lista: {{ steps["consulta"].outputs.rows }}
Columnas: id, nombre, email
Mapeos: id, nombre, email
```

Si la lista contiene objetos, QANode puede inferir columnas comunes, pero en flujos compartidos es mejor definirlas explícitamente.

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `fileRef` | `fileRef` | Referencia del archivo generado |
| `name` | `string` | Nombre del archivo |
| `mimeType` | `string` | MIME type del archivo |
| `sizeBytes` | `number` | Tamaño en bytes |

### Accediendo a los Outputs

```
{{ steps["file-generate"].outputs.fileRef }}
{{ steps["file-generate"].outputs.name }}
{{ steps["file-generate"].outputs.mimeType }}
{{ steps["file-generate"].outputs.sizeBytes }}
```

Usa `fileRef` para pasar el archivo a otros nodos. Los demás campos sirven para logs, validaciones y mensajes.

---

## Ejemplo: Generar CSV de Datos

```
Tipo de archivo: CSV
Nombre del archivo: clientes.csv
Modo: A partir de lista
Lista: {{ steps["consulta-clientes"].outputs.rows }}
Columnas:
  - id
  - nombre
  - email
```

Después, el archivo puede usarse con:

```
{{ steps["file-generate"].outputs.fileRef }}
```

---

## Consejos

- Incluye la extensión en el nombre: `.txt`, `.json`, `.csv`, `.xlsx` o `.pdf`.
- Para CSV y Excel, prefiere **A partir de lista** cuando el origen ya es un array.
- Usa nombres claros, como `datos-clientes.csv` o `reporte-pedido.pdf`.
- Para enviar un archivo a una API o hacer upload web, pasa siempre el `fileRef`, no el contenido en texto.

