# Laboratorio D07 — CRUD Completo con Supabase

**SIS-0300 Programación Web I · Fase 2 · Día 7**
Universidad Privada Domingo Savio · Facultad de Ingeniería

---

## Dónde estamos

| Día | Tema | Estado |
|-----|------|--------|
| D05 | Tailwind CSS y custom hooks | ✅ Completado |
| D06 | Supabase — configuración y primera consulta | ✅ Completado |
| **D07** | **CRUD completo — crear, leer, actualizar, eliminar** | ← Hoy |

Al terminar D06 tenemos: tabla `peliculas` en Supabase, RLS con política SELECT pública, cliente centralizado en `supabaseClient.ts`, y el hook `useMovie` que carga y muestra películas en `Catalog`.

**Hoy agregamos las cuatro operaciones completas sobre esa misma tabla.**

---

## Estructura de archivos al finalizar el laboratorio

```
cine-app/
└── src/
    ├── lib/
    │   └── supabaseClient.ts          ← sin cambios
    ├── types/
    │   └── Movie.ts                   ← sin cambios
    ├── hooks/
    │   └── useMovie.ts                ← actualizado con CRUD completo
    ├── components/
    │   ├── MovieCard.tsx              ← actualizado (botón eliminar)
    │   ├── MovieForm.tsx              ← NUEVO — formulario crear/editar
    │   └── MovieList.tsx              ← sin cambios
    └── pages/
        └── Catalog.tsx                ← actualizado (conecta todo)
```

---

## Estándar de código del día

| Qué | Convención |
|-----|-----------|
| Componentes | `const NombreComponente = (props: Props) => { return (...) }` |
| Funciones handler | `handleCrear`, `handleActualizar`, `handleEliminar` |
| Errores | siempre mostrar en la UI, nunca solo en consola |

---

## Tarea 0 — Ampliar las políticas RLS

En D06 solo creaste la política SELECT. Necesitás agregar las políticas para INSERT, UPDATE y DELETE.

**0.1** En Supabase → **Authentication → Policies** → tabla `peliculas`.

**0.2** Hacé clic en **New policy** → **Create a policy from scratch** para cada una:

| Policy name | Allowed operation | USING expression | WITH CHECK expression |
|-------------|------------------|------------------|-----------------------|
| `insercion publica` | INSERT | *(vacío)* | `true` |
| `actualizacion publica` | UPDATE | `true` | `true` |
| `eliminacion publica` | DELETE | `true` | *(vacío)* |

**0.3** Verificá que la tabla `peliculas` muestra **4 políticas** activas.

---

## Concepto previo — Las cuatro operaciones CRUD en Supabase JS

```ts
// CREATE
const { data, error } = await supabase
  .from("peliculas")
  .insert({ titulo: "...", anio: 2024, genero: "..." })
  .select()    // devuelve la fila insertada con su id asignado
  .single();   // espera exactamente 1 resultado

// READ — ya lo vimos en D06
const { data, error } = await supabase
  .from("peliculas")
  .select("*")
  .order("calificacion", { ascending: false });

// UPDATE
const { error } = await supabase
  .from("peliculas")
  .update({ titulo: "nuevo título" })
  .eq("id", 5);   // solo la fila con id = 5

// DELETE
const { error } = await supabase
  .from("peliculas")
  .delete()
  .eq("id", 5);
```

> **Regla de oro:** Sin `.eq()`, `.update()` y `.delete()` afectan TODAS las filas — Supabase lo bloquea por defecto.

---

# PARTE 1 — Trabajo en clase

---

# BLOQUE 1 — CREATE (Guardar)

En este bloque implementamos la operación de crear una película de punta a punta: desde el hook hasta el formulario visible en pantalla.

---

## Tarea 1.1 — Agregar `crearPelicula` a `useMovie.ts`

### Qué haces

Agregás el tipo `MovieInput` y la función `crearPelicula` al hook existente. El READ que ya tenías no cambia.

### Por qué solo el READ va dentro de `useEffect`

`useEffect` ejecuta código al montar el componente — es el lugar correcto para cargar datos automáticamente. CREATE en cambio debe ejecutarse cuando el usuario hace una acción, no al cargar, por eso va como función normal fuera del efecto.

