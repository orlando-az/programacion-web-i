# Laboratorio D07 — React Router v7: Navegación y Rutas Protegidas

**SIS-0300 Programación Web I · Fase 3 · Día 7**
Universidad Privada Domingo Savio · Facultad de Ingeniería

---

## Dónde estamos

| Día | Tema | Estado |
|-----|------|--------|
| D05 | Supabase — configuración y primera consulta | ✅ Completado |
| D06 | CRUD completo | ✅ Completado |
| **D07** | **React Router v7 — navegación y rutas protegidas** | ← Hoy |

Al terminar D06 tenemos: CRUD completo sobre `peliculas`, hook `useMovie` con las cuatro operaciones, `MovieForm`, `MovieCard` y `Catalog.tsx` conectando todo desde una sola pantalla.

**El problema:** toda la app vive en una URL. No hay forma de compartir un enlace a una película, ni de tener páginas separadas para distintas funciones. Hoy resolvemos eso.

---

## Qué construimos hoy

Arrancamos con una sola pantalla y terminamos con una aplicación con navegación real:

```
/              →  Catálogo de películas       (pública)
/pelicula/:id  →  Detalle de una película     (pública)
/login         →  Formulario de login         (pública)
/admin         →  Panel de administración     (protegida — solo con sesión)
```

---

## Estructura de archivos al finalizar

```
cine-app/
└── src/
    ├── lib/
    │   └── supabaseClient.ts          ← sin cambios
    ├── types/
    │   └── Movie.ts                   ← sin cambios
    ├── hooks/
    │   └── useMovie.ts                ← sin cambios
    ├── components/
    │   ├── MovieCard.tsx              ← actualizado (botón Ver detalle)
    │   ├── MovieForm.tsx              ← sin cambios
    │   ├── MovieList.tsx              ← actualizado (pasa onVerDetalle)
    │   ├── Navbar.tsx                 ← NUEVO
    │   └── ProtectedRoute.tsx         ← NUEVO
    ├── pages/
    │   ├── Catalog.tsx                ← actualizado (Navbar + useNavigate)
    │   ├── MovieDetail.tsx            ← NUEVO
    │   ├── Login.tsx                  ← NUEVO
    │   └── Admin.tsx                  ← NUEVO
    └── main.tsx                       ← actualizado en dos momentos
```

---

## Estándar de código del día

| Qué | Convención |
|-----|-----------|
| Componentes | `const NombreComponente = (props: Props) => { return (...) }` |
| Rutas | definidas en `main.tsx`, una por página |
| Navegación programática | `useNavigate` dentro de handlers |
| Parámetros de URL | `useParams` tipado con `{ id: string }` |

---

## Concepto previo — El problema de la URL única

Sin Router, React muestra siempre el mismo componente sin importar lo que diga la URL. Podés escribir `/pelicula/5` en el navegador y React igual muestra el catálogo completo — no sabe qué hacer con esa URL.

React Router intercepta la URL y decide qué componente renderizar:

```
Sin Router:   cualquier URL  →  siempre <Catalog />

Con Router:   /              →  <Catalog />
              /pelicula/5    →  <MovieDetail /> con id = 5
              /login         →  <Login />
              /admin         →  <Admin /> (si hay sesión) o redirect a /login
```

Los tres hooks que usás hoy:

```ts
// Navegar a otra ruta desde código (dentro de un handler)
const navigate = useNavigate();
navigate("/catalog");
navigate(-1);           // equivale al botón "atrás" del navegador

// Leer parámetros de la URL — /pelicula/:id → id = "5" (siempre string)
const { id } = useParams<{ id: string }>();

// Redirigir sin dejar rastro en el historial
<Navigate to="/login" replace />
```

Y el componente para enlaces:

```tsx
// <a href> recarga la página completa — React pierde todo el estado
// <Link to> cambia la URL sin recargar — el estado se mantiene
<Link to="/catalog">Ver catálogo</Link>
```

---

# PARTE 1 — Trabajo en clase

---

# BLOQUE 1 — Instalación y primera navegación

En este bloque instalamos React Router, creamos la barra de navegación y configuramos las rutas públicas. Al terminar ya podés moverte entre páginas.

