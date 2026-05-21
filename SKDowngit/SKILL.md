# Generación de Enlaces DownGit
Guía y especificación para la creación de enlaces de descarga directa de carpetas y archivos específicos de repositorios de GitHub usando DownGit.

## Guía de Ejecución Interactiva (Instrucciones para el Agente)

Cuando un desarrollador te solicite utilizar esta habilidad en su espacio de trabajo, debes detenerte y realizar las siguientes preguntas interactivas en español para configurar la implementación adecuada:

1. **Stack Tecnológico y Framework**: ¿Cuál es el stack de tu proyecto actual (ej. Next.js/React, Vanilla HTML/CSS/JS, Node.js, Python)?
2. **Ubicación de la Implementación**: ¿En qué archivo, componente o ruta del proyecto deseas integrar la descarga con DownGit (ej. en un componente de botón específico, una función de utilidad compartida, etc.)?
3. **Repositorio o Ruta de Origen**: ¿Cuál es la URL pública del repositorio de GitHub que deseas configurar?
4. **Alcance de la Descarga**: ¿Deseas descargar el repositorio completo o solo un archivo o carpeta en concreto dentro de él? (En caso de ser una subcarpeta/archivo, especifica su ruta).
5. **Estructura del archivo ZIP**:
   - ¿Deseas mantener la carpeta raíz original en el ZIP (por defecto)?
   - ¿Prefieres omitir la carpeta raíz (los archivos directamente dentro del ZIP)?
   - ¿Quieres renombrar la carpeta raíz a un nombre personalizado?
6. **Nombre del Archivo ZIP**: ¿El nombre del archivo ZIP debe ser dinámico (tomado del título de la habilidad o del repositorio) o fijo?

Una vez recopiladas las respuestas, analiza el archivo destino y aplica los cambios necesarios utilizando las utilidades de DownGit descritas en esta especificación.

## Introducción a DownGit

DownGit es una herramienta web de código abierto que soluciona una limitación clásica de GitHub: la imposibilidad de descargar una carpeta específica o un archivo individual de forma directa sin tener que clonar o descargar todo el repositorio.

## Cómo funciona internamente

Cuando un usuario accede a un enlace de DownGit, la aplicación realiza las siguientes tareas en el lado del cliente (navegador):
1. **Análisis de la URL de GitHub**: Extrae el propietario, el nombre del repositorio, la rama y la ruta específica de la carpeta o archivo.
2. **Llamadas a la API de GitHub**: Utiliza la API pública de GitHub para navegar por el árbol de directorios del repositorio (`/repos/{owner}/{repo}/git/trees/{sha}`) u obtener el contenido directo de los archivos.
3. **Descarga en memoria**: Descarga de forma asíncrona todos los archivos dentro de la ruta especificada.
4. **Compresión al vuelo**: Utiliza bibliotecas JavaScript (como JSZip) para empaquetar los archivos en un archivo comprimido `.zip` directamente en el navegador del usuario.
5. **Descarga final**: Genera un enlace blob local para que el usuario descargue el archivo comprimido resultante.

---

## Estructura de Enlaces de DownGit

Los enlaces se componen de una dirección base y parámetros de consulta (query parameters) específicos:

```
https://downgit.github.io/#/home?url=<LINK_DE_GITHUB>&fileName=<NOMBRE_ARCHIVO>&rootDirectory=<OPCION_RAIZ>
```

### Parámetros Disponibles

| Parámetro | Tipo / Opciones | Descripción |
| :--- | :--- | :--- |
| `url` | **Requerido** (URL) | Enlace directo al directorio o archivo en GitHub. Puede apuntar a cualquier rama (ej. `master`, `main`) o commit específico. |
| `fileName` | Opcional (Texto) | Nombre que se le asignará al archivo `.zip` resultante. No requiere la extensión `.zip`. Por defecto, usa el nombre de la carpeta/archivo original. |
| `rootDirectory` | Opcional (`true` / `false` / Texto) | Controla el comportamiento del directorio raíz dentro del archivo `.zip` (ver detalles abajo). |

