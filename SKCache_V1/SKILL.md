---
name: sk-caching
description: >-
  Guidelines, templates, and patterns for implementing SWR caching (sessionStorage/localStorage) to speed up initial loads on Next.js/React tables.
---

# Skill de Caching y Estrategia SWR - V1
Guidelines, templates, and patterns for implementing SWR caching (sessionStorage/localStorage) to speed up initial loads on Next.js/React tables.

## Resumen
Esta habilidad detalla el uso del patrón Stale-While-Revalidate (SWR) para renderizar instantáneamente datos cacheados desde sessionStorage o localStorage al cargar la página, seguido de una recarga en segundo plano desde la base de datos para asegurar frescura y actualizar la caché.

## Dependencias
* React 18+ / Next.js
* Cliente de Supabase (`@supabase/supabase-js`) o APIs REST equivalentes
* APIs Web (`sessionStorage` o `localStorage`)

## Guía de Ejecución Interactiva (Instrucciones para el Agente)
Cuando implementes almacenamiento en caché de tablas, haz estas preguntas interactivas antes:
1. **Tipo de Persistencia**: ¿Los datos deben persistir entre pestañas/sesiones (localStorage) o solo durante la pestaña activa (sessionStorage)?
2. **Frecuencia de Modificación**: ¿Qué acciones de la interfaz (insertar, editar, eliminar, sincronizar) requieren invalidar la caché inmediatamente?
3. **Manejo de Pestañas/Filtros**: ¿Existen diferentes pestañas o vistas (por ejemplo, modo normal frente a modo original) que requieran claves de caché separadas?

---

## Flujo de Trabajo y Patrones de Diseño

### 1. Patrón Stale-While-Revalidate (SWR)
Para evitar que el usuario espere con una pantalla en blanco o un spinner, carga los datos guardados en la caché inmediatamente. Si existen, desactiva el estado de carga (`isLoading = false`) y muestra los registros viejos mientras lanzas la consulta a la base de datos en segundo plano.

### 2. Invalidación en Mutaciones (Force Refresh)
Cada vez que el usuario agregue, edite, elimine o ejecute tareas de importación de datos, la caché local queda obsoleta.
* **Solución**: Pasa un parámetro `forceRefresh = true` a la función de carga que elimine la entrada de la caché en el almacenamiento del navegador antes de realizar la petición HTTP/Supabase.

### 3. Evitar Condiciones de Carrera (Race Conditions)
Si el usuario cambia de filtros o pestañas rápidamente, múltiples consultas asíncronas de fondo pueden ejecutarse al mismo tiempo. La respuesta más lenta podría sobrescribir los datos actuales con información errónea o de otra vista.
* **Solución**: Mantener una referencia de ID de consulta única (`activeQueryIdRef`) que se incrementa en cada llamada a la función. Las promesas de fondo solo deben actualizar el estado si su `queryId` coincide con el valor actual de la referencia.

---

## Código de Referencia de Implementación

### Implementación en React (`page.tsx`)
```tsx
import { useEffect, useState, useRef } from "react";
import { supabase } from "./lib/supabase";

export default function CachedGrid() {
  const [items, setItems] = useState<any[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const activeQueryIdRef = useRef(0);

  const fetchItems = async (forceRefresh = false) => {
    const cacheKey = "sc_items_cache";
    const queryId = ++activeQueryIdRef.current;

    let hasCache = false;
    if (!forceRefresh) {
      try {
        const cachedData = sessionStorage.getItem(cacheKey);
        if (cachedData) {
          const parsed = JSON.parse(cachedData);
          if (Array.isArray(parsed) && queryId === activeQueryIdRef.current) {
            setItems(parsed);
            hasCache = true;
          }
        }
      } catch (err) {
        console.error("Error reading cache:", err);
      }
    } else {
      sessionStorage.removeItem(cacheKey);
    }

    // Solo mostrar spinner si no hay cache previa
    if (!hasCache) {
      setIsLoading(true);
    }

    try {
      // 1. Carga inicial rapida de base de datos
      const { data: initialData, error } = await supabase
        .from("items")
        .select("id, name, status, updated_at") // EVITAR select("*") para prevenir QuotaExceededError
        .range(0, 49);

      if (error) {
        console.error("Error en carga inicial:", error);
        if (queryId === activeQueryIdRef.current) {
          setIsLoading(false);
        }
        return;
      }

      if (queryId !== activeQueryIdRef.current) return;

      if (initialData) {
        setItems(initialData);
      }
      setIsLoading(false);

      // 2. Cargar el resto en segundo plano
      let allItems = initialData ? [...initialData] : [];
      let from = 50;
      let to = 1049;
      let hasMore = initialData && initialData.length === 50;

      const fetchRemaining = async () => {
        while (hasMore && queryId === activeQueryIdRef.current) {
          const { data, error } = await supabase
            .from("items")
            .select("id, name, status, updated_at") // EVITAR select("*") para prevenir QuotaExceededError
            .range(from, to);

          if (error) {
            console.error("Error cargando resto:", error);
            break;
          }

          if (queryId !== activeQueryIdRef.current) break;

          if (data && data.length > 0) {
            allItems = [...allItems, ...data];
            setItems([...allItems]);
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

        // Guardar cache completa cuando finalice
        if (queryId === activeQueryIdRef.current) {
          try {
            sessionStorage.setItem(cacheKey, JSON.stringify(allItems));
          } catch (err) {
            console.error("Error saving cache:", err);
          }
        }
      };

      if (hasMore) {
        fetchRemaining();
      } else {
        try {
          sessionStorage.setItem(cacheKey, JSON.stringify(allItems));
        } catch (err) {
          console.error("Error saving cache:", err);
        }
      }

    } catch (err) {
      console.error("Fallo general:", err);
      if (queryId === activeQueryIdRef.current) {
        setIsLoading(false);
      }
    }
  };

  useEffect(() => {
    fetchItems();
    return () => {
      activeQueryIdRef.current = 0; // Desactivar consultas pendientes al desmontar
    };
  }, []);

  // Ejemplo de mutacion (Insertar)
  const handleInsert = async () => {
    // ... logica de insertado ...
    await fetchItems(true); // Forzar refresco e invalidar cache
  };

  return (
    <div>
      {isLoading ? <p>Cargando...</p> : <List items={items} />}
    </div>
  );
}
```

---

## Errores Comunes
1. **Omitir la invalidacion**: No pasar `forceRefresh = true` en mutaciones, lo que provoca que la UI siga mostrando datos cacheados obsoletos hasta recargar la pestaña.
2. **Sin control de Query ID**: No validar `queryId === activeQueryIdRef.current` antes de guardar la cache o actualizar el estado, causando mezcla de datos en cambios rápidos de vistas.
3. **Guardar datos parciales**: Almacenar la lista en la caché local antes de que el bucle de segundo plano haya terminado, guardando solo la página inicial.
4. **QuotaExceededError (Límite de Almacenamiento)**: Al usar `.select("*")` en tablas con columnas pesadas (ej. texto largo, markdown, json, dependencias), la caché guardada en `sessionStorage` o `localStorage` excede la cuota de almacenamiento del navegador (generalmente 5MB).
   * **Solución**: Especificar siempre de forma explícita las columnas mínimas y necesarias en la cláusula `.select("col1, col2, ...")` para evitar traer datos pesados que no se muestran en la cuadrícula/tabla principal.