### Por qué `setPeliculas((prev) => ...)`

`setPeliculas` es asíncrono. Si usás el valor directo `[...peliculas]` podés leer un estado desactualizado si ocurrió otro cambio mientras Supabase respondía. Con el callback `(prev) => ...` React garantiza que `prev` siempre es el valor más reciente.

### Código — agregar al final de `useMovie.ts` antes del `return`

```ts
// src/hooks/useMovie.ts
// Agregar el tipo MovieInput arriba, junto a los imports
export type MovieInput = Omit<Movie, "id">;

// Agregar esta función antes del return del hook
const crearPelicula = async (nueva: MovieInput): Promise<boolean> => {
  setError(null);

  const { data, error: supabaseError } = await supabase
    .from("peliculas")
    .insert(nueva)
    .select()
    .single();

  if (supabaseError) {
    setError("Error al crear: " + supabaseError.message);
    return false;
  }

  // agregar al inicio sin recargar toda la lista
  setPeliculas((prev) => [data as Movie, ...prev]);
  return true;
};
```

### Agregar `crearPelicula` al return del hook

```ts
return {
  peliculas,
  cargando,
  error,
  crearPelicula,   // ← agregar
};
```

---

## Tarea 1.2 — Crear `MovieForm.tsx`

### Qué haces

Un formulario controlado para ingresar los datos de una película nueva. Por ahora solo funciona en modo creación — el modo edición lo agregamos en el Bloque 2.

### Código

Creá el archivo `src/components/MovieForm.tsx`:

```tsx
// src/components/MovieForm.tsx
// Formulario para crear una película (modo editar se agrega en Bloque 2)

import { useState } from "react";
import type { MovieInput } from "../hooks/useMovie";

interface MovieFormProps {
  onGuardar: (datos: MovieInput) => void;
  onCancelar: () => void;
}

const FORMULARIO_VACIO: MovieInput = {
  titulo: "",
  anio: new Date().getFullYear(),
  genero: "",
  director: "",
  sinopsis: "",
  calificacion: 0,
};

const MovieForm = ({ onGuardar, onCancelar }: MovieFormProps) => {
  const [form, setForm] = useState<MovieInput>(FORMULARIO_VACIO);

  // Un solo handler para todos los campos — computed property name
  const handleCampo = (campo: keyof MovieInput, valor: string | number) => {
    setForm((prev) => ({ ...prev, [campo]: valor }));
  };

  const handleEnviar = () => {
    if (!form.titulo.trim() || !form.genero.trim()) {
      alert("Título y género son obligatorios.");
      return;
    }
    onGuardar(form);
  };

  return (
    <div className="bg-gray-800 rounded-xl p-6 w-full max-w-lg mx-auto">
      <h2 className="text-white text-xl font-bold mb-5">Nueva película</h2>

      {/* Título */}
      <div className="mb-4">
        <label className="block text-gray-300 text-sm mb-1">
          Título <span className="text-red-400">*</span>
        </label>
        <input
          type="text"
          value={form.titulo}
          onChange={(e) => handleCampo("titulo", e.target.value)}
          className="w-full bg-gray-700 text-white rounded-lg px-3 py-2 text-sm
                     border border-gray-600 focus:outline-none focus:border-blue-400"
          placeholder="Ej: Interstellar"
        />
      </div>

      {/* Año y Género */}
      <div className="flex gap-3 mb-4">
        <div className="flex-1">
          <label className="block text-gray-300 text-sm mb-1">Año</label>
          <input
            type="number"
            value={form.anio}
            onChange={(e) => handleCampo("anio", parseInt(e.target.value))}
            className="w-full bg-gray-700 text-white rounded-lg px-3 py-2 text-sm
                       border border-gray-600 focus:outline-none focus:border-blue-400"
            min={1888}
            max={new Date().getFullYear() + 2}
          />
        </div>
        <div className="flex-1">
          <label className="block text-gray-300 text-sm mb-1">
            Género <span className="text-red-400">*</span>
          </label>
          <input
            type="text"
            value={form.genero}
            onChange={(e) => handleCampo("genero", e.target.value)}
            className="w-full bg-gray-700 text-white rounded-lg px-3 py-2 text-sm
                       border border-gray-600 focus:outline-none focus:border-blue-400"
            placeholder="Ej: Drama"
          />
        </div>
      </div>

      {/* Director */}
      <div className="mb-4">
        <label className="block text-gray-300 text-sm mb-1">Director</label>
        <input
          type="text"
          value={form.director ?? ""}
          onChange={(e) => handleCampo("director", e.target.value)}
          className="w-full bg-gray-700 text-white rounded-lg px-3 py-2 text-sm
                     border border-gray-600 focus:outline-none focus:border-blue-400"
          placeholder="Ej: Christopher Nolan"
        />
      </div>

      {/* Calificación */}
      <div className="mb-4">
        <label className="block text-gray-300 text-sm mb-1">Calificación (0–10)</label>
        <input
          type="number"
          value={form.calificacion ?? 0}
          onChange={(e) => handleCampo("calificacion", parseFloat(e.target.value))}
          className="w-full bg-gray-700 text-white rounded-lg px-3 py-2 text-sm
                     border border-gray-600 focus:outline-none focus:border-blue-400"
          min={0} max={10} step={0.1}
        />
      </div>

      {/* Sinopsis */}
      <div className="mb-6">
        <label className="block text-gray-300 text-sm mb-1">Sinopsis</label>
        <textarea
          value={form.sinopsis ?? ""}
          onChange={(e) => handleCampo("sinopsis", e.target.value)}
          rows={3}
          className="w-full bg-gray-700 text-white rounded-lg px-3 py-2 text-sm
                     border border-gray-600 focus:outline-none focus:border-blue-400
                     resize-none"
          placeholder="Breve descripción de la película..."
        />
      </div>

      {/* Botones */}
      <div className="flex gap-3 justify-end">
        <button
          onClick={onCancelar}
          className="px-4 py-2 rounded-lg text-sm text-gray-300 bg-gray-700
                     hover:bg-gray-600 transition-colors"
        >
          Cancelar
        </button>
        <button
          onClick={handleEnviar}
          className="px-4 py-2 rounded-lg text-sm font-semibold text-white
                     bg-blue-600 hover:bg-blue-500 transition-colors"
        >
          Agregar
        </button>
      </div>
    </div>
  );
};

export default MovieForm;
```

