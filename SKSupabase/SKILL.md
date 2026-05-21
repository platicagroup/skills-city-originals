# Protección de Contraseñas Filtradas en Supabase (Leaked Password Protection)
Especificación y guía paso a paso para habilitar y configurar la protección contra contraseñas comprometidas (filtradas) en Supabase Auth, integrando la validación contra HaveIBeenPwned.org.

## Guía de Ejecución Interactiva (Instrucciones para el Agente)

Cuando un desarrollador te solicite configurar o solucionar la advertencia de contraseñas filtradas en Supabase, debes realizar las siguientes preguntas interactivas en español antes de continuar:

1. **Acceso al Dashboard**: ¿Tienes acceso como administrador al panel de control de Supabase (https://supabase.com/dashboard)?
2. **Entornos de Aplicación**: ¿Deseas habilitar esta regla de seguridad solo en tu proyecto de Producción o también en entornos de Desarrollo/Staging?
3. **Políticas de Contraseña Complementarias**: ¿Te gustaría configurar otras reglas de complejidad de contraseña al mismo tiempo (ej. longitud mínima, caracteres especiales, números)?

---

## Introducción a Leaked Password Protection en Supabase

Supabase Auth incluye una característica de seguridad nativa que compara las contraseñas introducidas por los usuarios durante el registro o cambio de credenciales contra la base de datos de contraseñas filtradas de **HaveIBeenPwned.org**.

Esta validación se realiza de manera segura y confidencial utilizando un método de hashing parcial (k-Anonymity), lo que significa que Supabase nunca envía la contraseña en texto plano ni hashes completos a servidores externos. Si la contraseña coincide con una filtración conocida, Supabase rechaza el registro o actualización con un error descriptivo.

---

## Cómo Solucionar el Error desde el Dashboard de Supabase

Sigue estos pasos detallados para activar la protección de contraseñas filtradas:

1. **Iniciar Sesión**:
   * Accede a [Supabase Dashboard](https://supabase.com/dashboard).
   * Selecciona tu proyecto.

2. **Navegar a la Configuración de Autenticación**:
   * En el menú lateral izquierdo, haz clic en **Authentication** (icono de llave).
   * Selecciona la pestaña o sección de **Providers** o **Auth Settings** (dependiendo de la versión de la interfaz). En versiones recientes se encuentra bajo **Settings** (icono de engranaje) -> **Auth**.

3. **Habilitar la Protección**:
   * Desplázate hacia abajo hasta encontrar la sección **Security Settings** o **Password Protection**.
   * Busca la opción llamada **Prevent use of compromised passwords** (o **Leaked password protection**).
   * Activa el interruptor/switch correspondiente.

4. **Guardar los Cambios**:
   * Haz clic en el botón **Save** en la parte inferior o derecha de la pantalla.

---

## Mensajes de Error Esperados

Una vez activado, cuando un usuario intente registrarse o cambiar su contraseña por una que ha sido comprometida en filtraciones públicas, la API de Supabase Auth devolverá un error HTTP 400 con el siguiente código y descripción:

```json
{
  "code": 400,
  "error_code": "weak_password",
  "message": "Password is too weak or has been compromised in a data breach"
}
```

Es recomendable capturar este código de error en el frontend (`frontend/src/app/page.tsx` u otros formularios de login/registro) para mostrar un aviso amigable e instructivo al usuario.

---

## Seguridad en Funciones de Supabase (SECURITY DEFINER)

Supabase y PostgreSQL permiten definir funciones con el modificador `SECURITY DEFINER`. Estas funciones se ejecutan con los privilegios del rol propietario (usualmente `postgres`), lo cual requiere precauciones estrictas de seguridad.

### 1. Establecer `search_path` Seguro
Cuando una función se ejecuta como `SECURITY DEFINER`, hereda los esquemas de búsqueda (`search_path`) del usuario que la invoca. Esto abre la posibilidad de que un atacante cree objetos falsificados en su propio esquema y engañe al sistema.
* **Solución**: Fijar siempre la ruta del esquema explícitamente en la definición de la función:
  ```sql
  alter function public.handle_new_user() set search_path = public;
  ```

### 2. Revocar Permisos de Ejecución al Grupo `PUBLIC`
Por defecto, toda función nueva en PostgreSQL tiene permisos de ejecución otorgados al rol especial `PUBLIC` (cualquier visitante o usuario autenticado). En funciones críticas o desencadenadas por triggers (como la creación automática de perfiles de usuario), esto permitiría a clientes externos invocar el código arbitrariamente.
* **Solución**: Revocar explícitamente los privilegios públicos:
  ```sql
  revoke execute on function public.handle_new_user() from public;
  revoke execute on function public.rls_auto_enable() from public;
  ```
  *(Nota: Los triggers del sistema seguirán ejecutando la función correctamente a nivel interno de base de datos).*

