---
name: sk-lazy-loading
description: >-
  Guidelines, templates, and patterns for implementing progressive and background loading patterns in Next.js/React grids.
---

# Skill de Carga Progresiva y Segundo Plano (Lazy Loading) - V1
Guidelines, templates, and patterns for implementing progressive and background loading patterns in Next.js/React grids.

## Resumen
Esta habilidad detalla las mejores prácticas para acelerar el tiempo de carga percibido en páginas web con grandes listados de datos provenientes de Supabase o bases de datos externas. Se enfoca en la carga inicial rápida (bloque prioritario) seguida de una hidratación progresiva en segundo plano (non-blocking).

## Dependencias
* React 18+ / Next.js
* Cliente de Supabase (`@supabase/supabase-js`) o APIs REST equivalentes

## Guía de Ejecución Interactiva (Instrucciones para el Agente)
Cuando se te asigne optimizar el rendimiento de carga de una grilla o listado, haz las siguientes preguntas interactivas antes de empezar:
1. **Tamaño del Catálogo**: ¿Cuántos registros se estiman en total a mediano/largo plazo en esta base de datos?
2. **Distribución de Páginas**: ¿Cuál es el tamaño de página (`pageSize`) en escritorio y móvil para ajustar el primer lote de carga?
3. **Complejidad del Buscador**: ¿Los usuarios realizan búsquedas exactas críticas al instante de cargar la página, o podemos depender de la hidratación progresiva en segundo plano?

---

## Flujo de Trabajo y Patrones de Diseño

### 1. Carga Inicial de Alta Prioridad
El principal cuello de botella de rendimiento es esperar a que una consulta asíncrona de base de datos retorne miles de filas antes de renderizar la UI.
* **Solución**: Solicita un rango de datos muy pequeño que sea suficiente para rellenar las primeras pantallas visibles (ej. de 3 a 5 páginas de la grilla).
* **Ejemplo**: Para un tamaño de página de 9 elementos, un rango inicial de `0` a `44` (45 elementos) es ideal.

### 2. Hidratación en Segundo Plano (Progressive Loading)
Una vez que el primer lote de alta prioridad ha sido devuelto y el estado de carga (`isLoading`) se desactiva, se inicia un proceso asíncrono secundario que descarga secuencialmente el resto del catálogo y lo concatena al estado global.
* **Búsqueda Reactiva**: Dado que los filtros de búsqueda y categoría se evalúan sobre el estado global reactivo de React (`skills`), la UI se actualizará automáticamente y sin parpadeos a medida que se descargan los datos en segundo plano.

### 3. Seguridad de Ciclo de Vida (Evitar fugas de memoria)
Si el usuario navega fuera de la página principal mientras el bucle de segundo plano sigue descargando páginas de Supabase, React emitirá advertencias de actualización de estado en componentes no montados.
* **Solución**: Mantener una variable bandera local de montaje (`isMounted = true`) que se configure a `false` en la función de limpieza del hook `useEffect`.

---

## Código de Referencia de Implementación

### Implementación en React (`page.tsx`)
```tsx
import { useEffect, useState } from "react";
import { supabase } from "./lib/supabase";

export default function ExplorationGrid() {
  const [items, setItems] = useState<any[]>([]);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    let isMounted = true;

    async function fetchItems() {
      try {
        // 1. Carga inicial rápida (primeras 5 páginas)
        const { data: initialData, error: initialError } = await supabase
          .from("items")
          .select("*")
          .range(0, 44); // Rango 0-44 para 45 elementos

        if (initialError) {
          console.error("Error en carga inicial:", initialError);
          return;
        }

        if (initialData && isMounted) {
          setItems(initialData);
        }

        // 2. Proceso de fondo para el resto de páginas
        let allItems = initialData ? [...initialData] : [];
        let from = 45;
        let to = 1044;
        let hasMore = true;

        const fetchRemaining = async () => {
          while (hasMore && isMounted) {
            const { data, error } = await supabase
              .from("items")
              .select("*")
              .range(from, to);

            if (error) {
              console.error("Error en carga en segundo plano:", error);
              break;
            }

            if (data && data.length > 0) {
              allItems = [...allItems, ...data];
              if (isMounted) {
                setItems([...allItems]);
              }
              if (data.length < 1000) {
                hasMore = false;
              } else {
                from += 1000;
                to += 1000;
              }
            } else {
              hasMore = false;
            }
          }
        };

        // Lanzar el proceso de fondo sin bloquear el hilo principal
        fetchRemaining();

      } catch (err) {
        console.error("Fallo general de carga:", err);
      }
    }

    async function loadData() {
      setIsLoading(true);
      await fetchItems();
      if (isMounted) {
        setIsLoading(false);
      }
    }

    loadData();

    // Limpieza de ciclo de vida
    return () => {
      isMounted = false;
    };
  }, []);

  return (
    <div>
      {isLoading ? (
        <SkeletonGrid />
      ) : (
        <Grid items={items} />
      )}
    </div>
  );
}
```

---

## Errores Comunes
1. **Bloquear la Interfaz Completa**: Colocar un `await` en la llamada del segundo plano (`fetchRemaining()`) dentro de la función de carga principal, lo cual anula el propósito de la optimización.
2. **Omisión de la Limpieza**: No utilizar la bandera de ciclo de vida (`isMounted`), lo que causa advertencias de consola y fugas de memoria en navegadores de gama baja.
3. **No Concatenar**: Sobrescribir el estado (`setItems(data)`) en el bucle secundario en lugar de concatenarlo con la información previamente descargada (`allItems = [...allItems, ...data]`).
