# Laboratorio D06 — Supabase: Configuración y primera consulta

**SIS-0300 Programación Web I · Fase 2 · Día 6**  
Universidad Privada Domingo Savio · Facultad de Ingeniería

---

## ¿Qué cambia hoy?

Hasta D05, `useMovie.ts` leía películas de un array local en `data/peliculas.ts`.  
Hoy ese array desaparece. Las películas van a vivir en una base de datos real en la nube — PostgreSQL gestionado por Supabase.

El resto de la app (`MovieCard`, `MovieList`, `Catalog`) **no toca ni una línea**. Eso demuestra que el hook funciona como una capa de abstracción: los componentes no saben ni les importa de dónde vienen los datos.

---

## Estructura de archivos del laboratorio completo

```
cine-app/
└── src/
    ├── lib/
    │   └── supabaseClient.ts     ← NUEVO (Tarea 5)
    ├── hooks/
    │   └── useMovie.ts           ← MODIFICADO (Tarea 6)
    ├── types/
    │   └── Movie.ts              ← sin cambios
    ├── components/
    │   ├── MovieCard.tsx         ← sin cambios
    │   └── MovieList.tsx         ← sin cambios
    └── pages/
        └── Catalog.tsx           ← agrega manejo de error (Tarea 7)
```

---

---

# PARTE 1 — Trabajo en clase

---

## Tarea 0 — Crear el proyecto en Supabase

### Qué haces

Crear el proyecto remoto que va a alojar la base de datos de CineApp.

### Pasos

