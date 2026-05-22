---
name: sk-seo-optimizer
description: Guía y flujo de trabajo avanzado para optimizar el SEO en aplicaciones Next.js App Router, incluyendo metadatos de servidor, JSON-LD estructurado, sitemaps dinámicos y configuración de robots.
---

# Next.js SEO Optimization Skill
Guía y flujo de trabajo avanzado para optimizar el SEO en aplicaciones Next.js App Router, solucionando conflictos de metadatos en componentes de cliente, implementando SEO dinámico, datos estructurados (JSON-LD), sitemaps y directivas de robots.

## Overview
Esta skill describe el proceso técnico y flujo de diseño para implementar optimizaciones de SEO premium en aplicaciones Next.js que utilizan App Router, resolviendo incompatibilidades entre componentes interactivos de cliente (`"use client"`) y la exportación de metadatos, estructurando datos para motores de búsqueda y configurando sitemaps automatizados.

## Dependencies
- Next.js 13+ (App Router)
- React 18+
- TypeScript (Recomendado)

## Quick Start
1. Inspecciona si las páginas de tu aplicación contienen la directiva `"use client"` al inicio y si heredan metadatos globales genéricos en lugar de tener metadatos dedicados.
2. Identifica si tienes rutas dinámicas (por ejemplo, `[id]`) que necesiten SEO dinámico.
3. Sigue el flujo detallado a continuación para reestructurar el sitio, añadir metadatos asíncronos, configurar datos estructurados JSON-LD, canonicals, sitemaps y robots.

---

## Workflow

### 1. Auditoría e Identificación de Problemas
- **Idioma y Localización**: Comprueba si el idioma declarado en `layout.tsx` (`<html lang="es">` o el correspondiente) coincide con el idioma principal del contenido de la web.
- **Páginas Clientes con "use client"**: Identifica las páginas que inician con `"use client"` debido a que usan React hooks (`useState`, `useEffect`, context, etc.). Estas páginas no pueden exportar `metadata` de forma directa.
- **Rutas Dinámicas**: Comprueba si las páginas dinámicas cargan datos desde una API o base de datos y carecen de previsualización en redes sociales.
- **Jerarquía Semántica (H1 Único)**: Asegura que cada página HTML contenga exactamente una etiqueta `<h1>`. Si el diseño visual no requiere un H1 visible o usa estilos alternativos, utiliza un encabezado oculto para accesibilidad y rastreadores utilizando clases como `.srOnly` (CSS `position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0, 0, 0, 0); border: 0;`).

### 2. Arquitectura de Separación Servidor/Cliente
Para mantener la interactividad del lado del cliente y aun así permitir que Next.js renderice los metadatos desde el servidor para los rastreadores SEO:
- Extrae la lógica visual e interactiva de `page.tsx` a un nuevo archivo en el mismo directorio llamado `[PageName]Client.tsx` (ejemplo: `HomeClient.tsx`, `BlogClient.tsx`, `ComunidadClient.tsx`).
- Define `"use client"` en la cabecera de este nuevo componente cliente.
- Reemplaza el archivo `page.tsx` original convirtiéndolo en un Server Component.
- Importa el componente cliente y exporta el objeto `metadata` de forma estática.

*Ejemplo de `page.tsx` (Server Component):*
```tsx
import ComunidadClient from "./ComunidadClient";
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Comunidad de Desarrolladores de Agentes de IA | NombreSitio",
  description: "Únete a la comunidad de creadores de Agent Skills. Comparte código, debate arquitecturas de agentes autónomos y colabora en proyectos.",
  alternates: {
    canonical: "/comunidad",
  },
  openGraph: {
    title: "Comunidad de Desarrolladores de Agentes de IA | NombreSitio",
    description: "Únete a la comunidad de creadores de Agent Skills. Comparte código, debate arquitecturas de agentes autónomos y colabora en proyectos.",
    type: "website",
    url: "https://tusitio.com/comunidad",
  },
  twitter: {
    card: "summary_large_image",
    title: "Comunidad de Desarrolladores de Agentes de IA | NombreSitio",
    description: "Únete a la comunidad de creadores de Agent Skills. Comparte código, debate arquitecturas de agentes autónomos y colabora en proyectos.",
  },
};

export default function Page() {
  return <ComunidadClient />;
}
```

