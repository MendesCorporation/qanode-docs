# Nodo Extraer Archivo

El nodo **Extraer Archivo** lee un archivo recibido como `fileRef` y devuelve texto o datos estructurados según el tipo de extracción elegido.

---

## Visión General

[Visión General](../../assets/images/nos-extract-file.mp4)

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `file-extract` |
| **Categoría** | Archivos |
| **Color** | 🟤 Dorado (#C89F65) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Cuándo Usar

Usa este nodo cuando el flujo necesita:

- leer TXT o Markdown como texto;
- validar o reutilizar contenido JSON;
- transformar CSV en filas estructuradas;
- leer planillas Excel;
- extraer texto de PDF;
- usar OCR como fallback cuando el PDF es una imagen.

---

## Configuración

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Archivo** | `fileRef` | Archivo de entrada |
| **Tipo de extracción** | selección | TXT, JSON, CSV, Excel o PDF |

Los campos siguientes cambian según el tipo de extracción.

### TXT

| Campo | Descripción |
|-------|-------------|
| **Codificación** | Codificación usada para leer el texto, como `utf8` |

### JSON

| Campo | Descripción |
|-------|-------------|
| **Codificación** | Codificación usada para leer el archivo |

El contenido debe ser JSON válido.

### CSV

| Campo | Descripción |
|-------|-------------|
| **Delimitador** | Separador de columnas, como `,` o `;` |
| **Tiene encabezado** | Usa la primera línea como nombres de columnas |
| **Codificación** | Codificación del archivo |

### Excel

| Campo | Descripción |
|-------|-------------|
| **Nombre de hoja** | Hoja a leer. Si está vacío, usa la primera hoja |
| **Fila de encabezado** | Fila usada como encabezado |

### PDF

| Campo | Descripción |
|-------|-------------|
| **Rango de páginas** | Páginas a extraer, como `1-3,5` |
| **OCR** | Activa OCR como fallback para PDFs en imagen |
| **Idiomas OCR** | Idiomas del OCR, como `por+eng+spa` |
| **Escala OCR** | Escala usada al renderizar antes del OCR |

---

## OCR en PDF

Cuando el PDF tiene texto real, QANode intenta extraerlo directamente. Si el PDF parece ser imagen, el OCR puede usarse automáticamente como fallback.

Usa OCR para:

- PDFs escaneados;
- comprobantes y recibos en imagen;
- documentos digitalizados;
- archivos sin capa de texto.

Idiomas comunes:

| Valor | Idiomas |
|-------|---------|
| `por` | Portugués |
| `eng` | Inglés |
| `spa` | Español |
| `por+eng+spa` | Portugués, inglés y español |

---

## Outputs

Los outputs dependen del tipo elegido.

### TXT

| Output | Tipo | Descripción |
|--------|------|-------------|
| `text` | `string` | Texto extraído |

### JSON

| Output | Tipo | Descripción |
|--------|------|-------------|
| `json` | `any` | JSON parseado |
| `text` | `string` | Texto original |

### CSV y Excel

| Output | Tipo | Descripción |
|--------|------|-------------|
| `rows` | `array` | Filas extraídas |
| `columns` | `array` | Columnas detectadas |
| `rowCount` | `number` | Cantidad de filas |
| `sheets` | `array` | Hojas encontradas, para Excel |

### PDF

| Output | Tipo | Descripción |
|--------|------|-------------|
| `text` | `string` | Texto extraído |
| `pages` | `array` | Texto por página |
| `pageCount` | `number` | Cantidad de páginas procesadas |

---

## Ejemplos

### Extraer CSV

```
Archivo: {{ steps["file-generate"].outputs.fileRef }}
Tipo de extracción: CSV
Delimitador: ,
Tiene encabezado: true
```

Uso posterior:

```
{{ steps["file-extract"].outputs.rows[0].email }}
{{ steps["file-extract"].outputs.rowCount }}
```

### Extraer texto de PDF

```
Archivo: {{ steps["http-request"].outputs.fileRef }}
Tipo de extracción: PDF
Rango de páginas: 1-2
OCR: true
Idiomas OCR: por+eng+spa
```

---

## Validación de Tipo

El nodo valida si el archivo es compatible con el tipo de extracción. Si el archivo no es soportado, el campo queda en rojo en el panel y la ejecución falla con un mensaje claro, por ejemplo:

```
Archivo no soportado: planilla.xlsx
```

---

## Consejos

- Elige el tipo de extracción según el contenido real del archivo.
- Para PDF escaneado, habilita OCR; para PDF con texto real, deja OCR como fallback.
- Para CSV con `;`, cambia el delimitador antes de ejecutar.
- Usa `rows` para alimentar loops, componentes, generación de archivos o validaciones.

