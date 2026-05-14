# Laboratorio — useState y Búsqueda en Tiempo Real

**Asignatura:** Programación Web I — SIS-0300  
**Carrera:** Ingeniería de Sistemas · UPDS Tarija  
**Proyecto:** CineApp · Fase 1 · Día 2

---

## Prerequisito

Tener el proyecto `cine-app` del Día 1 funcionando con:

- `src/components/MovieCard.tsx` con props tipadas
- `src/App.tsx` renderizando las 6 películas

Ejecutar el proyecto antes de empezar:

```bash
npm run dev
```

---

## Lo que vas a construir hoy

Al final del laboratorio, CineApp va a tener una barra de búsqueda que filtra las películas en tiempo real mientras el usuario escribe.

Para llegar ahí, primero hay que entender **qué es el estado** y cómo funciona `useState`.

---

# Parte 1 — Entender useState (ejercicio guiado)

---

## Ejercicio 1.1 — Tu primer estado

**Consigna:** Modificar `App.tsx` para agregar un contador que sume 1 cada vez que se hace clic en un botón. El número debe mostrarse en pantalla y actualizarse sin recargar la página.

> Antes de escribir el código, responder: ¿podrías hacer esto con una variable normal (`let contador = 0`)? ¿Por qué sí o por qué no?

**Pistas:**

```tsx
// 1. Importar useState al inicio del archivo
import { useState } from "react";

// 2. Declarar el estado dentro del componente, antes del return
//    El tipo va entre < >, el valor inicial entre ( )
const [contador, setContador] = useState<number>(0);

// 3. Mostrar el valor en el JSX
<h2>Contador: {contador}</h2>

// 4. Cambiar el valor con un botón
<button onClick={() => setContador(contador + 1)}>
  Sumar 1
</button>
```

**Verificar:** al hacer clic, el número cambia en pantalla sin recargar.

---

## Ejercicio 1.2 — Botón de reinicio

**Consigna:** Agregar un segundo botón "Reiniciar" que devuelva el contador a 0.

**Pista:**

```tsx
// setContador acepta cualquier número, no solo contador + 1
```

---

## Ejercicio 1.3 — Romper el estado a propósito

**Consigna:** Reemplazar `setContador(contador + 1)` por una asignación directa:

```tsx
// ⚠️ Esto está mal — hacerlo solo para ver qué pasa
onClick={() => { contador = contador + 1 }}
```

Leer el error de TypeScript. Luego deshacer el cambio.

> Responder: ¿qué dice el error? ¿Por qué TypeScript no permite modificar la variable directamente?

---

## Ejercicio 1.4 — Estado de texto

**Consigna:** Reemplazar el contador por un input de texto. Mientras el usuario escribe su nombre, mostrar debajo: `Hola, [nombre]`. Si el input está vacío, no mostrar nada.

**Pistas:**

```tsx
// 1. El estado ahora es string
const [nombre, setNombre] = useState<string>("");

// 2. El input debe ser "controlado": su value viene del estado
<input
  value={nombre}
  onChange={(e) => setNombre(e.target.value)}
/>

// 3. Renderizado condicional: mostrar algo solo si se cumple una condición
{nombre !== "" && (
  <p>Hola, <strong>{nombre}</strong></p>
)}
```

**Verificar:** al escribir, el saludo aparece en tiempo real. Al borrar todo, desaparece.

---

# Parte 2 — Aplicar en CineApp (ejercicio guiado)

---

## Ejercicio 2.1 — Convertir las películas a un array

**Consigna:** En el Día 1 las películas eran 6 bloques JSX repetidos. Ahora hay que definirlas como un array de objetos tipados con una `interface`, y usar `.map()` para renderizarlas.

El resultado visual debe ser **exactamente igual** al del Día 1.

**Pistas:**

```tsx
// 1. Definir la interface ANTES del componente
interface Pelicula {
  id: number;
  titulo: string;
  anio: number;
  genero: string;
  calificacion: number;
}

// 2. Definir el array de datos ANTES del componente
//    (fuera del return, fuera del componente)
const PELICULAS: Pelicula[] = [
  { id: 1, titulo: "Interstellar", anio: 2014, genero: "Ciencia ficción", calificacion: 8.7 },
  // ... completar con las 6 películas
];

// 3. Dentro del return, usar .map() para renderizar
{PELICULAS.map((pelicula) => (
  <MovieCard
    key={pelicula.id}
    titulo={pelicula.titulo}
    // ... completar el resto de props
  />
))}
```

