# Laboratorio D03 — Listas, `.map()` y composición

**SIS-0300 Programación Web I · Fase 1 · Día 3**
Universidad Privada Domingo Savio · Facultad de Ingeniería

---

## Estándar de nomenclatura del proyecto

| Qué                   | Convención | Ejemplo                             |
| --------------------- | ---------- | ----------------------------------- |
| Componentes y páginas | PascalCase | `MovieCard`, `MovieList`, `Catalog` |
| Variables y funciones | camelCase  | `peliculas`, `handleEliminar`       |
| Tipos e interfaces    | PascalCase | `Movie`, `MovieCardProps`           |
| Comentarios           |            | `// lista de películas`             |

---

## Estructura de archivos del laboratorio completo

```
cine-app/
└── src/
    ├── types/
    │   └── Movie.ts              ← Tarea 0 (clase)
    ├── components/
    │   ├── MovieCard.tsx         ← Tarea 1 (clase)
    │   └── MovieList.tsx         ← Tarea 2 (clase)
    ├── pages/
    │   └── Catalog.tsx           ← Tarea autónoma
    └── App.tsx                   ← Tarea autónoma
```

---

---

# PARTE 1 — Trabajo en clase

---

## Tarea 0 — Crear el tipo Movie

### Qué haces

En D00 practicaste interfaces en archivos sueltos. Ahora creas la primera interface real del proyecto — la que va a representar una película en todo CineApp, desde hoy hasta el final del módulo.

### Por qué un archivo separado

Cuando varios componentes necesitan el mismo tipo, lo mejor es definirlo una sola vez y que todos lo importen desde el mismo lugar. Si mañana el tipo cambia, lo modificas en un solo archivo y TypeScript te avisa en todos los componentes que usan ese tipo.

### Pasos

**0.1** Creá la carpeta `src/types/` y dentro el archivo `Movie.ts`.

**0.2** Escribí la interface:

```ts
// src/types/Movie.ts

// Representa una película dentro del sistema CineApp
export interface Movie {
  id: number;
  titulo: string;
  anio: number;
  genero: string;
  director: string;
  sinopsis: string;
  calificacion: number; // escala 0 a 10
}
```

**0.3** Guardá. El archivo no debe tener errores.

### ¿Qué verificar?

En cualquier otro archivo escribí `import { Movie } from '../types/Movie'` y luego `pelicula.` — el autocompletado de VS Code debe mostrarte los 7 campos. Si los ves, la interface está bien conectada.

---

## Tarea 1 — Actualizar MovieCard

### Qué haces

`MovieCard` ya existe desde D01 pero todavía no usa el tipo `Movie`. Lo actualizas para que reciba una película como prop tipada.

### Pasos

**1.1** Reemplazá el contenido de `src/components/MovieCard.tsx`:

```tsx
// src/components/MovieCard.tsx
import { Movie } from "../types/Movie";

interface MovieCardProps {
  pelicula: Movie;
}

const MovieCard = ({ pelicula }: MovieCardProps) => {
  return (
    <div
      style={{
        border: "1px solid #ccc",
        borderRadius: "8px",
        padding: "16px",
        backgroundColor: "#fff",
      }}
    >
      <h2 style={{ margin: "0 0 4px", color: "#003A61" }}>{pelicula.titulo}</h2>
      <p style={{ margin: "0 0 8px", color: "#666", fontSize: "14px" }}>
        {pelicula.anio} · {pelicula.genero}
      </p>
      <p style={{ margin: "0 0 12px", fontSize: "14px" }}>
        {pelicula.sinopsis}
      </p>
      <p style={{ margin: 0, fontWeight: "bold", color: "#27BCEE" }}>
        ⭐ {pelicula.calificacion} / 10
      </p>
    </div>
  );
};

export default MovieCard;
```

**1.2** Guardá. No debe haber errores de TypeScript.

### ¿Qué verificar?

Probá estos errores intencionalmente, luego deshacelos:

- Escribí `pelicula.presupuesto` — debe decir que esa propiedad no existe.
- Eliminá la prop `pelicula` al usar el componente — debe pedirte que la agregues.

---

## Tarea 2 — Componente MovieList

### Qué haces

Creás el componente que recibe un **array** de películas y las renderiza usando `.map()`. También maneja el caso de lista vacía.

### Pasos

**2.1** Creá `src/components/MovieList.tsx`:

