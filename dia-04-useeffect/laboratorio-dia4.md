# Laboratorio — Día 4

## Tailwind CSS · useEffect · Custom Hooks

**SIS-0300 Programación Web I · Fase 1 · CineApp**  
**Modalidad:** individual

---

## Contexto

La `CineApp` ya tiene `Catalog.tsx` con películas hardcodeadas y estilos inline. En este laboratorio vamos a trabajar en dos partes:

- **Parte 1:** reemplazar los estilos inline de `MovieCard` y `MovieList` por **Tailwind CSS**
- **Parte 2:** extraer la lógica de datos a un **custom hook `useMovies`** usando `useEffect`

> `Catalog.tsx`, `HeaderTitle` y `MovieFooter` **no se tocan**.

---

# Parte 1 — Tailwind CSS

## Instalación

```bash
npm install -D tailwindcss @tailwindcss/vite
```

En `vite.config.ts`, agregar el plugin:

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

En `src/index.css`, reemplazar todo el contenido por:

```css
@import "tailwindcss";
```

Verificar que `src/main.tsx` importa el CSS:

```tsx
import "./index.css";
```

---

## Actividad 1 — Tailwind en `MovieCard`

Reemplazar todos los `style={{...}}` por clases de Tailwind:

```tsx
// src/components/MovieCard.tsx
import type { Movie } from "../types/Movie";

interface MovieCardProps {
  pelicula: Movie;
}

const MovieCard = ({ pelicula }: MovieCardProps) => {
  return (
    <div className="bg-white rounded-xl shadow-md p-4 flex flex-col gap-2 hover:shadow-lg transition-shadow">
      <h3 className="text-base font-bold text-slate-800 leading-tight">
        {pelicula.titulo}
      </h3>

      <p className="text-sm text-slate-500">
        {pelicula.anio} — {pelicula.genero}
      </p>

      <p className="text-sm font-semibold text-emerald-600 mt-auto">
        ⭐ {pelicula.calificacion} / 10
      </p>
    </div>
  );
};

export default MovieCard;
```

---

## Actividad 2 — Tailwind en `MovieList`

Reemplazar el `style={{...}}` del grid por clases de Tailwind:

```tsx
// src/components/MovieList.tsx
import type { Movie } from "../types/Movie";
import MovieCard from "./MovieCard";

interface MovieListProps {
  peliculas: Movie[];
}

const MovieList = ({ peliculas }: MovieListProps) => {
  return (
    <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-5">
      {peliculas.map((p) => (
        <MovieCard key={p.id} pelicula={p} />
      ))}
    </div>
  );
};

export default MovieList;
```

---

# Parte 2 — useEffect y Custom Hook

## Actividad 3 — Separar los datos

Crear `src/data/peliculas.ts` y mover ahí la constante `Peliculas` que hoy vive en `Catalog.tsx`:

```ts
// src/data/peliculas.ts
import type { Movie } from "../types/Movie";

// Datos hardcodeados — en Fase 5 esto se reemplaza por la API de TMDB
export const PELICULAS: Movie[] = [
  {
    id: 1,
    titulo: "Interestelar",
    anio: 2014,
    genero: "Ciencia ficción",
    calificacion: 9.5,
  },
  {
    id: 2,
    titulo: "Spiderman",
    anio: 2002,
    genero: "Acción",
    calificacion: 9.0,
  },
  {
    id: 3,
    titulo: "El conjuro",
    anio: 2013,
    genero: "Terror",
    calificacion: 8.5,
  },
  { id: 4, titulo: "Parasite", anio: 2019, genero: "Drama", calificacion: 9.0 },
  {
    id: 5,
    titulo: "Dune",
    anio: 2021,
    genero: "Ciencia ficción",
    calificacion: 8.8,
  },
  {
    id: 6,
    titulo: "El conjuro 2",
    anio: 2016,
    genero: "Terror",
    calificacion: 8.0,
  },
];
```

---

## Actividad 4 — Custom Hook `useMovies`

Crear `src/hooks/useMovies.ts`:

```ts
// src/hooks/useMovies.ts
import { useState, useEffect } from "react";
import type { Movie } from "../types/Movie";
import { PELICULAS } from "../data/peliculas";

const useMovies = () => {
  const [peliculas, setPeliculas] = useState<Movie[]>([]);
  const [cargando, setCargando] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // Simula latencia de red — en Fase 5 aquí irá fetch a TMDB
    const timer = setTimeout(() => {
      try {
        setPeliculas(PELICULAS);
      } catch {
        setError("Error al cargar las películas");
      } finally {
        setCargando(false);
      }
    }, 800);

    // Cleanup: cancela el timer si el componente se desmonta antes de los 800ms
    return () => clearTimeout(timer);
  }, []); // [] = solo se ejecuta al montar el componente

  return { peliculas, cargando, error };
};

export default useMovies;
```

