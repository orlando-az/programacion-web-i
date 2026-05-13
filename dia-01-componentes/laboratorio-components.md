# Laboratorio - Componentes, TSX y Props

**Asignatura:** Programación Web I — SIS-0300  
**Carrera:** Ingeniería de Sistemas · UPDS Tarija  
**Proyecto:** CineApp · Fase 1

---

## Objetivo

Construir la primera pantalla de CineApp escribiendo todo en un solo archivo, identificar el problema del código repetido y resolverlo extrayendo el primer componente reutilizable.

---

# Configuración inicial

Abrir VSCode y la terminal integrada con `Ctrl + J`.

## Paso 1 — Crear el proyecto

```bash
npm create vite@latest
```

Vite lanzará un asistente interactivo.

Responder:

```txt
√ Project name: cine-app
√ Select a framework: React
√ Select a variant: TypeScript
```

Instalar dependencias:

```bash
cd cine-app
npm install
```

---

## Paso 2 — Revisar los archivos principales

Antes de modificar código, revisar:

- `index.html` → contiene `<div id="root">`
- `src/main.tsx` → punto de entrada de React
- `src/App.tsx` → componente principal
- `tsconfig.json` → configuración TypeScript
- `vite.config.ts` → configuración de Vite

---

## Paso 3 — Crear estructura de carpetas

```bash
mkdir src/components src/pages src/hooks src/lib src/services
```

---

## Paso 4 — Ejecutar el proyecto

```bash
npm run dev
```

Abrir:

```txt
http://localhost:5173
```

No continuar hasta verificar que funciona correctamente.

---

# Lab A — Todo en un solo archivo

Por ahora no vamos a crear componentes.

Toda la interfaz estará dentro de `App.tsx`.

Reemplazar todo el contenido del archivo:

```tsx
const App = () => {
  return (
    <div
      style={{
        fontFamily: "Arial",
        backgroundColor: "#f8fafc",
        minHeight: "100vh",
      }}
    >
      {/* Header */}
      <header
        style={{
          backgroundColor: "#003A61",
          color: "white",
          padding: "1rem",
          marginBottom: "1.5rem",
        }}
      >
        <h1 style={{ margin: 0 }}>CineApp</h1>

        <p
          style={{
            marginTop: "0.25rem",
            fontSize: "0.9rem",
          }}
        >
          Tu catálogo de películas favoritas
        </p>
      </header>

      {/* Contenedor */}
      <div
        style={{
          padding: "0 1rem",
        }}
      >
        {/* Grid */}
        <div
          style={{
            display: "grid",
            gridTemplateColumns: "repeat(3, 1fr)",
            gap: "1rem",
          }}
        >
          {/* Película 1 */}
          <div
            style={{
              backgroundColor: "white",
              border: "1px solid #d1d5db",
              padding: "1rem",
            }}
          >
            <h3 style={{ marginTop: 0 }}>Interstellar</h3>

            <p
              style={{
                color: "#64748b",
                fontSize: "0.9rem",
              }}
            >
              2014 · Ciencia ficción
            </p>

            <p style={{ color: "green", marginBottom: 0 }}>⭐ 8.7 / 10</p>
          </div>

          {/* Película 2 */}
          <div
            style={{
              backgroundColor: "white",
              border: "1px solid #d1d5db",
              padding: "1rem",
            }}
          >
            <h3 style={{ marginTop: 0 }}>El Padrino</h3>

            <p
              style={{
                color: "#64748b",
                fontSize: "0.9rem",
              }}
            >
              1972 · Drama
            </p>

            <p style={{ color: "green", marginBottom: 0 }}>⭐ 9.2 / 10</p>
          </div>

          {/* Película 3 */}
          <div
            style={{
              backgroundColor: "white",
              border: "1px solid #d1d5db",
              padding: "1rem",
            }}
          >
            <h3 style={{ marginTop: 0 }}>Avengers: Endgame</h3>

            <p
              style={{
                color: "#64748b",
                fontSize: "0.9rem",
              }}
            >
              2019 · Acción
            </p>

            <p style={{ color: "green", marginBottom: 0 }}>⭐ 8.4 / 10</p>
          </div>

          {/* Película 4 */}
          <div
            style={{
              backgroundColor: "white",
              border: "1px solid #d1d5db",
              padding: "1rem",
            }}
          >
            <h3 style={{ marginTop: 0 }}>The Dark Knight</h3>

            <p
              style={{
                color: "#64748b",
                fontSize: "0.9rem",
              }}
            >
              2008 · Acción
            </p>

            <p style={{ color: "green", marginBottom: 0 }}>⭐ 9.0 / 10</p>
          </div>

          {/* Película 5 */}
          <div
            style={{
              backgroundColor: "white",
              border: "1px solid #d1d5db",
              padding: "1rem",
            }}
          >
            <h3 style={{ marginTop: 0 }}>Parasite</h3>

            <p
              style={{
                color: "#64748b",
                fontSize: "0.9rem",
              }}
            >
              2019 · Thriller
            </p>

            <p style={{ color: "green", marginBottom: 0 }}>⭐ 8.5 / 10</p>
          </div>

          {/* Película 6 */}
          <div
            style={{
              backgroundColor: "white",
              border: "1px solid #d1d5db",
              padding: "1rem",
            }}
          >
            <h3 style={{ marginTop: 0 }}>Forrest Gump</h3>

            <p
              style={{
                color: "#64748b",
                fontSize: "0.9rem",
              }}
            >
              1994 · Drama
            </p>

            <p style={{ color: "orange", marginBottom: 0 }}>⭐ 5.8 / 10</p>
          </div>
        </div>
      </div>
    </div>
  );
};

export default App;
```

