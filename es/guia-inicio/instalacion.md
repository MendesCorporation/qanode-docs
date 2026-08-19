# Instalación


---

## Versión Desktop

La versión desktop es la forma más fácil de comenzar. Incluye todos los componentes necesarios en un único instalador, incluyendo una base de datos PostgreSQL integrada.

### Requisitos

- **Windows** 10/11, 64 bits
- **macOS** 11 o posterior, Intel o Apple Silicon
- **Linux** 64 bits con soporte para AppImage
- **Memoria RAM**: 8 GB mínimo (16 GB recomendado)
- **Espacio en Disco**: 2 GB para la instalación, además del espacio utilizado por evidencias y ejecuciones

### Instalación en Windows

1. Descarga el instalador `QANode Setup.exe` en el sitio https://www.qanode.com
2. Ejecuta el instalador y sigue las instrucciones
3. Al finalizar, QANode se abrirá automáticamente

[Instalador de QANode en Windows](../../assets/images/instalacao-nsis.png)
*Imagen: Pantalla del instalador NSIS de QANode en Windows*

### Instalación en macOS

1. Descargue el archivo `.dmg` correspondiente a su procesador: Apple Silicon (`arm64`) o Intel (`x64`)
2. Abra el DMG y mueva QANode a **Aplicaciones**
3. Abra QANode desde la carpeta **Aplicaciones** o la búsqueda de macOS

La aplicación está firmada y notarizada por Apple. Después de una actualización, macOS puede tardar unos instantes en actualizar el icono y el índice de búsqueda.

### Instalación en Linux

1. Descargue el archivo `.AppImage`
2. Marque el archivo como ejecutable
3. Abra el AppImage

Ejemplo:

```bash
chmod +x QANode-Setup-x64.AppImage
./QANode-Setup-x64.AppImage
```

En el primer inicio, QANode:
- Inicializará la base de datos PostgreSQL integrada
- Creará las tablas necesarias (migraciones automáticas)
- Iniciará el servidor de la API
- Abrirá la interfaz de la aplicación

> **Nota:** El primer inicio puede tardar algunos segundos más mientras se prepara la base de datos. Se mostrará una pantalla de splash durante este proceso.

### Carpeta de Datos

La versión desktop almacena la base de datos, evidencias, archivos y logs en la carpeta de datos del usuario:

```text
Windows: %APPDATA%\@qanode\desktop\
macOS:   ~/Library/Application Support/@qanode/desktop/
Linux:   ~/.config/@qanode/desktop/
```

Dentro de ella, `postgres/` contiene la base de datos integrada, `storage/` guarda los artefactos y `desktop-main.log` registra el inicio de la aplicación.

> **Importante:** Nunca modifique ni elimine manualmente la carpeta `postgres/`. Antes de cualquier procedimiento de recuperación, haga una copia de la carpeta de datos completa.

---


## Solución de Problemas

### La versión desktop no inicia

- Verifique que no haya otra instancia de QANode en ejecución
- Abra `desktop-main.log` en la carpeta de datos y busque `Fatal startup error`
- En Windows, confirme que Microsoft Visual C++ Redistributable x64 esté actualizado
- No elimine la carpeta `postgres/`; preserve la base de datos y envíe el log al soporte para su diagnóstico

---

## Próximos Pasos

- [Inicio Rápido](inicio-rapido.md) — Crea tu primera prueba
- [Conceptos Fundamentales](conceptos.md) — Comprende la estructura de la plataforma