**Preguntas de reflexión** — responder como comentario en el archivo:

- ¿Qué pasaría si sacás el `[]` del `useEffect`?
- ¿Por qué hacemos `clearTimeout` en el cleanup?

---

## Actividad 5 — Conectar `useMovies` en `Catalog`

Son tres cambios puntuales en `Catalog.tsx`. El panel de claps, el input de búsqueda, los botones de género y todos los estilos **no se tocan**.

**Cambio 1 — imports:** quitar el import de `Peliculas` y agregar `useMovies`:

```tsx
// ❌ Quitar
import { Peliculas } from "../data/peliculas";

// ✅ Agregar
import useMovies from "../hooks/useMovies";
```

**Cambio 2 — dentro del componente:** reemplazar el uso directo de `Peliculas` por el hook:

```tsx
// ❌ Quitar — ya no se necesita, los datos vienen del hook
// (la constante Peliculas ya no existe en este archivo)

// ✅ Agregar al inicio del componente
const { peliculas, cargando, error } = useMovies();
```

**Cambio 3 — guards antes del return:** agregar justo antes del `return (`:

```tsx
if (cargando)
  return (
    <p className="text-center text-slate-400 mt-20">Cargando películas...</p>
  );
if (error) return <p className="text-center text-red-500 mt-20">{error}</p>;
```

**Cambio 4 — `peliculasFiltradas`:** solo renombrar `Peliculas` por `peliculas` (minúscula):

```tsx
// ❌ Antes
const peliculasFiltradas =
  generoActivo === "Todos"
    ? Peliculas
    : Peliculas.filter(
        (p) => p.genero.toLowerCase() == generoActivo.toLowerCase(),
      );

// ✅ Después
const peliculasFiltradas =
  generoActivo === "Todos"
    ? peliculas
    : peliculas.filter(
        (p) => p.genero.toLowerCase() == generoActivo.toLowerCase(),
      );
```

> Todo lo demás — panel de claps, nombre, búsqueda, botones de género, estilos inline — queda exactamente igual.

---

---

## Actividad Autónoma — `useFavorites`

### Paso 1 — Crear el hook

Crear `src/hooks/useFavorites.ts`. El estado interno es un array de `number` que almacena los IDs de las películas marcadas.

El hook debe exponer:

- `favoritos` — array de IDs marcados, tipo `number[]`
- `toggleFavorito(id: number)` — si el ID ya está en el array lo quita, si no está lo agrega
- `esFavorito(id: number): boolean` — retorna `true` si el ID está en `favoritos`

> Pista: para quitar un elemento de un array sin mutarlo usá `.filter()`. Para agregar sin mutar usá el spread `[...prev, id]`.

### Paso 2 — Actualizar `MovieCardProps`

Agregar dos props opcionales a la interface en `MovieCard.tsx`:

- `isFavorito?: boolean`
- `onToggleFavorito?: () => void`

Mostrar un botón `★` en la tarjeta que:

- Llame a `onToggleFavorito` al hacer clic
- Tenga color amarillo si `isFavorito` es `true`, gris si es `false`

### Paso 3 — Actualizar `MovieListProps`

`MovieList` necesita recibir las funciones del hook y pasarlas a cada `MovieCard`. Agregar a su interface:

- `esFavorito: (id: number) => boolean`
- `onToggleFavorito: (id: number) => void`

### Paso 4 — Conectar en `Catalog`

Llamar al hook dentro de `Catalog`:

```tsx
const { esFavorito, toggleFavorito } = useFavorites();
```

Y pasar las funciones a `MovieList`:

```tsx
<MovieList
  peliculas={peliculasFiltradas}
  esFavorito={esFavorito}
  onToggleFavorito={toggleFavorito}
/>
```

### Resultado esperado

Al hacer clic en ★ en cualquier tarjeta, el botón cambia de color. Al hacer clic de nuevo, vuelve al color original. El resto de la app no se ve afectada.

---

## Estructura final del proyecto

```
src/
├── data/
│   └── peliculas.ts       ← NUEVO
├── hooks/
│   ├── useMovies.ts       ← NUEVO
│   └── useFavorites.ts    ← AUTÓNOMO
├── components/
│   ├── MovieCard.tsx      ← MODIFICADO (Tailwind)
│   ├── MovieList.tsx      ← MODIFICADO (Tailwind)
│   ├── HeaderTitle.tsx    ← sin cambios
│   ├── MovieFooter.tsx    ← sin cambios
│   └── MovieSocial.tsx    ← sin cambios
├── page/
│   └── Catalog.tsx        ← MODIFICADO (consume useMovies)
└── types/
    └── Movie.ts           ← sin cambios
```
