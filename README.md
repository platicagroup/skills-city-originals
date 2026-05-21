# SkillsCity Originals - Directorio de Habilidades (Skills)

Este repositorio contiene la colección de Habilidades (Skills) oficiales utilizadas por los agentes de desarrollo y programación en la plataforma de SkillsCity. Cada habilidad está diseñada para guiar el flujo de desarrollo, documentar patrones de arquitectura óptimos y servir como referencia directa de código limpio y especificaciones técnicas.

## Contenido del Repositorio

El repositorio está organizado en subdirectorios temáticos para cada habilidad:

### 1. SKSEO_optimizer
* **Nombre de la habilidad**: `sk-seo-optimizer`
* **Descripción**: Flujo de trabajo para optimizar el SEO en aplicaciones Next.js App Router, solucionando conflictos de metadatos en componentes de cliente e implementando SEO dinámico.
* **Componentes clave**:
  - Arquitectura de separación Servidor/Cliente (separación de páginas servidor y cliente).
  - Implementación de metadatos dinámicos asíncronos mediante `generateMetadata` en rutas dinámicas.
  - Configuración de metadatos globales en `layout.tsx`.

### 2. SKDowngit
* **Nombre de la habilidad**: `sk-downgit`
* **Descripción**: Guía y especificación para la generación de enlaces de descarga directa de subcarpetas y archivos individuales de repositorios de GitHub usando DownGit.
* **Componentes clave**:
  - Estructuración de parámetros URL (`url`, `fileName`, `rootDirectory`).
  - Lógica y script para la generación dinámica de enlaces.

### 3. SKLazy_Loading
* **Nombre de la habilidad**: `sk-lazy-loading`
* **Descripción**: Patrones de carga progresiva e hidratación en segundo plano (Lazy Loading) para mejorar el rendimiento percibido en grillas o listados de datos masivos.
* **Componentes clave**:
  - Carga inicial rápida de alta prioridad (rango acotado).
  - Descarga asíncrona del resto del catálogo en segundo plano.
  - Prevención de fugas de memoria mediante control del ciclo de vida del componente (`isMounted`).

### 4. SKModals V1
* **Nombre de la habilidad**: `sk-modals-v1`
* **Descripción**: Diseño e implementación de modales de estado (éxito, error y fallo) con estética premium de terminal retro y glassmorphism.
* **Componentes clave**:
  - Plantillas React configurables con estilos en CSS Modules.
  - Prevención de scroll de fondo (`overflow: hidden`).
  - Animaciones y sombras personalizadas según el estado.

### 5. SKResponsive
* **Nombre de la habilidad**: `sk-responsive`
* **Descripción**: Directrices y patrones de diseño adaptativo para convertir componentes diseñados para escritorio a resoluciones móviles.
* **Componentes clave**:
  - Diseño responsive usando CSS Modules y media queries para cabeceras, paginación y modales.

### 6. SKSupabase
* **Nombre de la habilidad**: `sk-supabase`
* **Descripción**: Configuración y resolución de problemas de seguridad en Supabase Auth y PostgreSQL.
* **Componentes clave**:
  - Habilitación de "Leaked Password Protection" contra HaveIBeenPwned.org.
  - Configuración del parámetro `search_path` en funciones con `SECURITY DEFINER` para evitar ataques de escalada de privilegios.
  - Revocación de permisos de ejecución para el grupo `PUBLIC`.

### 7. SKVercel
* **Nombre de la habilidad**: `sk-vercel-analytics`
* **Descripción**: Proceso para integrar Vercel Analytics en aplicaciones Next.js/React de manera nativa y con impacto mínimo en rendimiento.
* **Componentes clave**:
  - Integración del componente de analíticas en App Router y Pages Router.
  - Habilitación y diagnóstico de analíticas desde el Dashboard de Vercel.

---

## Cómo utilizar estas Habilidades

Cada directorio incluye un archivo `SKILL.md` estructurado que proporciona:
1. **Metadata**: Nombre e información de descripción del componente.
2. **Guía Interactiva**: Preguntas iniciales de alineación que debe formular el agente antes de modificar código.
3. **Flujo de Trabajo (Workflow)**: Instrucciones paso a paso.
4. **Plantillas de Código**: Bloques listos para integrar.
5. **Errores Comunes**: Advertencias para evitar bugs durante la implementación.