---

## Tarea 1.1 — Instalar react-router

```bash
npm install react-router
```

Verificar en `package.json` que aparece `"react-router"` en `dependencies`.

> **React Router v7:** a partir de esta versión el paquete `react-router-dom` fue eliminado. Todo se importa desde `react-router`. Los componentes y hooks son los mismos que en v6.

---

## Tarea 1.2 — Crear `Navbar.tsx`

Antes de definir las rutas necesitamos la barra de navegación, porque cada página la va a incluir.

### Por qué `<Link>` y no `<a>`

Un `<a href="/catalog">` recarga la página completa — React pierde todo el estado (películas cargadas, formularios abiertos, etc.). `<Link to="/catalog">` intercepta el clic y actualiza solo la URL, sin recargar. El estado se mantiene.

### Código

```tsx
// src/components/Navbar.tsx
// Barra de navegación principal de CineApp

import { Link } from "react-router";

const Navbar = () => {
  return (
    <nav className="bg-gray-900 border-b border-gray-700 px-6 py-3
                    flex items-center justify-between">
      {/* Logo — vuelve al catálogo */}
      <Link to="/" className="text-white text-xl font-bold tracking-tight">
        🎬 CineApp
      </Link>

      {/* Links de navegación */}
      <div className="flex items-center gap-6">
        <Link
          to="/"
          className="text-gray-300 hover:text-white text-sm transition-colors"
        >
          Catálogo
        </Link>
        <Link
          to="/login"
          className="bg-blue-600 hover:bg-blue-500 text-white text-sm
                     font-semibold px-4 py-1.5 rounded-lg transition-colors"
        >
          Iniciar sesión
        </Link>
      </div>
    </nav>
  );
};

export default Navbar;
```

---

## Tarea 1.3 — Configurar rutas públicas en `main.tsx`

Ahora conectamos el Router. Por ahora solo definimos las rutas públicas — las protegidas las agregamos en el Bloque 3 una vez que tengamos todo lo que necesitan.

### Por qué `BrowserRouter` va en `main.tsx` y no en `App.tsx`

`BrowserRouter` debe envolver toda la aplicación. Ponerlo en `main.tsx` garantiza que cualquier componente en cualquier nivel pueda usar `useNavigate`, `useParams` o `<Link>` sin restricciones.

### Código — primera versión de `main.tsx` (solo rutas públicas)

```tsx
// src/main.tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter, Routes, Route } from "react-router";
import Catalog from "./pages/Catalog";
import MovieDetail from "./pages/MovieDetail";
import Login from "./pages/Login";
import "./index.css";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Catalog />} />
        <Route path="/pelicula/:id" element={<MovieDetail />} />
        <Route path="/login" element={<Login />} />
      </Routes>
    </BrowserRouter>
  </StrictMode>
);
```

> `MovieDetail` y `Login` todavía no existen — el proyecto va a mostrar errores de compilación hasta que los creemos en los bloques siguientes. Eso es normal.

---

## Tarea 1.4 — Agregar `Navbar` a `Catalog.tsx`

Todas las páginas siguen este patrón: `<Navbar />` al inicio, contenido debajo.

```tsx
// Agregar el import en Catalog.tsx
import Navbar from "../components/Navbar";

// Envolver el JSX existente
return (
  <div className="min-h-screen bg-gray-950">
    <Navbar />
    {/* resto del JSX que ya tenías */}
  </div>
);
```

---

# BLOQUE 2 — Páginas públicas y parámetros de URL

Con la navegación funcionando, creamos las páginas que faltan. Empezamos por las más simples y terminamos con `MovieDetail`, que introduce `useParams`.

---

## Tarea 2.1 — Crear `Login.tsx`

Por ahora es una página estática — la conectamos con Supabase Auth en D08. Hoy solo necesitamos que la ruta `/login` exista para poder redirigir a ella.