**0.1** Abrí el navegador y entrá a [https://supabase.com](https://supabase.com).

**0.2** Iniciá sesión con tu cuenta (o creá una con GitHub si no tenés).

**0.3** En el dashboard hacé clic en **New project**.

**0.4** Completá el formulario:

| Campo | Valor |
|---|---|
| Name | `cine-app` |
| Database Password | una contraseña segura — **anotarla** |
| Region | South America (São Paulo) |

**0.5** Hacé clic en **Create new project** y esperá aproximadamente 1 minuto mientras Supabase aprovisiona la base de datos.

> No cierres la pestaña. Cuando el indicador de estado pase a verde, el proyecto está listo.

### ¿Qué verificar?

El dashboard muestra el proyecto `cine-app` con el estado **Active** en verde.

---

## Tarea 1 — Crear la tabla `peliculas`

### Qué haces

Crear la tabla y cargar los datos de prueba usando únicamente la interfaz visual de Supabase — sin escribir SQL.

### Paso 1.1 — Abrir el Table Editor

En el menú lateral izquierdo hacé clic en **Table Editor**.

---

### Paso 1.2 — Crear la tabla

Hacé clic en el botón **New table** (esquina superior derecha).

Se abre un panel de configuración. Completá los campos generales:

| Campo | Valor |
|---|---|
| Name | `peliculas` |
| Description | *(dejar vacío)* |
| Enable Row Level Security (RLS) | **desactivado por ahora** — lo configuramos en la Tarea 2 |

---

### Paso 1.3 — Definir las columnas

El panel ya incluye la columna `id` creada automáticamente — no la toques.

Hacé clic en **Add column** para agregar cada columna de la siguiente tabla. Repetí el proceso para las 6 columnas:

| Name | Type | Default value | Nullable |
|---|---|---|---|
| `titulo` | `text` | *(vacío)* | **desactivado** |
| `anio` | `int4` | *(vacío)* | **desactivado** |
| `genero` | `text` | *(vacío)* | **desactivado** |
| `director` | `text` | *(vacío)* | activado |
| `sinopsis` | `text` | *(vacío)* | activado |
| `calificacion` | `numeric` | *(vacío)* | activado |

> **Nullable desactivado** = campo obligatorio (`NOT NULL`).  
> Las columnas `director`, `sinopsis` y `calificacion` pueden estar vacías, por eso van con Nullable activado.

**Al terminar de agregar las 6 columnas hacé clic en Save.**

---

### Paso 1.4 — Verificar la estructura

La tabla `peliculas` aparece en el listado del Table Editor con 7 columnas: `id`, `titulo`, `anio`, `genero`, `director`, `sinopsis`, `calificacion`.

---

### Paso 1.5 — Insertar las películas de prueba

Con la tabla `peliculas` seleccionada, hacé clic en **Insert** → **Insert row**.

Se abre un formulario con todos los campos. Completá los datos de la primera película:

| Campo | Valor |
|---|---|
| titulo | `Interstellar` |
| anio | `2014` |
| genero | `Ciencia Ficción` |
| director | `Christopher Nolan` |
| sinopsis | `Un equipo de astronautas viaja a través de un agujero de gusano en busca de un nuevo hogar para la humanidad.` |
| calificacion | `8.6` |

Hacé clic en **Save**. La fila aparece en la tabla.

Repetí el proceso para las 5 películas restantes:

| titulo | anio | genero | director | sinopsis | calificacion |
|---|---|---|---|---|---|
| `El Padrino` | `1972` | `Drama` | `Francis Ford Coppola` | `La historia de la familia mafiosa Corleone y la sucesión del poder entre generaciones.` | `9.2` |
| `Spirited Away` | `2001` | `Animación` | `Hayao Miyazaki` | `Una niña de diez años queda atrapada en un mundo de espíritus y debe trabajar para liberar a sus padres.` | `8.6` |
| `Parasite` | `2019` | `Drama` | `Bong Joon-ho` | `Una familia pobre se infiltra en la vida de una familia adinerada con consecuencias inesperadas.` | `8.5` |
| `The Dark Knight` | `2008` | `Drama` | `Christopher Nolan` | `Batman enfrenta al Joker, un criminal que busca sumir Gotham City en el caos.` | `9.0` |
| `Your Name` | `2016` | `Animación` | `Makoto Shinkai` | `Dos estudiantes de ciudades distintas intercambian cuerpos misteriosamente mientras duermen.` | `8.4` |

### ¿Qué verificar?

El Table Editor muestra la tabla `peliculas` con 6 filas. La columna `id` fue asignada automáticamente — no la editaste en ningún momento.

---

## Tarea 2 — Habilitar RLS y crear política de lectura pública

### Qué haces

Row Level Security (RLS) es el sistema de seguridad de Supabase que controla qué operaciones puede hacer cada tipo de usuario sobre cada fila. Por defecto, cuando se habilita RLS, **todo queda bloqueado** hasta que se definan políticas explícitas.

Hoy vas a habilitar RLS y crear una política que permita a cualquier visitante leer las películas, pero **sin poder insertar, modificar ni eliminar**.

### Por qué RLS y no solo la anon key

La `anon key` (clave pública) viaja en el código del frontend — cualquier persona que abra las herramientas de desarrollador del navegador puede verla. RLS asegura que aunque alguien tenga esa clave, solo pueda hacer lo que las políticas le permiten.

### Pasos por la interfaz

**2.1** En el menú lateral hacé clic en **Authentication**.

**2.2** En el submenú hacé clic en **Policies**.

**2.3** Buscá la tabla `peliculas` en la lista. Vas a ver que dice **RLS disabled** — eso significa que la tabla está completamente abierta.

**2.4** Hacé clic en el botón **Enable RLS** que aparece a la derecha del nombre de la tabla.

**2.5** Un modal de confirmación aparece. Hacé clic en **Enable RLS**.

> Ahora la tabla dice **RLS enabled** pero tiene 0 políticas. En este momento **nadie puede leer nada**, ni siquiera los visitantes anónimos.

**2.6** Hacé clic en **New policy**.

**2.7** Supabase muestra una pantalla con plantillas. Hacé clic en **Create a policy from scratch** (crear desde cero).

**2.8** Completá el formulario con exactamente estos valores:

| Campo | Valor |
|---|---|
| Policy name | `lectura publica` |
| Allowed operation | **SELECT** |
| Target roles | *(dejar vacío — aplica a todos, incluyendo anon)* |
| USING expression | `true` |

> La expresión `true` significa "esta política se cumple siempre" — cualquier fila puede ser leída. En D12 (Auth) vamos a crear políticas más específicas para INSERT y UPDATE que requieran usuario autenticado.

**2.9** Hacé clic en **Save policy**.

**2.10** La tabla `peliculas` ahora muestra **1 policy** activa.

### ¿Cómo leer la política que creaste?

```
Para la operación SELECT sobre la tabla peliculas:
  ¿Puede ejecutarse?  → true (siempre sí)
  ¿A quién aplica?    → a todos los roles (anon, authenticated)
```

### ¿Qué verificar?

En la sección Policies, la tabla `peliculas` muestra RLS enabled con 1 política llamada `lectura publica` de tipo `SELECT`.

---

## Tarea 3 — Instalar el SDK de Supabase

### Qué haces

Agregar la librería oficial de Supabase al proyecto para poder comunicarte con la base de datos desde React.

### Pasos

**3.1** Abrí la terminal integrada de VS Code con `Ctrl + J`.

**3.2** Asegurate de estar dentro de la carpeta del proyecto:

```bash
cd cine-app
```

**3.3** Ejecutá el comando de instalación:

```bash
npm install @supabase/supabase-js
```

**3.4** Cuando termine, verificá que `package.json` ahora incluye `@supabase/supabase-js` dentro de `dependencies`.

### ¿Qué verificar?

En `package.json` aparece la línea `"@supabase/supabase-js": "^2.x.x"` bajo `dependencies`. La versión exacta puede variar.

---

## Tarea 4 — Obtener las credenciales y crear el archivo `.env.local`

### Qué haces

Guardar las credenciales del proyecto Supabase en un archivo de variables de entorno. Así el código nunca va a tener credenciales hardcodeadas.

### Por qué `.env.local`

`.env.local` está incluido en el `.gitignore` que Vite genera automáticamente al crear el proyecto — las credenciales nunca van a subir al repositorio por accidente.

### Paso 4.1 — Copiar las credenciales desde Supabase

En el dashboard de Supabase:

**a)** Hacé clic en el ícono de engranaje (**Project Settings**) en el menú lateral inferior.