### 3. Reglas de Metadatos y Enlaces Canónicos
- **Límites de Caracteres**: 
  - El título (`title`) debe mantenerse por debajo de **60 caracteres** para evitar que se recorte en las SERPs (páginas de resultados de búsqueda).
  - La descripción (`description`) debe mantenerse por debajo de **155 caracteres** y contener palabras clave relevantes sin saturarlas artificialmente.
- **Enlaces Canónicos (`alternates.canonical`)**: Añade siempre la ruta canonical relativa o absoluta correspondiente para evitar la indexación de páginas duplicadas (causadas por parámetros de búsqueda, barras inclinadas finales o alias de dominio).

### 4. Datos Estructurados JSON-LD
Los datos estructurados ayudan a los motores de búsqueda a entender el contexto de tu contenido y habilitan Rich Snippets en Google.
- Inyecta JSON-LD usando la etiqueta `<script type="application/ld+json">` dentro de tus Server Components.
- Utiliza la estructura `@graph` para consolidar todas las entidades (como `Organization`, `WebSite`, `WebPage`, `Blog` o `BlogPosting`) de forma interconectada mediante identificadores únicos (`@id`).

*Ejemplo de integración de datos estructurados:*
```tsx
export default function Page() {
  const jsonLd = {
    "@context": "https://schema.org",
    "@graph": [
      {
        "@type": "WebPage",
        "@id": "https://tusitio.com/comunidad#webpage",
        "url": "https://tusitio.com/comunidad",
        "name": "Comunidad de Desarrolladores de Agentes de IA",
        "isPartOf": {
          "@id": "https://tusitio.com/#website"
        },
        "description": "Únete a la comunidad de creadores de Agent Skills."
      }
    ]
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <ComunidadClient />
    </>
  );
}
```

### 5. Configuración de Sitemap y Robots
Para garantizar que Google y otros rastreadores descubran y prioricen adecuadamente todas tus páginas clave, utiliza las capacidades integradas de Next.js App Router para generar archivos `sitemap.xml` y `robots.txt`.

- **Sitemap Dinámico (`src/app/sitemap.ts`)**:
  ```typescript
  import { MetadataRoute } from "next";

  export default function sitemap(): MetadataRoute.Sitemap {
    const baseUrl = "https://tusitio.com";
    return [
      {
        url: baseUrl,
        lastModified: new Date(),
        changeFrequency: "daily",
        priority: 1.0,
      },
      {
        url: `${baseUrl}/blog`,
        lastModified: new Date(),
        changeFrequency: "weekly",
        priority: 0.8,
      },
      {
        url: `${baseUrl}/changelog`,
        lastModified: new Date(),
        changeFrequency: "weekly",
        priority: 0.8,
      },
      {
        url: `${baseUrl}/comunidad`,
        lastModified: new Date(),
        changeFrequency: "weekly",
        priority: 0.8,
      },
    ];
  }
  ```

- **Robots.txt (`src/app/robots.ts`)**:
  ```typescript
  import { MetadataRoute } from "next";

  export default function robots(): MetadataRoute.Robots {
    const baseUrl = "https://tusitio.com";
    return {
      rules: {
        userAgent: "*",
        allow: "/",
        disallow: ["/admin/", "/api/"],
      },
      sitemap: `${baseUrl}/sitemap.xml`,
    };
  }
  ```

### 6. Implementación de SEO Dinámico (`generateMetadata`)
En rutas dinámicas como `[id]/page.tsx`, implementa la función asíncrona `generateMetadata` de Next.js.
- Crea un helper de fetching de datos (ejemplo: `getSkill(id)`) que pueda ser compartido tanto por `generateMetadata` como por el componente principal de la página.
- Ejecuta la consulta de base de datos o API en `generateMetadata` para extraer los campos `title` y `description` específicos de cada elemento.

*Ejemplo en `[id]/page.tsx`:*
```tsx
import type { Metadata } from "next";

async function getSkill(id: string) {
  // fetch asíncrono de base de datos
  const data = await db.fetchSkill(id);
  return data;
}

export async function generateMetadata({ params }: { params: Promise<{ id: string }> }): Promise<Metadata> {
  const { id } = await params;
  const skill = await getSkill(id);

  if (!skill) {
    return {
      title: "Elemento No Encontrado",
      description: "El recurso solicitado no está disponible.",
    };
  }

  return {
    title: `${skill.title} | Plataforma`,
    description: skill.desc,
    openGraph: {
      title: `${skill.title} | Plataforma`,
      description: skill.desc,
      type: "article",
      images: [{ url: "/Logo/SC_logo.png" }]
    }
  };
}

export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const skill = await getSkill(id);
  // ... renderizado normal
}
```