```tsx
// src/components/MovieList.tsx
import { Movie } from "../types/Movie";
import MovieCard from "./MovieCard";

interface MovieListProps {
  peliculas: Movie[];
}

const MovieList = ({ peliculas }: MovieListProps) => {
  // caso vacío — siempre hay que manejarlo
  if (peliculas.length === 0) {
    return (
      <p style={{ textAlign: "center", color: "#999", padding: "40px 0" }}>
        No hay películas para mostrar.
      </p>
    );
  }

  return (
    <div
      style={{
        display: "grid",
        gridTemplateColumns: "repeat(3, 1fr)",
        gap: "16px",
      }}
    >
      {peliculas.map((pelicula) => (
        <MovieCard key={pelicula.id} pelicula={pelicula} />
      ))}
    </div>
  );
};

export default MovieList;
```

**2.2** Guardá. No debe haber errores.

### ¿Qué verificar?

- `key={pelicula.id}` es obligatorio. Probá sacarlo — VS Code debe mostrar una advertencia.
- Este componente usa `MovieCard` internamente. Eso es **composición**: un componente construido a partir de otro.

---

---

# PARTE 2 — Tarea autónoma

---

## Contexto

Ya tienes el tipo `Movie`, el componente `MovieCard` y el componente `MovieList`. Ahora vas a construir la página `Catalog` que los integra todos, agrega un filtro por género y permite eliminar películas.

Leé cada tarea con atención antes de escribir código. Si algo no está claro, revisa lo que hicimos en clase — los patrones que necesitás ya los viste.

---

## Datos iniciales

Usá estos datos para construir el array `peliculasIniciales: Movie[]` en tu `Catalog.tsx`:

| id  | titulo       | anio | genero          | director             | calificacion |
| --- | ------------ | ---- | --------------- | -------------------- | ------------ |
| 1   | Inception    | 2010 | Ciencia Ficción | Christopher Nolan    | 8.8          |
| 2   | El Padrino   | 1972 | Drama           | Francis Ford Coppola | 9.2          |
| 3   | Parasite     | 2019 | Drama           | Bong Joon-ho         | 8.6          |
| 4   | Interstellar | 2014 | Ciencia Ficción | Christopher Nolan    | 8.7          |
| 5   | Coco         | 2017 | Animación       | Lee Unkrich          | 8.4          |
| 6   | Joker        | 2019 | Drama           | Todd Phillips        | 8.4          |

> La sinopsis de cada película la escribes tú — una oración es suficiente.

---

## Tarea 3 — Página Catalog con filtro

### Qué tienes que lograr

Una página que muestre las 6 películas en una grilla de 3 columnas con botones para filtrar por género.

### Requisitos

- Creá `src/pages/Catalog.tsx`
- El array `peliculasIniciales` debe vivir en `useState<Movie[]>`
- Los géneros disponibles son `'Todos'`, `'Drama'`, `'Ciencia Ficción'`, `'Animación'` — hardcodeados en un array
- Al hacer clic en un botón de género, la lista muestra solo las películas de ese género
- Al hacer clic en `'Todos'` aparecen todas
- Debe haber un contador que muestre cuántas películas se están mostrando
- Actualizá `App.tsx` para que renderice `Catalog`

### ¿Qué verificar?

- Al seleccionar "Animación" aparece solo Coco
- El contador cambia con cada filtro
- Al seleccionar un género sin resultados aparece el mensaje vacío de `MovieList`

---

## Tarea 4 — Eliminar película (lift state up)

### Qué tienes que lograr

Un botón "Eliminar" en cada `MovieCard` que quite esa película de la lista.

### Requisitos

- El estado `peliculas` sigue viviendo en `Catalog` — no lo muevas
- `MovieCard` debe recibir una prop `onEliminar: (id: number) => void`
- `MovieList` debe recibir y reenviar esa prop a cada `MovieCard`
- Al eliminar una película el contador debe actualizarse
- Si eliminás todas las películas de un género y filtrás por ese género, debe aparecer el mensaje vacío

### Pista — diagrama del flujo

```
Catalog
  │  estado: peliculas[]
  │  función: handleEliminar(id)
  │
  └── MovieList
        │  prop: onEliminar
        │
        └── MovieCard
              botón onClick → llama onEliminar(pelicula.id)
                                    ↑
                        sube hasta Catalog y actualiza el estado
```

---

## Tarea 5 — Extra ⭐

> No es obligatoria.

Agregá un botón **"Restablecer"** en `Catalog` que restaure la lista completa y resetee el filtro a `'Todos'`, sin recargar la página.

---

## Preguntas de reflexión

Respondé en un comentario al inicio de `Catalog.tsx` antes de entregar:

1. ¿Por qué `peliculasIniciales` y `generos` se declaran **fuera** del componente y no adentro?
2. `handleEliminar` vive en `Catalog` pero se ejecuta desde `MovieCard`. ¿Cómo llega hasta ahí? ¿Qué nombre recibe ese patrón?
3. Si aplicás el filtro "Drama" y eliminás una película de Drama, ¿el contador muestra el número correcto? ¿Por qué?