**b)** En el submenú hacé clic en **API**.

**c)** Copiá los dos valores marcados en la siguiente tabla:

| Qué copiar | Dónde está en la pantalla | Nombre en `.env.local` |
|---|---|---|
| URL del proyecto | Sección **Project URL** | `VITE_SUPABASE_URL` |
| Clave pública | Sección **Project API keys** → fila **anon public** | `VITE_SUPABASE_ANON_KEY` |

> **No copies** la `service_role` key — esa clave tiene acceso total y nunca debe ir al frontend.

### Paso 4.2 — Crear el archivo `.env.local`

**a)** En VS Code, en el panel **Explorer** (izquierda), hacé clic derecho sobre la raíz del proyecto → **New File**.

**b)** Nombrá el archivo exactamente `.env.local` (con el punto al inicio).

**c)** Pegá el siguiente contenido y reemplazá los valores de ejemplo con los que copiaste:

```env
VITE_SUPABASE_URL=https://xxxxxxxxxxxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

> El prefijo `VITE_` es obligatorio. Vite solo expone al navegador las variables que empiezan con `VITE_` — las demás quedan ocultas aunque estén en el archivo.

**d)** Guardá con `Ctrl + S`.

### Paso 4.3 — Verificar que `.env.local` está protegido

Abrí el archivo `.gitignore` en la raíz del proyecto. Debe contener la línea:

```
*.local
```

Esa regla cubre `.env.local` y cualquier otro archivo `.local` que puedas agregar en el futuro. Vite la incluye por defecto — no necesitás modificar nada.

### ¿Qué verificar?

El archivo `.env.local` aparece en el Explorer de VS Code con el ícono o color que indica que está ignorado por Git (gris en la mayoría de los temas con la extensión GitLens o el soporte nativo de VS Code).

---

## Tarea 5 — Crear el cliente centralizado

### Qué haces

Crear el único archivo que inicializa la conexión con Supabase. Todos los hooks y servicios de la app van a importar el cliente desde aquí — nunca lo van a inicializar por su cuenta.

### Pasos

**5.1** Creá el archivo `src/lib/supabaseClient.ts`:

```ts
// src/lib/supabaseClient.ts
// Cliente único de Supabase — importar desde aquí en toda la app
// Nunca inicialices createClient() dentro de un componente o hook

import { createClient } from "@supabase/supabase-js";

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

**5.2** Guardá. No debe haber errores de TypeScript.

### ¿Por qué un archivo centralizado?

Si mañana necesitás cambiar las credenciales o agregar opciones al cliente (como tiempo de espera o headers personalizados), lo hacés en un solo lugar y el cambio aplica a toda la app.

