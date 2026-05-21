---
name: sk-vercel-analytics
description: >-
  Guidelines, templates, and procedures for integrating Vercel Analytics into a Next.js/React application.
---

# Skill de Integración de Vercel Analytics - V1
Guidelines, templates, and procedures for integrating Vercel Analytics into a Next.js/React application.

## Resumen
Esta habilidad especifica el proceso paso a paso para añadir y configurar Vercel Analytics en una aplicación web basada en Next.js (App Router), permitiendo recolectar métricas de visitas y rendimiento del sitio en tiempo real de manera nativa y con impacto de rendimiento mínimo.

## Dependencias
* Proyecto Next.js (versión 13+ con App Router de preferencia)
* Despliegue en la plataforma de Vercel (para visualización del dashboard)

## Guía de Ejecución Interactiva (Instrucciones para el Agente)
Cuando un desarrollador te pida integrar analíticas de Vercel en su proyecto, realiza las siguientes preguntas antes de comenzar:
1. **Router Utilizado**: ¿Estás utilizando la estructura de App Router (`src/app/layout.tsx`) o la estructura clásica de Pages Router (`src/pages/_app.tsx`)?
2. **Entorno de Redirección**: ¿El sitio web ya se encuentra vinculado a un proyecto en el dashboard de Vercel, o necesitas ayuda para vincularlo primero?
3. **Analítica de Eventos Personalizados**: ¿Deseas únicamente medir visitas a páginas (page views) de forma automática o te gustaría configurar el seguimiento de eventos personalizados (como clics en botones de compra o inicios de sesión)?

---

## Flujo de Trabajo

### Paso 1: Instalación de la Dependencia Oficial
Para comenzar, debes instalar el paquete `@vercel/analytics` desde el gestor de paquetes correspondiente en la raíz del proyecto de frontend.

```bash
npm i @vercel/analytics
```

### Paso 2: Integración en el Layout Raíz (App Router)
Importa el componente `<Analytics />` e insértalo dentro de la etiqueta `<body>` en tu archivo de diseño principal. Esto asegura que el script de seguimiento se cargue en todas las páginas de la aplicación.

* Archivo a modificar: `src/app/layout.tsx`

```tsx
import type { Metadata } from "next";
import { Analytics } from "@vercel/analytics/react";

export const metadata: Metadata = {
  title: "Tu Proyecto",
  description: "Descripción de tu proyecto",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="es">
      <body>
        {children}
        {/* Componente inyectado para rastreo automático de visitas */}
        <Analytics />
      </body>
    </html>
  );
}
```

### Paso 3: Integración en el Archivo de Aplicación (Pages Router - Alternativo)
Si el proyecto utiliza la estructura de páginas clásica de Next.js, el componente debe inyectarse en el punto de entrada global de la aplicación.

* Archivo a modificar: `src/pages/_app.tsx`

```tsx
import type { AppProps } from "next/app";
import { Analytics } from "@vercel/analytics/react";

export default function App({ Component, pageProps }: AppProps) {
  return (
    <>
      <Component {...pageProps} />
      <Analytics />
    </>
  );
}
```

### Paso 4: Despliegue y Verificación
1. Confirma los cambios locales en git y realiza un push a la rama principal (ej. `main` o `master`) para iniciar la compilación en Vercel.
2. Accede a tu panel en [Vercel Dashboard](https://vercel.com/dashboard).
3. Selecciona tu proyecto y dirígete a la pestaña **Analytics**.
4. Haz clic en **Enable** o activa la integración si aún no lo has hecho.
5. Visita la URL de despliegue en producción y navega por el sitio. Las analíticas comenzarán a registrarse tras unos 30 segundos.

---

## Errores Comunes
1. **Posicionamiento del Componente**: Renderizar `<Analytics />` fuera de las etiquetas `<body>` o `<html>` en el layout raíz, lo cual puede generar advertencias de hidratación de Next.js o problemas en la carga de la página.
2. **Duplicidad del Componente**: Añadir el componente `<Analytics />` en páginas individuales además del layout raíz, provocando duplicidad de datos en el conteo de visitas únicas.
3. **Bloqueadores de Contenido**: No ver datos de prueba en la consola o dashboard en el navegador local debido a extensiones AdBlockers o bloqueadores de cookies. Es recomendable probar en una ventana de incógnito o desactivar temporalmente las extensiones.
