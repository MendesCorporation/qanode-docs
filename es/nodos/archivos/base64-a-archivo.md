# Nodo Base64 a Archivo

El nodo **Base64 a Archivo** recibe contenido Base64 o Data URI y crea un archivo dentro de QANode, devolviendo un `fileRef` para los nodos siguientes.

---

## Visión General

[Visión General](../../assets/images/nos-base64-to-file.mp4)

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `file-from-base64` |
| **Categoría** | Archivos |
| **Color** | 🟤 Dorado (#C89F65) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Cuándo Usar

Usa este nodo cuando:

- una API devuelve un archivo como Base64;
- una integración personalizada produce un Data URI;
- necesitas transformar Base64 en `fileRef` antes de enviarlo a otro nodo;
- quieres convertir Base64 en archivo real y luego extraerlo.

---

## Configuración

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Base64** | `string` | Contenido Base64 o Data URI |
| **Nombre del archivo** | `string` | Nombre final del archivo |

El usuario no necesita llenar el MIME type. QANode intenta inferirlo automáticamente a partir de:

- prefijo Data URI, cuando existe;
- firma del archivo;
- extensión del archivo.

Ejemplo de Data URI:

```
data:application/pdf;base64,JVBERi0xLjQK...
```

Ejemplo de Base64 puro:

```
JVBERi0xLjQK...
```

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `fileRef` | `fileRef` | Referencia del archivo generado |
| `name` | `string` | Nombre del archivo |
| `mimeType` | `string` | MIME type inferido |
| `sizeBytes` | `number` | Tamaño del archivo |

### Accediendo a los Outputs

```
{{ steps["file-from-base64"].outputs.fileRef }}
{{ steps["file-from-base64"].outputs.name }}
{{ steps["file-from-base64"].outputs.mimeType }}
{{ steps["file-from-base64"].outputs.sizeBytes }}
```

---

## Ejemplos

### API devuelve PDF en Base64

```
[HTTP Request] → devuelve body.documentBase64
[Base64 a Archivo]
  Base64: {{ steps["http-request"].outputs.body.documentBase64 }}
  Nombre del archivo: contrato.pdf
```

Después:

```
{{ steps["file-from-base64"].outputs.fileRef }}
```

### Convertir imagen Data URI

```
Base64: {{ steps["custom-js"].outputs.result.imageDataUri }}
Nombre del archivo: screenshot.png
```

---

## Ida y Vuelta

Este nodo es el inverso de **Archivo a Base64**:

```
[Archivo a Base64] → [Base64 a Archivo]
```

o:

```
[Base64 a Archivo] → [Extraer Archivo]
```

---

## Consejos

- Prefiere Data URI cuando el origen ya informa el MIME type.
- Si usas Base64 puro, informa un nombre de archivo con extensión para ayudar la inferencia.
- Para validar el contenido, encadena con **Extraer Archivo**.