---

## Tarea 1.3 — Conectar CREATE en `Catalog.tsx`

### Qué haces

Agregar el botón "+ Nueva película", el panel lateral y el handler `handleGuardar` que llama a `crearPelicula`.

### Cambios en `Catalog.tsx`

```tsx
// src/pages/Catalog.tsx
// Agregar estos imports
import { useState } from "react";
import type { MovieInput } from "../hooks/useMovie";
import MovieForm from "../components/MovieForm";

// Agregar crearPelicula a la desestructuración del hook
const { peliculas, cargando, error, crearPelicula } = useMovie();

// Agregar estos estados de UI
const [panelAbierto, setPanelAbierto] = useState<boolean>(false);

// Agregar estos handlers
const handleNuevaPelicula = () => {
  setPanelAbierto(true);
};

const handleCancelar = () => {
  setPanelAbierto(false);
};

const handleGuardar = async (datos: MovieInput) => {
  const exito = await crearPelicula(datos);
  if (exito) setPanelAbierto(false);
};
```

### En el JSX — agregar el botón en el header

```tsx
<button
  onClick={handleNuevaPelicula}
  className="bg-blue-600 hover:bg-blue-500 text-white text-sm font-semibold
             px-4 py-2 rounded-lg transition-colors"
>
  + Nueva película
</button>
```

### En el JSX — agregar el panel lateral al final del layout

```tsx
{panelAbierto && (
  <div className="w-full max-w-sm border-l border-gray-700
                  bg-gray-900 overflow-y-auto p-6">
    <MovieForm
      onGuardar={handleGuardar}
      onCancelar={handleCancelar}
    />
  </div>
)}
```

### ✅ Verificar CREATE

- Clic en "+ Nueva película" → panel con formulario vacío
- Completar campos y guardar → película aparece al tope sin recargar
- En Supabase → Table Editor → fila nueva con `id` asignado

---

# BLOQUE 2 — UPDATE (Actualizar)

Ahora que crear funciona, agregamos la edición. Los cambios son incrementales sobre lo que ya existe.