```tsx
// src/pages/Login.tsx
// Página de inicio de sesión — se conecta con Supabase Auth en D08

import { useState } from "react";
import { useNavigate } from "react-router";
import Navbar from "../components/Navbar";

const Login = () => {
  const navigate = useNavigate();
  const [email, setEmail] = useState<string>("");
  const [password, setPassword] = useState<string>("");

  const handleSubmit = () => {
    // Simulación temporal — en D08 se reemplaza con supabase.auth.signInWithPassword()
    if (email.trim() && password.trim()) {
      alert("Login simulado. D08 implementa Auth real con Supabase.");
      navigate("/");
    }
  };

  return (
    <div className="min-h-screen bg-gray-950">
      <Navbar />
      <div className="flex items-center justify-center mt-20 px-4">
        <div className="bg-gray-800 rounded-xl p-8 w-full max-w-sm">
          <h2 className="text-white text-2xl font-bold mb-6 text-center">
            Iniciar sesión
          </h2>

          <div className="mb-4">
            <label className="block text-gray-300 text-sm mb-1">Email</label>
            <input
              type="email"
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              className="w-full bg-gray-700 text-white rounded-lg px-3 py-2 text-sm
                         border border-gray-600 focus:outline-none focus:border-blue-400"
              placeholder="usuario@ejemplo.com"
            />
          </div>

          <div className="mb-6">
            <label className="block text-gray-300 text-sm mb-1">Contraseña</label>
            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              className="w-full bg-gray-700 text-white rounded-lg px-3 py-2 text-sm
                         border border-gray-600 focus:outline-none focus:border-blue-400"
              placeholder="••••••••"
            />
          </div>

          <button
            onClick={handleSubmit}
            className="w-full bg-blue-600 hover:bg-blue-500 text-white font-semibold
                       py-2 rounded-lg text-sm transition-colors"
          >
            Ingresar
          </button>
        </div>
      </div>
    </div>
  );
};

export default Login;
```

### ✅ Verificar

- `npm run dev` compila sin errores
- Clic en "Iniciar sesión" en la Navbar → navega a `/login` sin recargar
- El logo 🎬 vuelve al catálogo

---

## Tarea 2.2 — Conectar "Ver detalle" en los tres archivos

Antes de crear `MovieDetail.tsx` necesitamos una forma de llegar a ella. El handler nace en `Catalog`, pasa por `MovieList` y llega a `MovieCard` — el mismo patrón de callbacks que usaste en D06 con `onEditar` y `onEliminar`.

### Paso 1 — `MovieCard.tsx`

Agregar `onVerDetalle` a la interfaz y el botón en el JSX:

```tsx
interface MovieCardProps {
  pelicula: Movie;
  onEditar: (pelicula: Movie) => void;
  onEliminar: (id: number) => void;
  onVerDetalle: (id: number) => void;   // ← agregar
}

const MovieCard = ({ pelicula, onEditar, onEliminar, onVerDetalle }: MovieCardProps) => {
  // ...
  // Agregar este botón junto a Editar y Eliminar
  <button
    onClick={() => onVerDetalle(pelicula.id)}
    className="w-full py-1.5 rounded-lg text-xs font-medium
               bg-blue-900 text-blue-200 hover:bg-blue-700 hover:text-white
               transition-colors"
  >
    👁 Ver detalle
  </button>
}
```

### Paso 2 — `MovieList.tsx`

`MovieList` es el intermediario — recibe la prop y la reenvía a cada `MovieCard`. También corregimos el `key` para usar `p.id` en lugar de `p.titulo` (el título puede repetirse, el id no):

```tsx
interface MovieListProps {
  peliculas: Movie[];
  onEditar: (pelicula: Movie) => void;
  onEliminar: (id: number) => void;
  onVerDetalle: (id: number) => void;   // ← agregar
}

const MovieList = ({ onEditar, onEliminar, onVerDetalle, peliculas }: MovieListProps) => {
  return (
    <div className="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 gap-4">
      {peliculas.map((p) => (
        <MovieCard
          key={p.id}                              // ← corregido: era p.titulo
          pelicula={p}
          onEditar={() => onEditar(p)}
          onEliminar={() => onEliminar(p.id)}
          onVerDetalle={() => onVerDetalle(p.id)} // ← agregar
        />
      ))}
    </div>
  );
};

export default MovieList;
```

### Paso 3 — `Catalog.tsx`