### 7. Proceso de Indexación Guiada y Registro en Google Search Console
Una vez completadas todas las configuraciones técnicas del lado del servidor, el último paso obligatorio consiste en guiar al buscador para que descubra e indexe la aplicación de manera acelerada:

1. **Alineación de Dominios**: Asegura que las propiedades `metadataBase`, sitemaps, robots.txt y canonicals coincidan exactamente con la URL de despliegue de producción activa (ej. `https://tusitio.vercel.app` para entornos de pruebas en Vercel, o el dominio definitivo `https://tusitio.com`).
2. **Crear la Propiedad**: Accede a [Google Search Console](https://search.google.com/search-console) y añade una propiedad de tipo **Prefijo de la URL** con el enlace exacto del despliegue activo.
3. **Verificar Propiedad**:
   - Selecciona el método de verificación por **Archivo HTML**.
   - Descarga el archivo de verificación (normalmente llamado `google[codigo].html`).
   - Crea un archivo con ese nombre exacto dentro del directorio `public/` del proyecto de frontend. El contenido interno del archivo debe ser la línea proporcionada (ej: `google-site-verification: google[codigo].html`).
   - Realiza un `git commit` y `git push` para desplegar el archivo en el servidor.
   - Confirma que el archivo sea accesible públicamente desde el navegador en `https://tusitio.vercel.app/google[codigo].html` y pulsa **Verificar** en la Search Console.
4. **Enviar el Mapa del Sitio**: En el apartado lateral de **Sitemaps**, ingresa la ruta `sitemap.xml` y haz clic en **Enviar** para que Google conozca todas las rutas válidas.
5. **Inspección e Indexación Manual**:
   - Utiliza la herramienta de inspección de URLs en la barra de búsqueda superior de la Search Console para evaluar la URL raíz (`https://tusitio.vercel.app/`).
   - Haz clic en **Solicitar indexación** para forzar a Googlebot a procesar la página inmediatamente en lugar de esperar al rastreo periódico convencional.

---

## Cómo Funciona
Next.js procesa los Server Components en el servidor y extrae las definiciones de `metadata` o el resultado de `generateMetadata` antes de enviar la respuesta HTML inicial. El motor de búsqueda (o rastreadores de Discord, Slack y Twitter) recibe un archivo HTML plano estructurado con etiquetas `<title>`, `<meta name="description">` y propiedades `<meta property="og:*">` completamente formadas.
Esto elimina la necesidad de depender de técnicas de renderizado en el cliente (Client-Side Rendering) que a menudo resultan en previsualizaciones vacías o indexaciones fallidas en bots que no ejecutan JavaScript de forma avanzada.

---

## Para Quiénes Funciona
- **Desarrolladores Next.js**: Que buscan estructurar correctamente la indexación utilizando la API oficial de metadatos de Next.js en arquitecturas de App Router.
- **Sitios Web Interactivos (SPAs híbridas)**: Que necesitan conservar el estado de React (`"use client"`) en la interfaz de usuario pero no quieren sacrificar el SEO individual de cada sección.
- **Plataformas Dinámicas**: Repositorios de código, blogs, tiendas online y foros que generan cientos de páginas basadas en IDs o slugs únicos, garantizando que cada enlace compartido contenga su previsualización enriquecida de forma automática.

---

## Errores Comunes
- **Declarar "use client" y metadatos en el mismo archivo**: Provocará un error en el compilador de Next.js indicando que las exportaciones de metadatos solo están permitidas en Server Components.
- **Ignorar el parámetro asíncrono de params**: En versiones recientes de Next.js, `params` debe ser esperado asíncronamente (`await params`), de lo contrario fallará en el runtime.
- **Olvidar metadatos Open Graph específicos para Twitter**: Algunas plataformas dependen exclusivamente de etiquetas og:* mientras que otras requieren twitter:card para renderizar previsualizaciones completas.
- **No definir una `metadataBase`**: Causa que las imágenes con rutas relativas fallen en resolverse a URLs completas absolutas, arruinando la previsualización en redes sociales y servicios de mensajería.