---

## Tarea 2.1 — Agregar `actualizarPelicula` a `useMovie.ts`

### Código — agregar después de `crearPelicula`

```ts
// src/hooks/useMovie.ts — agregar esta función
const actualizarPelicula = async (
  id: number,
  cambios: MovieInput
): Promise<boolean> => {
  setError(null);

  const { error: supabaseError } = await supabase
    .from("peliculas")
    .update(cambios)
    .eq("id", id);

  if (supabaseError) {
    setError("Error al actualizar: " + supabaseError.message);
    return false;
  }

  // actualizar solo esa película en el estado local
  setPeliculas((prev) =>
    prev.map((p) => (p.id === id ? { id, ...cambios } : p))
  );
  return true;
};
```

### Agregar `actualizarPelicula` al return del hook

```ts
return {
  peliculas,
  cargando,
  error,
  crearPelicula,
  actualizarPelicula,  // ← agregar
};
```

---

## Tarea 2.2 — Actualizar `MovieForm.tsx` para modo edición

### Qué cambia

`MovieForm` ahora recibe una prop opcional `pelicula?: Movie`. Si viene con datos, el formulario se pre-llena y el botón dice "Guardar cambios".

### Cambios en `MovieForm.tsx`

```tsx
// Agregar el import de Movie
import type { Movie } from "../types/Movie";

// Actualizar la interfaz de props
interface MovieFormProps {
  pelicula?: Movie;                        // ← agregar — undefined = crear
  onGuardar: (datos: MovieInput) => void;
  onCancelar: () => void;
}

// Actualizar la firma del componente
const MovieForm = ({ pelicula, onGuardar, onCancelar }: MovieFormProps) => {

  // Reemplazar el useState inicial
  const [form, setForm] = useState<MovieInput>(
    pelicula
      ? {
          titulo: pelicula.titulo,
          anio: pelicula.anio,
          genero: pelicula.genero,
          director: pelicula.director ?? "",
          sinopsis: pelicula.sinopsis ?? "",
          calificacion: pelicula.calificacion ?? 0,
        }
      : FORMULARIO_VACIO
  );

  const esModoEdicion = pelicula !== undefined;

  // Actualizar el título del formulario
  // <h2>  {esModoEdicion ? "Editar película" : "Nueva película"}  </h2>

  // Actualizar el botón Agregar
  // {esModoEdicion ? "Guardar cambios" : "Agregar"}
```

---

## Tarea 2.3 — Conectar UPDATE en `Catalog.tsx`

### Cambios en `Catalog.tsx`

```tsx
// Agregar Movie al import de types
import type { Movie } from "../types/Movie";

// Agregar actualizarPelicula a la desestructuración
const {
  peliculas,
  cargando,
  error,
  crearPelicula,
  actualizarPelicula,  // ← agregar
} = useMovie();

// Agregar estado para la película que se está editando
const [peliculaEditando, setPeliculaEditando] = useState<Movie | null>(null);

// Agregar handler de edición
const handleEditarPelicula = (pelicula: Movie) => {
  setPeliculaEditando(pelicula);
  setPanelAbierto(true);
};

// Actualizar handleCancelar para limpiar peliculaEditando
const handleCancelar = () => {
  setPanelAbierto(false);
  setPeliculaEditando(null);  // ← agregar
};

// Actualizar handleGuardar para distinguir crear vs editar
const handleGuardar = async (datos: MovieInput) => {
  let exito: boolean;

  if (peliculaEditando) {
    exito = await actualizarPelicula(peliculaEditando.id, datos);
  } else {
    exito = await crearPelicula(datos);
  }

  if (exito) {
    setPanelAbierto(false);
    setPeliculaEditando(null);
  }
};
```

### En el JSX — pasar `pelicula` al formulario

```tsx
<MovieForm
  pelicula={peliculaEditando ?? undefined}  // ← agregar
  onGuardar={handleGuardar}
  onCancelar={handleCancelar}
/>
```

### En el JSX — pasar `onEditar` a cada tarjeta

```tsx
<MovieCard
  key={pelicula.id}
  pelicula={pelicula}
  onEditar={handleEditarPelicula}  // ← agregar
  onEliminar={handleEliminar}      // (viene en Bloque 3)
/>
```