Crear el handler con `useNavigate` y pasarlo a `MovieList`:

```tsx
// Agregar el import
import { useNavigate } from "react-router";

// Dentro del componente — agregar junto a los otros handlers
const navigate = useNavigate();

const handleVerDetalle = (id: number) => {
  navigate(`/pelicula/${id}`);
};

// En el JSX — agregar la prop a MovieList
<MovieList
  peliculas={peliculasFiltradas}
  onEditar={handleEditarPelicula}
  onEliminar={handleEliminar}
  onVerDetalle={handleVerDetalle}   // ← agregar
/>
```

---

## Tarea 2.3 — Crear `MovieDetail.tsx`

Esta página recibe el `id` desde la URL y carga los datos de esa película desde Supabase.

### Por qué `useParams` devuelve `string`

La URL es texto. `/pelicula/5` da `id = "5"` como string, aunque en la base de datos sea un número. Hay que convertirlo con `parseInt` antes de pasarlo a Supabase.

### Por qué `useEffect` depende de `[id]`

Si el usuario navega de `/pelicula/5` a `/pelicula/8`, el componente no se desmonta — React reutiliza el mismo componente con un `id` diferente. Sin `[id]` en el array de dependencias, el efecto no se volvería a ejecutar y quedaría mostrando la película anterior.

### Código

```tsx
// src/pages/MovieDetail.tsx
// Página de detalle — carga una película por su id desde la URL

import { useEffect, useState } from "react";
import { useParams, useNavigate } from "react-router";
import { supabase } from "../lib/supabaseClient";
import type { Movie } from "../types/Movie";
import Navbar from "../components/Navbar";

const MovieDetail = () => {
  const { id } = useParams<{ id: string }>();
  const navigate = useNavigate();
  const [pelicula, setPelicula] = useState<Movie | null>(null);
  const [cargando, setCargando] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const cargarPelicula = async () => {
      if (!id) return;

      setCargando(true);
      setError(null);

      const { data, error: supabaseError } = await supabase
        .from("peliculas")
        .select("*")
        .eq("id", parseInt(id))
        .single();

      if (supabaseError) {
        setError("Película no encontrada.");
      } else {
        setPelicula(data as Movie);
      }

      setCargando(false);
    };

    cargarPelicula();
  }, [id]); // se re-ejecuta si el id cambia en la URL

  return (
    <div className="min-h-screen bg-gray-950">
      <Navbar />
      <div className="max-w-2xl mx-auto px-6 py-10">

        {cargando && (
          <p className="text-gray-400 text-center">Cargando...</p>
        )}

        {error && (
          <div className="bg-red-900 text-red-200 rounded-xl p-4 text-sm">
            {error}
          </div>
        )}

        {pelicula && (
          <div className="bg-gray-800 rounded-xl p-8">
            <h1 className="text-white text-3xl font-bold mb-2">
              {pelicula.titulo}
            </h1>
            <p className="text-gray-400 mb-1">
              {pelicula.anio} · {pelicula.genero}
            </p>
            {pelicula.director && (
              <p className="text-gray-500 text-sm mb-4">
                Dirección: {pelicula.director}
              </p>
            )}
            {pelicula.sinopsis && (
              <p className="text-gray-300 text-sm leading-relaxed mb-6">
                {pelicula.sinopsis}
              </p>
            )}
            <p className="text-yellow-400 text-lg font-semibold">
              ⭐ {pelicula.calificacion?.toFixed(1) ?? "—"} / 10
            </p>

            <button
              onClick={() => navigate(-1)}
              className="mt-6 text-sm text-gray-400 hover:text-white transition-colors"
            >
              ← Volver
            </button>
          </div>
        )}

      </div>
    </div>
  );
};

export default MovieDetail;
```

### ✅ Verificar rutas públicas completas

```bash
npm run dev
```

- [ ] `/` muestra el catálogo con películas de Supabase
- [ ] Clic en "👁 Ver detalle" navega a `/pelicula/:id`
- [ ] Los datos de la película se cargan correctamente
- [ ] "← Volver" regresa al catálogo
- [ ] `/login` muestra el formulario de inicio de sesión
- [ ] En ningún momento se recarga la página completa

