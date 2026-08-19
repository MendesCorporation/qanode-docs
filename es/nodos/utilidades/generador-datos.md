# Nodo Generador de Datos

El nodo **Generador de Datos** crea datos sintéticos para pruebas sin depender de servicios externos. En una sola ejecución puede generar de 1 a 1.000 registros con datos de identidad, documentos, direcciones, empresas, internet, textos, números, fechas, tarjetas y comercio.

Úselo cuando un escenario necesite usuarios, pedidos, productos, documentos u otros valores nuevos en cada ejecución. Los campos configurados aparecen en el selector de variables y pueden utilizarse directamente en los nodos siguientes.

> Los datos son sintéticos. Un documento estructuralmente válido respeta el formato y los dígitos de control aplicables, pero no representa necesariamente a una persona, empresa, dirección, teléfono o medio de pago real.

---

## Configuración General

| Campo | Descripción |
|-------|-------------|
| **Idioma y formato regional** | Idioma y convenciones regionales usados para generar los valores |
| **País predeterminado** | País usado por documentos, teléfonos, direcciones y empresas, salvo que el campo indique otro |
| **Cantidad** | Número de registros generados, entre 1 y 1.000 |
| **Caracteres** | Mantiene caracteres naturales o normaliza el texto a ASCII eliminando acentos |
| **Semilla** | Valor opcional que reproduce los mismos datos con la misma configuración |
| **Campos generados** | Propiedades que formarán cada registro |

Los formatos regionales disponibles incluyen portugués de Brasil y Portugal; inglés de Estados Unidos, Reino Unido, Canadá y Australia; español, francés, alemán, italiano, japonés y chino simplificado.

---

## Agregar Campos

Haga clic en **Agregar campo**, seleccione el tipo de dato e informe un **Nombre de salida** único. Ese nombre será la clave utilizada en las expresiones de los nodos siguientes.

| Categoría | Ejemplos |
|-----------|----------|
| **Identidad y datos personales** | nombre, apellido, nombre completo, correo, usuario, contraseña, nacimiento, edad, teléfono y avatar ficticio |
| **Documentos y tarjetas** | documento personal y empresarial, pasaporte, número, emisor, CVV y vencimiento de la tarjeta |
| **Dirección y ubicación** | dirección completa, calle, número, barrio, ciudad, estado, código postal, país y coordenadas |
| **Empresa** | razón social, nombre comercial, departamento, cargo, dominio, correo y teléfono corporativo |
| **Internet y desarrollo** | UUID, URL, dominio, IPv4, IPv6, dirección MAC, hash, token, slug, color hexadecimal y user agent |
| **Textos, números y valores** | palabra, frase, párrafo, entero, decimal, valor monetario, porcentaje, booleano, fechas, lista, secuencia, string y template |
| **Comercio** | producto, SKU, precio, cantidad, moneda, número de pedido, rastreo y estado del pedido |

Las opciones dependen del tipo seleccionado y pueden incluir país, formato, validez estructural, emisor de tarjeta, máscara, tamaño, prefijo, sufijo, dominio, moneda, decimales, rangos numéricos y de fecha, listas y composición de contraseña.

### Registros coherentes

Los campos relacionados comparten el mismo contexto dentro de cada registro. Por ejemplo:

- nombre, apellido, nombre completo, usuario y correo pertenecen a la misma persona sintética;
- ciudad, estado, código postal y dirección completa pertenecen a la misma dirección sintética;
- número, emisor, CVV y vencimiento pertenecen a la misma tarjeta sintética;
- producto, SKU, precio, moneda, pedido y rastreo pertenecen al mismo contexto comercial.

Al generar varios registros, cada elemento recibe su propio contexto.

---

## Datos Válidos e Inválidos

Los campos compatibles pueden configurarse como **Estructuralmente válido** o **Estructuralmente inválido**. El modo inválido es útil para probar rechazos y mensajes de error.

Esta opción está disponible para documentos personales y empresariales, pasaportes, teléfonos, números de tarjeta y máscaras personalizadas. El valor se altera deliberadamente y no se consultan bases gubernamentales, operadoras o emisores.

---

## Máscaras, Secuencias y Templates

Las máscaras personalizadas usan:

| Carácter | Resultado |
|----------|-----------|
| `#` | Dígito |
| `A` | Letra mayúscula |
| `a` | Letra minúscula |
| `*` | Carácter alfanumérico |

Ejemplo: `PED-####-AA`.

Una **Secuencia** permite definir inicio, incremento y relleno con ceros. Un **Template** combina valores del registro con texto fijo, secuencias y máscaras:

```text
USR-{sequence}-{firstName}-###
```

Los valores insertados, como `{firstName}`, se conservan literalmente. Los caracteres `A`, `a`, `*` o `#` incluidos en el nombre no se interpretan nuevamente como máscara.

---

## Salidas

| Salida | Descripción |
|--------|-------------|
| `data` | Array con todos los registros generados |
| `count` | Cantidad de registros generados |
| `first` | Primer registro generado |

Si el nodo se llama **Generar usuario** y contiene campos llamados `nombre` y `email`, use:

```text
{{ steps["Generar usuario"].outputs.first.nombre }}
{{ steps["Generar usuario"].outputs.first.email }}
{{ steps["Generar usuario"].outputs.count }}
{{ steps["Generar usuario"].outputs.data }}
```

Para procesar varios registros, pase `outputs.data` a un nodo [Loop](../control-flujo/loop.md).

---

## Generación Asistida por IA

Una IA conectada a QANode mediante MCP puede consultar el contrato del Generador de Datos, crear su configuración y usar los nombres de salida en los pasos siguientes. Describa los datos por el objetivo de la prueba, por ejemplo:

```text
Crea diez clientes brasileños con nombre, correo en el dominio example.com,
CPF válido, teléfono internacional y número de pedido secuencial.
```

Revise los nombres de salida y las reglas de validez antes de ejecutar, especialmente en pruebas negativas.

---

## Buenas Prácticas

- Use una semilla cuando necesite reproducir una falla con los mismos datos.
- Use nombres de salida cortos y únicos.
- Genere solo los campos necesarios para el escenario.
- Use ASCII cuando el sistema probado no acepte acentos o caracteres no latinos.
- No utilice los valores generados como datos personales, credenciales o información de pago real.