---

## Tarea 5.5 — Verificar la conexión desde la consola

### Qué haces

Antes de integrar Supabase con el hook, verificar que las credenciales son correctas y que la tabla responde. Una prueba rápida en `App.tsx` — la eliminás después.

### Pasos

**5.5.1** Abrí `src/App.tsx` y agregá este bloque al inicio del componente, antes del `return`:

```tsx
// src/App.tsx
import { supabase } from "./lib/supabaseClient";
import { useEffect } from "react";

const App = () => {
  useEffect(() => {
    // Prueba de conexión — eliminar después de verificar
    const probar = async () => {
      const { data, error } = await supabase
        .from("peliculas")
        .select("*");

      console.log("data:", data);
      console.log("error:", error);
    };

    probar();
  }, []);

  // ... resto del componente
```

**5.5.2** Guardá y abrí el navegador en `http://localhost:5173`.

**5.5.3** Abrí las herramientas de desarrollador con `F12` → pestaña **Console**.

Resultado esperado:
```
data: (6) [{...}, {...}, {...}, {...}, {...}, {...}]
error: null
```

Si `data` es `null` y `error` tiene un mensaje, revisá:
- Que las variables en `.env.local` no tengan espacios ni comillas
- Que la política RLS de la Tarea 2 esté activa
- Que el nombre de la tabla en la consulta sea exactamente `peliculas`

**5.5.4** Una vez verificado, **eliminá el bloque de prueba** de `App.tsx` y continuá con la Tarea 6.

---

## Tarea 6 — Actualizar `useMovie.ts`

### Qué haces

Reemplazar la lectura del array local por una consulta real a Supabase.

### Pasos

**6.1** Reemplazá el contenido completo de `src/hooks/useMovie.ts`:

```ts
// src/hooks/useMovie.ts
// Hook para obtener películas desde Supabase

import { useEffect, useState } from "react";
import type { Movie } from "../types/Movie";
import { supabase } from "../lib/supabaseClient";

const useMovie = () => {
  const [peliculas, setPeliculas] = useState<Movie[]>([]);
  const [cargando, setCargando] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // Función async dentro de useEffect — patrón estándar con Supabase
    const cargarPeliculas = async () => {
      setCargando(true);
      setError(null);

      const { data, error: supabaseError } = await supabase
        .from("peliculas")
        .select("*")
        .order("calificacion", { ascending: false });

      if (supabaseError) {
        // Guardar el mensaje de error para mostrarlo en la UI
        setError("Error al cargar las películas: " + supabaseError.message);
      } else {
        // data puede ser null si no hay filas — usar ?? para evitar null
        setPeliculas(data ?? []);
      }

      setCargando(false);
    };

    cargarPeliculas();
  }, []); // [] = ejecutar solo al montar el componente, no en cada render

  return { peliculas, cargando, error };
};

export default useMovie;
```

**6.2** Guardá. El linter no debe mostrar errores.

### ¿Qué verificar?

Notá que el componente `Catalog` no cambió en absoluto — llama a `useMovie()` igual que antes y recibe `{ peliculas, cargando }`. El hook es la única pieza que sabe que los datos vienen de Supabase.

---

## Tarea 7 — Mostrar errores en `Catalog.tsx`

### Qué haces

El hook ahora expone `error`. Si la consulta falla (credenciales incorrectas, red caída, política RLS bloqueando), el usuario debe ver un mensaje claro en lugar de una pantalla vacía.

### Pasos

**7.1** En `src/pages/Catalog.tsx` actualizá la desestructuración del hook:

```tsx
// Antes
const { peliculas, cargando } = useMovie();

// Después
const { peliculas, cargando, error } = useMovie();
```

**7.2** Agregá el bloque de error **después** del bloque de cargando:

```tsx
if (cargando)
  return (
    <div className="min-h-screen bg-gray-900 flex items-center justify-center">
      <p className="text-gray-400">Cargando películas...</p>
    </div>
  );

// Bloque nuevo — agregar aquí
if (error)
  return (
    <div className="min-h-screen bg-gray-900 flex items-center justify-center">
      <p className="text-red-400">{error}</p>
    </div>
  );
```