---

# BLOQUE 3 — Rutas protegidas

Con la navegación funcionando y la página `/login` existente, ahora tiene sentido proteger rutas. Si alguien intenta acceder a `/admin` sin sesión, lo mandamos a `/login` — que ya existe.

---

## Tarea 3.1 — Crear `Admin.tsx`

Primero creamos la página protegida. Es simple por ahora — en D08 va a mostrar funcionalidades reales.

```tsx
// src/pages/Admin.tsx
// Panel de administración — solo accesible con sesión activa

import Navbar from "../components/Navbar";

const Admin = () => {
  return (
    <div className="min-h-screen bg-gray-950">
      <Navbar />
      <div className="max-w-4xl mx-auto px-6 py-10">
        <h1 className="text-white text-3xl font-bold mb-2">
          Panel de administración
        </h1>
        <p className="text-gray-400 text-sm mb-8">
          Solo los usuarios con sesión activa pueden ver esta página.
        </p>
        <div className="bg-green-900 text-green-200 rounded-xl p-4 text-sm">
          ✅ Ruta protegida funcionando correctamente.
          En D08 esto verifica sesión real con Supabase Auth.
        </div>
      </div>
    </div>
  );
};

export default Admin;
```

---

## Tarea 3.2 — Crear `ProtectedRoute.tsx`

`ProtectedRoute` es un componente que envuelve otros componentes. Su trabajo es uno solo: verificar si hay sesión activa antes de mostrar el contenido.

### Cómo funciona el patrón `children`

```tsx
// En main.tsx vas a escribir esto:
<ProtectedRoute>
  <Admin />
</ProtectedRoute>

// Dentro de ProtectedRoute, <Admin /> llega como la prop "children"
// Si hay sesión  → renderiza children (Admin)
// Si no hay sesión → redirige a /login
```

### Por qué `replace` en `<Navigate>`

Sin `replace`: el historial queda `[..., /admin, /login]`. Si el usuario presiona "atrás" desde `/login`, vuelve a `/admin` y lo redirigen de nuevo a `/login` — un loop. Con `replace`, `/admin` nunca entra al historial: `[..., /login]`.

### Código

```tsx
// src/components/ProtectedRoute.tsx
// Guarda de rutas — redirige a /login si no hay sesión activa

import { Navigate } from "react-router";

interface ProtectedRouteProps {
  children: React.ReactNode;
}

// Sesión simulada con una constante — en D08 viene de supabase.auth.getSession()
// Cambiá este valor a true/false para probar el comportamiento
const SESION_SIMULADA = false;

const ProtectedRoute = ({ children }: ProtectedRouteProps) => {
  if (!SESION_SIMULADA) {
    // Sin sesión: redirigir a login sin dejar /admin en el historial
    return <Navigate to="/login" replace />;
  }

  // Con sesión: mostrar el contenido protegido
  return <>{children}</>;
};

export default ProtectedRoute;
```

---

## Tarea 3.3 — Agregar la ruta protegida en `main.tsx`

Ahora sí agregamos `/admin` al router. `main.tsx` queda completo con todas las rutas.

### Código final de `main.tsx`

```tsx
// src/main.tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter, Routes, Route } from "react-router";
import Catalog from "./pages/Catalog";
import MovieDetail from "./pages/MovieDetail";
import Login from "./pages/Login";
import Admin from "./pages/Admin";
import ProtectedRoute from "./components/ProtectedRoute";
import "./index.css";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <BrowserRouter>
      <Routes>
        {/* Rutas públicas — accesibles sin sesión */}
        <Route path="/" element={<Catalog />} />
        <Route path="/pelicula/:id" element={<MovieDetail />} />
        <Route path="/login" element={<Login />} />

        {/* Rutas protegidas — requieren sesión activa */}
        <Route
          path="/admin"
          element={
            <ProtectedRoute>
              <Admin />
            </ProtectedRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  </StrictMode>
);
```

### ✅ Verificar rutas protegidas