> `MovieCard` todavía no tiene el botón Editar — lo agregamos en el Bloque 3 junto con Eliminar para hacerlo de una sola vez.

### ✅ Verificar UPDATE

- Clic en "✏️ Editar" (lo agregamos en Bloque 3) → formulario pre-llenado
- Modificar la calificación y guardar → tarjeta se actualiza sin recargar
- En Supabase → Table Editor el cambio se refleja en la BD

---

# BLOQUE 3 — DELETE (Eliminar)

Último bloque. Agregamos eliminar y de paso actualizamos `MovieCard` con ambos botones de una sola vez.

---

## Tarea 3.1 — Agregar `eliminarPelicula` a `useMovie.ts`

### Código — agregar después de `actualizarPelicula`

```ts
// src/hooks/useMovie.ts — agregar esta función
const eliminarPelicula = async (id: number): Promise<boolean> => {
  setError(null);

  const { error: supabaseError } = await supabase
    .from("peliculas")
    .delete()
    .eq("id", id);

  if (supabaseError) {
    setError("Error al eliminar: " + supabaseError.message);
    return false;
  }

  // quitar del estado local
  setPeliculas((prev) => prev.filter((p) => p.id !== id));
  return true;
};
```

### Return final del hook — completo

```ts
return {
  peliculas,
  cargando,
  error,
  crearPelicula,
  actualizarPelicula,
  eliminarPelicula,  // ← agregar
};
```

---

## Tarea 3.2 — Actualizar `MovieCard.tsx` con ambos botones

### Código completo de `MovieCard.tsx`

```tsx
// src/components/MovieCard.tsx
// Tarjeta de película con botones Editar y Eliminar

import type { Movie } from "../types/Movie";

interface MovieCardProps {
  pelicula: Movie;
  onEditar: (pelicula: Movie) => void;
  onEliminar: (id: number) => void;
}

const MovieCard = ({ pelicula, onEditar, onEliminar }: MovieCardProps) => {
  const colorCalificacion =
    (pelicula.calificacion ?? 0) >= 8
      ? "text-green-400"
      : (pelicula.calificacion ?? 0) >= 6
      ? "text-yellow-400"
      : "text-red-400";

  return (
    <div className="bg-gray-800 rounded-xl p-4 flex flex-col gap-2
                    border border-gray-700 hover:border-blue-500 transition-colors">
      {/* Encabezado */}
      <div className="flex justify-between items-start gap-2">
        <h3 className="text-white font-semibold text-base leading-tight">
          {pelicula.titulo}
        </h3>
        <span className={`text-sm font-bold whitespace-nowrap ${colorCalificacion}`}>
          ⭐ {pelicula.calificacion?.toFixed(1) ?? "—"}
        </span>
      </div>

      <p className="text-gray-400 text-xs">
        {pelicula.anio} · {pelicula.genero}
      </p>

      {pelicula.director && (
        <p className="text-gray-500 text-xs">Dir. {pelicula.director}</p>
      )}

      {pelicula.sinopsis && (
        <p className="text-gray-400 text-xs line-clamp-2">{pelicula.sinopsis}</p>
      )}

      {/* Botones */}
      <div className="flex gap-2 mt-auto pt-2">
        <button
          onClick={() => onEditar(pelicula)}
          className="flex-1 py-1.5 rounded-lg text-xs font-medium
                     bg-gray-700 text-gray-200 hover:bg-blue-700 hover:text-white
                     transition-colors"
        >
          ✏️ Editar
        </button>
        <button
          onClick={() => onEliminar(pelicula.id)}
          className="flex-1 py-1.5 rounded-lg text-xs font-medium
                     bg-gray-700 text-gray-200 hover:bg-red-700 hover:text-white
                     transition-colors"
        >
          🗑️ Eliminar
        </button>
      </div>
    </div>
  );
};

export default MovieCard;
```

---

## Tarea 3.3 — Conectar DELETE en `Catalog.tsx`

### Cambios en `Catalog.tsx`