---

## Modos de Configuración del Directorio Raíz (`rootDirectory`)

El parámetro `rootDirectory` permite personalizar la estructura interna del archivo comprimido:

### 1. Directorio raíz por defecto (`rootDirectory=true` o no especificado)
Crea una carpeta envolvente dentro del archivo ZIP con el nombre original de la carpeta descargada.
* **URL de GitHub**: `https://github.com/MinhasKamal/DownGit/tree/master/res/images`
* **Enlace DownGit**: `https://downgit.github.io/#/home?url=https://github.com/MinhasKamal/DownGit/tree/master/res/images`
* **Estructura del ZIP**: 
  ```
  images.zip
  └── images/
      ├── downgit.png
      └── ...
  ```

### 2. Omitir el directorio raíz (`rootDirectory=false`)
Evita la creación de la carpeta contenedora. Los archivos internos del directorio se encuentran directamente en la raíz del archivo ZIP.
* **Enlace DownGit**: `https://downgit.github.io/#/home?url=https://github.com/MinhasKamal/DownGit/tree/master/res/images&rootDirectory=false`
* **Estructura del ZIP**:
  ```
  images.zip
  ├── downgit.png
  └── ...
  ```

### 3. Personalizar el nombre de la raíz (`rootDirectory=NombrePersonalizado`)
Renombra la carpeta contenedora dentro del archivo ZIP al valor especificado.
* **Enlace DownGit**: `https://downgit.github.io/#/home?url=https://github.com/MinhasKamal/DownGit/tree/master/res/images&rootDirectory=ImagesOfDownGit`
* **Estructura del ZIP**:
  ```
  images.zip
  └── ImagesOfDownGit/
      ├── downgit.png
      └── ...
  ```

---

## Ejemplos Prácticos

### Descarga de una carpeta con nombre de archivo personalizado
Si quieres descargar la carpeta de imágenes de DownGit con el nombre de archivo `DownGit-Images.zip` y que la carpeta raíz interna se llame `ImagesOfDownGit`:

* **Enlace generado**:
  ```
  https://downgit.github.io/#/home?url=https://github.com/MinhasKamal/DownGit/tree/master/res/images&fileName=DownGit-Images&rootDirectory=ImagesOfDownGit
  ```

### Descarga de un archivo individual con nombre personalizado
Si deseas descargar el archivo `downgit.png` directamente como un archivo comprimido llamado `DownGitIcon.zip`:

* **Enlace generado**:
  ```
  https://downgit.github.io/#/home?url=https://github.com/MinhasKamal/DownGit/blob/master/res/images/downgit.png&fileName=DownGitIcon
  ```

---

## Integración y Automatización

Para implementar esto en scripts o interfaces de usuario, se puede utilizar la siguiente lógica en JavaScript:

```javascript
/**
 * Genera un enlace de DownGit para un recurso de GitHub.
 * @param {string} githubUrl - La URL de GitHub (carpeta o archivo).
 * @param {object} options - Opciones de personalización.
 * @param {string} [options.fileName] - Nombre personalizado para el archivo zip.
 * @param {boolean|string} [options.rootDirectory] - Comportamiento de la carpeta raíz.
 * @returns {string} Enlace de DownGit formateado.
 */
function generateDownGitLink(githubUrl, options = {}) {
  const baseUrl = "https://downgit.github.io/#/home";
  const params = new URLSearchParams();
  
  params.append("url", githubUrl);
  
  if (options.fileName) {
    params.append("fileName", options.fileName);
  }
  
  if (options.rootDirectory !== undefined) {
    params.append("rootDirectory", String(options.rootDirectory));
  }
  
  return `${baseUrl}?${params.toString()}`;
}
```
