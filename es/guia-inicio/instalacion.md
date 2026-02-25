# Instalación


---

## Versión Desktop

La versión desktop es la forma más fácil de comenzar. Incluye todos los componentes necesarios en un único instalador, incluyendo una base de datos PostgreSQL integrada.

### Requisitos

- **Sistema Operativo**: Windows 10/11 (64-bit)
- **Memoria RAM**: 8 GB mínimo (16 GB recomendado)
- **Espacio en Disco**: 2 GB para la instalación

### Instalación en Windows

1. Descarga el instalador `QANode Setup.exe` en el sitio https://www.qanode.com
2. Ejecuta el instalador y sigue las instrucciones
3. Al finalizar, QANode se abrirá automáticamente

[Instalador de QANode en Windows](../../assets/images/instalacao-nsis.png)
*Imagen: Pantalla del instalador NSIS de QANode en Windows*

En el primer inicio, QANode:
- Inicializará la base de datos PostgreSQL integrada
- Creará las tablas necesarias (migraciones automáticas)
- Iniciará el servidor de la API
- Abrirá la interfaz de la aplicación

> **Nota:** El primer inicio puede tardar algunos segundos más mientras se prepara la base de datos. Se mostrará una pantalla de splash durante este proceso.

### Carpeta de Datos

La versión desktop almacena sus datos en:

```
%USERPROFILE%\AppData\Roaming\QANode\
├── pg-data/         # Datos de PostgreSQL integrado
├── storage/         # Artefactos (capturas de pantalla, PDFs, etc.)
└── custom nodes/    # Nodos personalizados locales
```

> **Importante:** Nunca modifiques manualmente los archivos en `pg-data/`. Usa la interfaz de QANode para gestionar tus datos.

---


## Solución de Problemas

### La versión desktop no inicia

- Verifica que no haya otra instancia de QANode en ejecución
- Intenta ejecutar como administrador
- Verifica que el puerto 5432 no esté en uso por otro PostgreSQL

---

## Próximos Pasos

- [Inicio Rápido](inicio-rapido.md) — Crea tu primera prueba
- [Conceptos Fundamentales](conceptos.md) — Comprende la estructura de la plataforma
