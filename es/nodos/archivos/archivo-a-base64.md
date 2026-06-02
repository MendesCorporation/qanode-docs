# Nodo Archivo a Base64

El nodo **Archivo a Base64** recibe un `fileRef` y devuelve el contenido del archivo en Base64 y Data URI.

---

## Visión General

| Propiedad | Valor |
|-----------|-------|
| **Tipo** | `file-to-base64` |
| **Categoría** | Archivos |
| **Color** | 🟤 Dorado (#C89F65) |
| **Entrada** | `in` |
| **Salida** | `out` |

---

## Cuándo Usar

Usa este nodo cuando:

- una API espera el archivo dentro de un campo JSON como Base64;
- necesitas enviar el contenido del archivo en un mensaje o payload;
- quieres convertir un `fileRef` en Data URI;
- otro sistema no acepta upload multipart o upload binario.

---

## Configuración

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **Archivo** | `fileRef` | Archivo a convertir |
| **Incluir Data URI** | `boolean` | También devuelve `dataUri` con prefijo MIME |

---

## Outputs

| Output | Tipo | Descripción |
|--------|------|-------------|
| `base64` | `string` | Contenido del archivo en Base64 |
| `dataUri` | `string` | Data URI cuando está activo |
| `name` | `string` | Nombre del archivo |
| `mimeType` | `string` | MIME type |
| `sizeBytes` | `number` | Tamaño original |

### Accediendo a los Outputs

```
{{ steps["file-to-base64"].outputs.base64 }}
{{ steps["file-to-base64"].outputs.dataUri }}
{{ steps["file-to-base64"].outputs.mimeType }}
```

---

## Ejemplo: Enviar Base64 en JSON

```
[Generar Archivo] → [Archivo a Base64] → [HTTP Request]
```

Body HTTP:

```json
{
  "fileName": "{{ steps["file-to-base64"].outputs.name }}",
  "mimeType": "{{ steps["file-to-base64"].outputs.mimeType }}",
  "content": "{{ steps["file-to-base64"].outputs.base64 }}"
}
```

---

## Ida y Vuelta

Usa junto con **Base64 a Archivo** cuando necesites convertir en los dos sentidos:

```
fileRef → Base64 → fileRef
```

---

## Consejos

- Prefiere `fileRef` entre nodos de QANode. Convierte a Base64 solo cuando el sistema externo lo exige.
- Base64 aumenta el tamaño del payload; evita usarlo con archivos muy grandes.
- Usa `dataUri` cuando el sistema receptor espera el formato `data:mime/type;base64,...`.