```tsx
// Agregar eliminarPelicula a la desestructuración
const {
  peliculas,
  cargando,
  error,
  crearPelicula,
  actualizarPelicula,
  eliminarPelicula,  // ← agregar
} = useMovie();

// Agregar el handler de eliminar
const handleEliminar = async (id: number) => {
  if (!confirm("¿Eliminar esta película? Esta acción no se puede deshacer.")) return;
  await eliminarPelicula(id);
};
```

### En el JSX — `onEliminar` ya está en `MovieCard` desde el Bloque 2

```tsx
<MovieCard
  key={pelicula.id}
  pelicula={pelicula}
  onEditar={handleEditarPelicula}
  onEliminar={handleEliminar}     // ← ahora sí está implementado
/>
```

### ✅ Verificar DELETE

- Clic en "🗑️ Eliminar" → `confirm()` pide confirmación
- Confirmar → película desaparece de la lista sin recargar
- En Supabase → Table Editor → la fila ya no existe

---

## Verificación final — Checklist CRUD completo

```bash
npm run dev
```

**READ**
- [ ] Las películas se muestran ordenadas por calificación descendente
- [ ] El buscador filtra por título y género en tiempo real

**CREATE**
- [ ] Clic en "+ Nueva película" → panel con formulario vacío
- [ ] Al agregar: película aparece al tope sin recargar la página
- [ ] En Supabase la fila nueva tiene `id` asignado

**UPDATE**
- [ ] Clic en "✏️ Editar" → formulario pre-llenado con datos actuales
- [ ] Al guardar: tarjeta se actualiza en la lista inmediatamente
- [ ] El cambio se refleja en Supabase

**DELETE**
- [ ] Clic en "🗑️ Eliminar" → `confirm()` pide confirmación
- [ ] Película desaparece de la lista sin recargar
- [ ] La fila ya no existe en Supabase

---

# PARTE 2 — Tarea autónoma

## Contexto

Ya tenés CRUD completo para `peliculas`. Ahora vas a repetir el mismo flujo para la tabla `directores` que creaste en D06, siguiendo el mismo orden: primero CREATE, después UPDATE, después DELETE.

---

## Tarea A — Agregar políticas RLS a directores

Agregá las políticas INSERT, UPDATE y DELETE siguiendo el mismo proceso de la Tarea 0.

---

## Tarea B — CREATE para directores

**B.1** Mové el tipo `Director` de `Directors.tsx` a `src/types/Director.ts`.

**B.2** Creá `src/hooks/useDirector.ts` con el READ y `crearDirector`.

**B.3** En `Directors.tsx` agregá el botón "+ Nuevo director" y el formulario inline con visibilidad condicional:

```tsx
{formularioAbierto && (
  <div className="...">
    {/* campos del formulario */}
  </div>
)}
```

**B.4** Verificá que un nuevo director aparece en Supabase.

---

## Tarea C — UPDATE para directores

**C.1** Agregá `actualizarDirector` a `useDirector.ts`.

**C.2** Actualizá el formulario inline para que funcione en modo edición.

**C.3** Verificá que el cambio se refleja en Supabase.

---

## Tarea D — DELETE para directores

**D.1** Agregá `eliminarDirector` a `useDirector.ts`.

**D.2** Agregá el botón Eliminar en cada tarjeta de director.

**D.3** Verificá que la fila desaparece de Supabase.

---

## ¿Qué aprendiste hoy?

| Concepto | Dónde lo aplicaste |
|----------|-------------------|
| `insert().select().single()` | `crearPelicula` — obtener la fila con su `id` |
| `update().eq("id", id)` | `actualizarPelicula` — modificar solo la fila correcta |
| `delete().eq("id", id)` | `eliminarPelicula` — eliminar solo la fila correcta |
| Solo READ en `useEffect` | CREATE/UPDATE/DELETE son funciones fuera del efecto |
| `setPeliculas((prev) => ...)` | Garantiza leer el estado más reciente |
| `Omit<Movie, "id">` | `MovieInput` — crear sin id (lo asigna la BD) |
| Panel lateral condicional | `panelAbierto && <MovieForm />` en Catalog |
| RLS con 4 políticas | SELECT + INSERT + UPDATE + DELETE sobre la tabla |
| Callbacks como props | `onEditar`, `onEliminar` — `MovieCard` no llama a Supabase |

---

*SIS-0300 · Universidad Privada Domingo Savio · Tarija, Bolivia*