**7.3** Guardá y verificá que no hay errores de TypeScript.

---

## Verificación final

Ejecutá el proyecto:

```bash
npm run dev
```

```
✅  Las películas aparecen en pantalla — ya no vienen del array local
✅  El orden es por calificación descendente (El Padrino primero)
✅  El filtro por género sigue funcionando sin cambios
✅  La consola del navegador no muestra errores de Supabase
✅  En Supabase → Table Editor → peliculas se ven las 6 filas
✅  En Supabase → Authentication → Policies: tabla peliculas tiene 1 política SELECT
```

---

---

# PARTE 2 — Tarea autónoma

---

## Contexto

Ya tenés la tabla `peliculas` conectada a CineApp. Ahora vas a repetir el mismo flujo de forma independiente: crear una nueva entidad en Supabase, conectarla desde React y mostrarla en una página propia — todo en un solo archivo, sin extraer componentes.

La entidad a crear es **`directores`** — una tabla simple con información básica de directores de cine.

---

## Tarea A — Crear la tabla `directores` en Supabase

**A.1** En Supabase → **Table Editor** → **New table**. Completá los campos:

| Campo | Valor |
|---|---|
| Name | `directores` |
| Enable RLS | activado |

**A.2** Agregá las siguientes columnas:

| Name | Type | Nullable |
|---|---|---|
| `nombre` | `text` | desactivado |
| `nacionalidad` | `text` | activado |
| `anio_nacimiento` | `int4` | activado |

Hacé clic en **Save**.

**A.3** Habilitá la política de lectura pública: **Authentication** → **Policies** → tabla `directores` → **New policy** → **Create from scratch**:

| Campo | Valor |
|---|---|
| Policy name | `lectura publica` |
| Allowed operation | SELECT |
| USING expression | `true` |

Guardá la política.

**A.4** Insertá al menos 5 directores desde **Insert row**:

| nombre | nacionalidad | anio_nacimiento |
|---|---|---|
| `Christopher Nolan` | `Británico` | `1970` |
| `Hayao Miyazaki` | `Japonés` | `1941` |
| `Bong Joon-ho` | `Surcoreano` | `1969` |
| `Francis Ford Coppola` | `Estadounidense` | `1939` |
| `Makoto Shinkai` | `Japonés` | `1973` |

---

## Tarea B — Crear la página `Directors.tsx`

**B.1** Creá el archivo `src/pages/Directors.tsx`. Toda la lógica y la UI van en este único archivo — sin componentes separados.

El archivo debe:

- Consultar la tabla `directores` desde Supabase al montar la página
- Mostrar un estado de carga mientras llegan los datos
- Mostrar un mensaje de error si la consulta falla
- Listar los directores en tarjetas simples con nombre, nacionalidad y año de nacimiento

**B.2** Estructura esperada del archivo:

```tsx
// src/pages/Directors.tsx

import { useEffect, useState } from "react";
import { supabase } from "../lib/supabaseClient";

// Tipo local — solo lo usa esta página, no necesita archivo separado
interface Director {
  id: number;
  nombre: string;
  nacionalidad: string | null;
  anio_nacimiento: number | null;
}

const Directors = () => {
  const [directores, setDirectores] = useState<Director[]>([]);
  const [cargando, setCargando] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // completar
  }, []);

  if (cargando) return // completar
  if (error) return // completar

  return (
    // completar — listar directores con sus datos
  );
};

export default Directors;
```

**B.3** Actualizá `App.tsx` para que renderice `Directors` en lugar de (o además de) `Catalog`. Verificá que los datos aparecen en pantalla.

---

## ¿Qué aprendiste hoy?

| Concepto | Dónde lo aplicaste |
|---|---|
| BaaS (Backend as a Service) | Supabase provee BD, API y auth sin servidor propio |
| Variable de entorno | `.env.local` con `VITE_SUPABASE_URL` y `VITE_SUPABASE_ANON_KEY` |
| Cliente centralizado | `src/lib/supabaseClient.ts` — un solo `createClient()` |
| Consulta asíncrona | `useEffect` + `async/await` + desestructuración `{ data, error }` |
| Row Level Security | Tabla con RLS habilitado + política SELECT pública |
| Abstracción con hooks | `Catalog` no sabe que los datos vienen de Supabase |