Verificar que las 6 tarjetas aparecen correctamente.

---

# La pregunta clave

Observar el código y responder:

- ¿Cuántas veces repetiste la tarjeta?
- ¿Qué pasa si ahora necesitas 50 películas?
- ¿Qué pasa si quieres cambiar el diseño?
- ¿Qué problema genera copiar y pegar tantas veces?

Ese es el problema que resuelven los componentes.

---

# Lab B — Extraer MovieCard

Ahora vamos a crear un componente reutilizable.

---

## Paso 1 — Crear MovieCard.tsx

Crear:

```txt
src/components/MovieCard.tsx
```

Código:

```tsx
interface MovieCardProps {
  titulo: string;
  anio: number;
  genero: string;
  calificacion: number;
}

const MovieCard = ({ titulo, anio, genero, calificacion }: MovieCardProps) => {
  const colorCalificacion =
    calificacion >= 8 ? "green" : calificacion >= 6 ? "orange" : "red";

  return (
    <div
      style={{
        backgroundColor: "white",
        border: "1px solid #d1d5db",
        padding: "1rem",
      }}
    >
      <h3 style={{ marginTop: 0 }}>{titulo}</h3>

      <p
        style={{
          color: "#64748b",
          fontSize: "0.9rem",
        }}
      >
        {anio} · {genero}
      </p>

      <p
        style={{
          color: colorCalificacion,
          marginBottom: 0,
        }}
      >
        ⭐ {calificacion} / 10
      </p>
    </div>
  );
};

export default MovieCard;
```

---

## Paso 2 — Actualizar App.tsx

```tsx
import MovieCard from "./components/MovieCard";

const App = () => {
  return (
    <div
      style={{
        fontFamily: "Arial",
        backgroundColor: "#f8fafc",
        minHeight: "100vh",
      }}
    >
      <header
        style={{
          backgroundColor: "#003A61",
          color: "white",
          padding: "1rem",
          marginBottom: "1.5rem",
        }}
      >
        <h1 style={{ margin: 0 }}>CineApp</h1>

        <p
          style={{
            marginTop: "0.25rem",
            fontSize: "0.9rem",
          }}
        >
          Tu catálogo de películas favoritas
        </p>
      </header>

      <div style={{ padding: "0 1rem" }}>
        <div
          style={{
            display: "grid",
            gridTemplateColumns: "repeat(3, 1fr)",
            gap: "1rem",
          }}
        >
          <MovieCard
            titulo="Interstellar"
            anio={2014}
            genero="Ciencia ficción"
            calificacion={8.7}
          />

          <MovieCard
            titulo="El Padrino"
            anio={1972}
            genero="Drama"
            calificacion={9.2}
          />

          <MovieCard
            titulo="Avengers: Endgame"
            anio={2019}
            genero="Acción"
            calificacion={8.4}
          />

          <MovieCard
            titulo="The Dark Knight"
            anio={2008}
            genero="Acción"
            calificacion={9.0}
          />

          <MovieCard
            titulo="Parasite"
            anio={2019}
            genero="Thriller"
            calificacion={8.5}
          />

          <MovieCard
            titulo="Forrest Gump"
            anio={1994}
            genero="Drama"
            calificacion={5.8}
          />
        </div>
      </div>
    </div>
  );
};

export default App;
```

Verificar que el resultado visual sea igual.

---

## Paso 3 — Comparar

- Antes: muchas líneas repetidas
- Después: un componente reutilizable
- Si cambia el diseño: solo modificas `MovieCard.tsx`

---

# Lab C — TypeScript en vivo

Provocar errores para observar cómo TypeScript ayuda al desarrollador.

---

## Error 1 — Tipo incorrecto

```tsx
<MovieCard
  titulo="Interstellar"
  anio={2014}
  genero="Ciencia ficción"
  calificacion="ocho"
/>
```

Leer el mensaje de error.

---

## Error 2 — Prop faltante

```tsx
<MovieCard titulo="Interstellar" anio={2014} genero="Ciencia ficción" />
```

Leer el mensaje de error.

---

## Error 3 — Hot Reload

Cambiar:

```tsx
calificacion={9.2}
```

por:

```tsx
calificacion={4.0}
```

Guardar y observar el cambio automático en pantalla.

---

# Ejercicio autónomo

> A partir de acá trabajamos solos

---

## 1

Agregar:

```tsx
director: string;
```

a `MovieCardProps`.

Mostrar el director en la tarjeta.

Actualizar todas las películas.

Agregar:

```tsx
esFavorita: boolean;
```

Mostrar ❤️ si la película es favorita.

---

## 2

Agregar dos películas más al grid.

Usar películas con distintas calificaciones para probar los colores.

---

## 3

Crear:

```txt
src/components/Header.tsx
```

Props:

```tsx
titulo: string;
subtitulo: string;
```

Reemplazar el `<header>` de `App.tsx`.

---

## 4

Provocar un error quitando `subtitulo`.

Leer el mensaje de TypeScript.

---

# Avance autónomo

## 1

Investigar:

- ¿Qué es `useState`?
- ¿Para qué sirve?
- ¿Cómo funciona en React?