**Verificar:** las 6 tarjetas siguen apareciendo igual que antes.

> Responder: ¿qué es `key` y por qué React lo necesita en los `.map()`?

---

## Ejercicio 2.2 — Agregar la barra de búsqueda

**Consigna:** Agregar un estado `busqueda` de tipo `string`. Mostrar un `<input>` arriba del grid que controle ese estado. Por ahora no tiene que filtrar nada — solo que al escribir, el estado se actualice.

**Pistas:**

```tsx
// Estado para el texto de búsqueda
const [busqueda, setBusqueda] = useState<string>("");

// Input controlado — completar el onChange
<input
  type="text"
  placeholder="Buscar película..."
  value={busqueda}
  onChange={/* completar */}
/>
```

**Verificar:** agregar temporalmente `<p>{busqueda}</p>` para confirmar que el estado se actualiza al escribir. Quitarlo después.

---

## Ejercicio 2.3 — Conectar la búsqueda con el grid

**Consigna:** Crear una variable `peliculasFiltradas` que contenga solo las películas cuyo título incluya el texto de `busqueda`. El grid debe renderizar `peliculasFiltradas` en lugar de `PELICULAS`.

**Pistas:**

```tsx
// Esta variable NO es estado — se calcula en cada render
// Va entre los useState y el return
const peliculasFiltradas = PELICULAS.filter((p) =>
  p.titulo.toLowerCase().includes(/* completar */)
);

// En el .map(), reemplazar PELICULAS por peliculasFiltradas
```

**Verificar:**
- Escribir `"inter"` → solo aparece Interstellar
- Escribir `"a"` → aparecen varias
- Borrar todo → vuelven las 6

> Responder: ¿`peliculasFiltradas` necesita `useState`? ¿Por qué no?

---

## Ejercicio 2.4 — Mensaje cuando no hay resultados

**Consigna:** Si `peliculasFiltradas` está vacío, mostrar el mensaje:  
`No se encontraron películas para "[texto buscado]".`  
Si hay resultados, mostrar el grid normalmente.

**Pista:**

```tsx
// Operador ternario: condición ? si_verdadero : si_falso
{peliculasFiltradas.length === 0 ? (
  <p>No se encontraron películas para "{/* completar */}".</p>
) : (
  /* el grid va acá */
)}
```

---

## Ejercicio 2.5 — Contador de resultados

**Consigna:** Agregar debajo del input, antes del grid, una línea que muestre cuántas películas se encontraron. Debe actualizarse mientras el usuario escribe.

Ejemplo: `3 película(s) encontrada(s)`

**Pista:**

```tsx
// peliculasFiltradas ya existe — solo necesitás su .length
```

---

# Parte 3 — Ejercicio autónomo

> A partir de acá trabajás solo. No hay pistas.

---

## Ejercicio 3.1

Agregar al array `PELICULAS` tres películas nuevas a elección, con distintas calificaciones y géneros.

Verificar que aparecen en el grid y que la búsqueda las incluye.

---

## Ejercicio 3.2

Agregar un segundo filtro: un `<select>` que permita filtrar por género.

Requisitos:
- Nuevo estado `generoSeleccionado` de tipo `string`, valor inicial `"Todos"`
- El select debe tener las opciones: `Todos`, `Acción`, `Drama`, `Thriller`, `Ciencia ficción`
- Al seleccionar un género, mostrar solo las películas de ese género
- Al seleccionar "Todos", mostrar todas
- El filtro de género y el de búsqueda deben funcionar **al mismo tiempo**

---

## Ejercicio 3.3

Agregar un botón "Limpiar filtros" que:
- Solo aparezca cuando hay texto escrito en el input **o** hay un género seleccionado distinto de `"Todos"`
- Al hacer clic, resetee ambos estados a sus valores iniciales

---

## Ejercicio 3.4

Agregar el campo `esFavorita: boolean` al tipo `Pelicula` y al array de datos.

Actualizar `MovieCard` para mostrar ❤️ junto al título si la película es favorita.

---

# Avance autónomo

Investigar para la próxima clase:

- ¿Qué es `useEffect`?
- ¿Para qué sirve el array de dependencias `[]`?
- ¿Cuándo se ejecuta si el array está vacío?
