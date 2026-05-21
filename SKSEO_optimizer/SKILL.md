---
name: sk-seo-optimizer
description: >-
  Guia y flujo de trabajo para optimizar el SEO en aplicaciones Next.js App Router, solucionando conflictos de metadatos en componentes de cliente e implementando SEO dinamico.
---

# Next.js SEO Optimization Skill

## Overview
Esta skill describe el proceso tecnico y flujo de diseño para implementar optimizaciones de SEO premium en aplicaciones Next.js que utilizan App Router, resolviendo incompatibilidades entre componentes interactivos de cliente (`"use client"`) y la exportacion de metadatos estaticos y dinamicos en el servidor.

## Dependencies
- Next.js 13+ (App Router)
- React 18+
- TypeScript (Recomendado)

## Quick Start
Para aplicar esta skill:
1. Inspecciona si las paginas de tu aplicacion contienen la directiva `"use client"` al inicio y si heredan metadatos globales genericos en lugar de tener metadatos dedicados.
2. Identifica si tienes rutas dinamicas (por ejemplo, `[id]`) que necesiten SEO dinamico.
3. Sigue el flujo detallado a continuacion para reestructurar el sitio y añadir metadatos asincronos.

---

## Workflow

### 1. Auditoria e Identificacion de Problemas
- **Idioma y Localizacion**: Comprueba si el idioma declarado en `layout.tsx` (`<html lang="en">`) coincide con el idioma principal del contenido de la web.
- **Paginas Clientes con "use client"**: Identifica las paginas que inician con `"use client"` debido a que usan React hooks (`useState`, `useEffect`, context, etc.). Estas paginas no pueden exportar `metadata` de forma directa.
- **Rutas Dinamicas**: Comprueba si paginas como articulos de blog o fichas tecnicas cargan datos desde una API o base de datos y carecen de previsualizacion en redes sociales.

### 2. Arquitectura de Separacion Servidor/Cliente
Para mantener la interactividad del lado del cliente y aun asi permitir que Next.js renderice los metadatos desde el servidor para los rastreadores SEO:
- Extrae la logica visual e interactiva de `page.tsx` a un nuevo archivo en el mismo directorio llamado `[PageName]Client.tsx` (ejemplo: `HomeClient.tsx`, `BlogClient.tsx`, `ComunidadClient.tsx`).
- Define `"use client"` en la cabecera de este nuevo componente cliente.
- Reemplaza el archivo `page.tsx` original convirtiendolo en un Server Component.
- Importa el componente cliente y exporta el objeto `metadata` de forma estatica.

*Ejemplo de `page.tsx` (Server Component):*
```tsx
import ComunidadClient from "./ComunidadClient";
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Comunidad",
  description: "Únete a la red global de desarrolladores. Comparte, colabora y debate sobre el futuro de los agentes autonomos.",
  openGraph: {
    title: "Comunidad | Plataforma",
    description: "Únete a la red global de desarrolladores de la plataforma.",
    type: "website",
  },
};

export default function Page() {
  return <ComunidadClient />;
}
```

### 3. Configuracion de Metadatos Globales (`layout.tsx`)
Actualiza el layout raiz para proveer un sistema robusto de metadatos globales:
- **`metadataBase`**: Configura una URL de origen para evitar errores al resolver rutas relativas de imagenes de Open Graph o Twitter.
  ```typescript
  metadataBase: new URL(process.env.NEXT_PUBLIC_SITE_URL || "https://tusitio.com"),
  ```
- **Plantilla de Titulo**: Usa un objeto de titulo con `default` y `template` para que las subpaginas añadan sufijos automaticamente.
  ```typescript
  title: {
    default: "NombreSitio - Descripcion Corta",
    template: "%s | NombreSitio",
  }
  ```
- **Open Graph y Twitter**: Declara imagenes de marca, nombres de sitio, localizaciones y configuraciones de rastreadores (`robots`).

### 4. Implementacion de SEO Dinamico (`generateMetadata`)
En rutas dinamicas como `[id]/page.tsx`, implementa la funcion asincrona `generateMetadata` de Next.js.
- Crea un helper de fetching de datos (ejemplo: `getSkill(id)`) que pueda ser compartido tanto por `generateMetadata` como por el componente principal de la pagina.
- Ejecuta la consulta de base de datos o API en `generateMetadata` para extraer los campos `title` y `description` especificos de cada elemento.

*Ejemplo en `[id]/page.tsx`:*
```tsx
import type { Metadata } from "next";

async function getSkill(id: string) {
  // fetch asincrono de base de datos
  const data = await db.fetchSkill(id);
  return data;
}

export async function generateMetadata({ params }: { params: Promise<{ id: string }> }): Promise<Metadata> {
  const { id } = await params;
  const skill = await getSkill(id);

  if (!skill) {
    return {
      title: "Elemento No Encontrado",
      description: "El recurso solicitado no esta disponible.",
    };
  }

  return {
    title: skill.title,
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

### 5. Compilacion y Verificacion
- Ejecuta `npm run build` o `next build`.
- Verifica que Next.js no emita advertencias relativas a la falta de `metadataBase`.
- Asegura que las paginas dinamicas se clasifiquen como `ƒ (Dynamic)` o `○ (Static)` segun corresponda sin fallas en el build de produccion.

---

## Como Funciona
Next.js procesa los Server Components en el servidor y extrae las definiciones de `metadata` o el resultado de `generateMetadata` antes de enviar la respuesta HTTP inicial. El motor de busqueda (o rastreadores de Discord, Slack y Twitter) recibe un archivo HTML plano estructurado con etiquetas `<title>`, `<meta name="description">` y propiedades `<meta property="og:*">` completamente formadas.
Esto elimina la necesidad de depender de tecnicas de renderizado en el cliente (Client-Side Rendering) que a menudo resultan en previsualizaciones vacias o indexaciones fallidas en bots que no ejecutan JavaScript de forma avanzada.

---

## Para Quienes Funciona
- **Desarrolladores Next.js**: Que buscan estructurar correctamente la indexacion utilizando la API oficial de metadatos de Next.js en arquitecturas de App Router.
- **Sitios Web Interactivos (SPAs hibridas)**: Que necesitan conservar el estado de React (`"use client"`) en la interfaz de usuario pero no quieren sacrificar el SEO individual de cada seccion.
- **Plataformas Dinamicas**: Repositorios de codigo, blogs, tiendas online y foros que generan cientos de paginas basadas en IDs o slugs unicos, garantizando que cada enlace compartido contenga su previsualizacion enriquecida de forma automatica.

---

## Errores Comunes
- **Declarar "use client" y metadatos en el mismo archivo**: Provocara un error en el compilador de Next.js indicando que las exportaciones de metadatos solo estan permitidas en Server Components.
- **Ignorar el parametro asincrono de params**: En versiones recientes de Next.js, `params` debe ser esperado asincronamente (`await params`), de lo contrario fallara en el runtime.
- **Olvidar metadatos Open Graph especificos para Twitter**: Algunas plataformas dependen exclusivamente de etiquetas og:* mientras que otras requieren twitter:card para renderizar previsualizaciones completas.
