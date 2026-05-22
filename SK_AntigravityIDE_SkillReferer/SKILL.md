---
name: antigravity-skill-referer
description: >-
  Registra y estructura cualquier carpeta de habilidad (con su SKILL.md) como un plugin válido en la ruta de plugins de Antigravity IDE.
---

# Registro de Habilidades en Antigravity IDE (antigravity-skill-referer)

Esta habilidad permite estructurar y mover de forma correcta cualquier carpeta local que contenga una habilidad (`SKILL.md`) hacia la ruta de plugins del IDE de Antigravity (`C:\Users\Admin\.gemini\config\plugins\`), asegurando que sea cargada y referenciada correctamente.

## Guía de Ejecución Interactiva (Instrucciones para el Agente)

Cuando el desarrollador invoque esta habilidad, **debes detenerte y realizar las siguientes preguntas interactivas en español**:

1. **Ruta origen**: ¿Cuál es la ruta absoluta de la carpeta de la habilidad que deseas registrar? (Ej. `c:\Users\Admin\SkillsCity\SC_Originals\SKDowngit`).
2. **Nombre del Plugin**: ¿Qué nombre descriptivo deseas darle al plugin? (Por defecto se usará el nombre del directorio origen convertido a minúsculas y separado por guiones).
3. **Versión y Descripción**: ¿Deseas especificar una versión (por defecto `1.0.0`) y descripción personalizadas para el plugin?

Una vez obtenida la información, el agente debe proceder a estructurar y migrar la habilidad siguiendo el proceso técnico descrito a continuación.

---

## Proceso de Migración y Estructuración

Para registrar una habilidad, debes estructurarla como un plugin válido en `C:\Users\Admin\.gemini\config\plugins\<nombre-plugin>\` siguiendo estos pasos:

### 1. Definir Estructura de Destino
La estructura correcta de un plugin en Antigravity IDE debe ser:
```
C:\Users\Admin\.gemini\config\plugins\<nombre-plugin>\
├── plugin.json
└── skills/
    ├── SKILL.md
    └── (otros scripts/archivos de soporte si existieran)
```

### 2. Crear archivo `plugin.json`
En la raíz del nuevo directorio del plugin, crea el archivo `plugin.json` con la siguiente estructura:
```json
{
  "name": "<nombre-plugin>",
  "version": "<version>",
  "description": "<descripcion>",
  "author": {
    "name": "User"
  },
  "license": "MIT"
}
```

### 3. Copiar Habilidades y Archivos de Soporte
- Copia el archivo `SKILL.md` de la carpeta origen a la subcarpeta `skills/` del plugin.
- Si existen archivos adicionales relevantes (ej. scripts de ejecución, referencias, código fuente de herramientas) en la carpeta origen, cópialos en la estructura de destino manteniendo su ubicación relativa respecto a `SKILL.md`.

### 4. Limpieza (Opcional)
Pregunta al usuario si desea conservar la carpeta origen original o si prefiere eliminarla/limpiarla una vez completada la migración de forma exitosa.

---

## Automatización en PowerShell (Comandos de Referencia)

El agente puede realizar estas tareas mediante comandos de consola de PowerShell.

### Ejemplo de Migración:

```powershell
# 1. Definir variables
$SourcePath = "Ruta\Origen"
$PluginName = "nombre-plugin"
$DestPath = "C:\Users\Admin\.gemini\config\plugins\$PluginName"

# 2. Crear directorios de destino
New-Item -ItemType Directory -Force -Path "$DestPath\skills"

# 3. Copiar archivos recursivamente
Copy-Item -Path "$SourcePath\*" -Destination "$DestPath\skills" -Recurse -Force

# 4. Mover plugin.json si se creó en skills a la raíz, o crearlo directamente en la raíz:
# (Crear plugin.json directamente en $DestPath\plugin.json)
```