- [ ] Con `SESION_SIMULADA = false`: navegar a `/admin` redirige automáticamente a `/login`
- [ ] Presionar "atrás" desde `/login` no vuelve a `/admin`
- [ ] Con `SESION_SIMULADA = true`: navegar a `/admin` muestra el panel de administración
- [ ] Dejar `SESION_SIMULADA = false` antes de continuar — D08 implementa la sesión real

---

## Agregar enlace a Admin en `Navbar.tsx`

Con `/admin` funcionando, podemos agregar el enlace en la barra de navegación:

```tsx
// Agregar en Navbar.tsx, dentro del div de links
<Link
  to="/admin"
  className="text-gray-300 hover:text-white text-sm transition-colors"
>
  Administración
</Link>
```

---

## Verificación final — Checklist completo

```bash
npm run dev
```

**Navegación**
- [ ] Los `<Link>` de Navbar cambian la URL sin recargar la página
- [ ] El logo 🎬 siempre vuelve al catálogo

**Rutas públicas**
- [ ] `/` muestra el catálogo con datos de Supabase
- [ ] `/pelicula/:id` carga la película correcta
- [ ] `← Volver` regresa a la pantalla anterior
- [ ] `/login` muestra el formulario

**Rutas protegidas**
- [ ] `SESION_SIMULADA = false` → `/admin` redirige a `/login`
- [ ] `SESION_SIMULADA = true` → `/admin` muestra el panel
- [ ] "Atrás" desde `/login` no hace loop hacia `/admin`

---

# PARTE 2 — Tarea autónoma

---

## Tarea A — Página 404

Creá `src/pages/NotFound.tsx` con un mensaje claro de error. Luego agregá una ruta comodín al **final** de `main.tsx` (debe ser la última):

```tsx
<Route path="*" element={<NotFound />} />
```

Verificá que navegar a `/pagina-inventada` muestra tu página en lugar de pantalla en blanco.

---

## Tarea B — NavLink con estilo activo

Reemplazá los `<Link>` de `Navbar.tsx` por `<NavLink>`. La diferencia es que `NavLink` recibe una función en `className` que sabe si esa ruta está activa:

```tsx
<NavLink
  to="/"
  className={({ isActive }) =>
    isActive
      ? "text-white font-semibold border-b border-blue-400 pb-0.5"
      : "text-gray-300 hover:text-white"
  }
>
  Catálogo
</NavLink>
```

Verificá que al estar en `/login` el enlace "Catálogo" se ve diferente al de "Iniciar sesión".

---

## Tarea C — `useSearchParams`

Investigá `useSearchParams`. Modificá `Catalog.tsx` para que el texto de búsqueda se guarde en la URL como `?q=interstellar`. Así al recargar o compartir la URL el filtro se mantiene.

```tsx
// Pista de inicio
import { useSearchParams } from "react-router";
const [searchParams, setSearchParams] = useSearchParams();
const busqueda = searchParams.get("q") ?? "";
```

---

## ¿Qué aprendiste hoy?

| Concepto | Dónde lo aplicaste |
|----------|-------------------|
| `<BrowserRouter>` | `main.tsx` — envuelve toda la app |
| `<Routes>` y `<Route>` | `main.tsx` — mapea cada URL a un componente |
| `<Link>` | `Navbar` — navegación sin recarga de página |
| `<NavLink>` | Tarea B — enlace con estilo según ruta activa |
| `useNavigate` | `Catalog`, `Login`, `MovieDetail` |
| `navigate(-1)` | `MovieDetail` — botón Volver |
| `useParams<{ id: string }>` | `MovieDetail` — leer `:id` de la URL |
| `parseInt(id)` | `MovieDetail` — convertir id de string a número |
| `useEffect` con `[id]` | `MovieDetail` — recargar al cambiar de película |
| `<Navigate replace>` | `ProtectedRoute` — redirigir sin historial |
| `children` como prop | `ProtectedRoute` envuelve `<Admin />` |
| Ruta comodín `*` | Tarea A — página 404 |
| `useSearchParams` | Tarea C — filtro persistente en la URL |
| Rutas en dos momentos | `main.tsx` primero sin protegidas, después completo |

---

*SIS-0300 · Universidad Privada Domingo Savio · Tarija, Bolivia*
