# Laboratorio D08 — Supabase Auth: Registro, Login y Rutas Reales

**SIS-0300 Programación Web I · Fase 4 · Día 8**
Universidad Privada Domingo Savio · Facultad de Ingeniería

---

## Dónde estamos

| Día | Tema | Estado |
|-----|------|--------|
| D06 | CRUD completo | ✅ Completado |
| D07 | React Router v7 — navegación y rutas protegidas | ✅ Completado |
| **D08** | **Supabase Auth — registro, login y rutas reales** | ← Hoy |

Al terminar D07 tenemos navegación entre páginas con React Router, rutas públicas `/`, `/pelicula/:id`, `/login` y una ruta protegida `/admin` que usa `SESION_SIMULADA = false` como guardia temporal.

**El problema de hoy:** la sesión es falsa. Cualquiera puede cambiar `SESION_SIMULADA = true` en el código y entrar a `/admin` sin ninguna credencial. Hoy reemplazamos esa constante con autenticación real usando Supabase Auth: usuarios almacenados en la base de datos, contraseñas encriptadas y tokens de sesión gestionados automáticamente.

---

## Qué construimos hoy

```
Antes (D07)                     Después (D08)
──────────────────────────────────────────────────────
Login simulado con alert()  →   login real con Supabase
SESION_SIMULADA = false     →   supabase.auth.getSession()
Sin registro de usuarios    →   /register con email+password
Navbar sin estado de sesión →   Navbar muestra usuario + Cerrar sesión
```

Flujo completo al finalizar:

```
/register  →  formulario de registro (nueva cuenta)
/login     →  formulario de login (sesión existente)
/          →  catálogo público
/admin     →  protegido con sesión REAL de Supabase
```

---

## Estructura de archivos al finalizar

```
src/
├── lib/
│   └── supabaseClient.ts          ← sin cambios
├── types/
│   └── Movie.ts                   ← sin cambios
├── hooks/
│   ├── useMovie.ts                ← sin cambios
│   └── useAuth.ts                 ← NUEVO
├── components/
│   ├── MovieCard.tsx              ← sin cambios
│   ├── MovieForm.tsx              ← sin cambios
│   ├── MovieList.tsx              ← sin cambios
│   ├── Navbar.tsx                 ← actualizado (estado de sesión real)
│   └── ProtectedRoute.tsx         ← actualizado (sesión real de Supabase)
├── pages/
│   ├── Catalog.tsx                ← sin cambios
│   ├── MovieDetail.tsx            ← sin cambios
│   ├── Login.tsx                  ← actualizado (auth real)
│   ├── Register.tsx               ← NUEVO
│   └── Admin.tsx                  ← sin cambios
└── main.tsx                       ← actualizado (ruta /register)
```

---

## Estándar de código del día

| Qué | Convención |
|-----|-----------|
| Componentes | `const NombreComponente = (props: Props) => { return (...) }` |
| Custom hooks | `const useAuth = () => { ... }` en `src/hooks/useAuth.ts` |
| Operaciones async | siempre con try/catch o destructuring `{ data, error }` |
| Manejo de sesión | nunca en componentes directamente — siempre a través de `useAuth` |

---

## Configuración en Supabase

Antes de escribir una sola línea de código hay que revisar tres ajustes en el dashboard. Si saltás este paso, el registro y el login van a fallar aunque el código esté perfecto.

### 1 — Habilitar el proveedor Email

Ir a **Authentication → Providers → Email** y verificar que el toggle esté en ON. Este proveedor es el que permite crear usuarios con email y contraseña. Por defecto ya viene activado en proyectos nuevos, pero conviene confirmarlo antes de empezar.

### 2 — Desactivar la confirmación de email

En la misma pantalla, buscar la opción **"Confirm email"** y desactivarla.

Cuando esta opción está activada, Supabase envía un correo de confirmación al usuario al registrarse y no le permite iniciar sesión hasta que haga clic en el enlace de ese correo. En producción eso es correcto porque verifica que el email existe. Pero en el laboratorio vamos a crear cuentas de prueba con emails inventados, así que el correo nunca llegaría y el login nunca funcionaría.

```
Dashboard → Authentication → Providers → Email
  ✅ Enable Email provider    (debe estar ON)
  ☐  Confirm email            (debe estar OFF para el laboratorio)
```

> En un proyecto real esta opción debería estar activada siempre. La desactivamos únicamente para simplificar las pruebas en clase.

### 3 — Verificar la URL de redirección

Ir a **Authentication → URL Configuration** y verificar que `http://localhost:5173` aparezca en la lista de **Redirect URLs**. Si no está, agregarlo manualmente. Supabase usa esta lista para saber a qué URLs puede redirigir al usuario después de confirmar el email o restablecer contraseña. Si la URL de tu app local no está en la lista, algunos flujos van a fallar con el error `"Redirect URL not allowed"`.

```
Dashboard → Authentication → URL Configuration → Redirect URLs
  Agregar:  http://localhost:5173
```

---

## Concepto previo — Cómo funciona Supabase Auth

Supabase Auth es un servicio de autenticación completo incluido en cada proyecto de Supabase. Maneja el registro, el login, los tokens de sesión y el cierre de sesión. El cliente JavaScript guarda automáticamente la sesión activa en `localStorage` del navegador — no tenés que hacerlo manualmente ni preocuparte por los tokens.

Cada usuario que se registra queda almacenado en una tabla especial llamada `auth.users`, separada de las tablas que vos creás. No podés ni debés manipular esa tabla directamente — Auth la gestiona por vos.

Los métodos que usás hoy:

```ts
// Registrar un usuario nuevo — crea la cuenta en auth.users
const { data, error } = await supabase.auth.signUp({ email, password });

// Iniciar sesión con usuario existente — devuelve un token de sesión
const { data, error } = await supabase.auth.signInWithPassword({ email, password });

// Cerrar sesión — invalida el token y limpia localStorage
const { error } = await supabase.auth.signOut();

// Obtener la sesión activa — lee el token guardado en localStorage
// Si no hay sesión activa devuelve session = null
const { data: { session } } = await supabase.auth.getSession();

// Suscribirse a cambios de sesión en tiempo real
// event puede ser: "SIGNED_IN", "SIGNED_OUT", "TOKEN_REFRESHED", "USER_UPDATED"
supabase.auth.onAuthStateChange((event, session) => {
  // session es null cuando no hay usuario autenticado
});
```

### `getSession` vs `onAuthStateChange`

`getSession` es una lectura puntual: consultás la sesión una vez y obtenés el resultado. Es útil para saber si hay sesión al cargar la app. Su límite es que no detecta cambios futuros — si el token expira o el usuario cierra sesión desde otra pestaña, `getSession` no se entera porque ya terminó su trabajo.

`onAuthStateChange` es una suscripción activa: escucha todos los cambios de estado mientras la app está montada. Cada vez que el usuario inicia sesión, cierra sesión o su token se renueva, el callback se ejecuta con el nuevo estado.

Por eso en `useAuth` usamos los dos juntos: `getSession` para la lectura inicial al montar, y `onAuthStateChange` para mantenerse sincronizado en tiempo real después.

---

# BLOQUE 1 — Hook de autenticación

El principio de este bloque es que **ningún componente debe hablar directamente con Supabase Auth**. Todo pasa por un custom hook llamado `useAuth`. Esto tiene dos ventajas concretas: si mañana cambiás de proveedor de autenticación, solo tocás `useAuth.ts` y los componentes no cambian; y toda la lógica de sesión vive en un solo lugar, fácil de leer y depurar.

---

## Tarea 1.1 — Crear `useAuth.ts`

El hook maneja dos estados internos: `usuario` y `cargando`.

**`usuario`** guarda el objeto `User` de Supabase cuando hay sesión activa, o `null` cuando no la hay. El tipo `User` viene de `@supabase/supabase-js` e incluye campos como `id`, `email` y `created_at`.

**`cargando`** arranca en `true` porque al montar el hook todavía no sabemos si hay una sesión guardada en localStorage. Pasa a `false` únicamente cuando `getSession` termina de leer. Este estado es clave para `ProtectedRoute` — sin él se produciría un "flash de redirección" que se explica en la Tarea 3.1.

Dentro del `useEffect` hay tres pasos: primero se llama a `getSession` para leer la sesión guardada (ocurre una sola vez al montar); segundo se registra `onAuthStateChange` para escuchar cambios futuros; tercero la función de limpieza llama a `unsubscribe` cuando el componente se desmonta, lo que evita una fuga de memoria porque el listener dejaría de existir en el DOM pero seguiría activo.

Las funciones `login` y `logout` usan `throw new Error` en lugar de retornar el error. Los hooks no tienen JSX para mostrar mensajes — lanzan el error para que el componente que los llama lo capture con `try/catch` y lo muestre en su propio estado.

Crear `src/hooks/useAuth.ts`:

```ts
import { useEffect, useState } from "react";
import type { User } from "@supabase/supabase-js";
import { supabase } from "../lib/supabaseClient";

const useAuth = () => {
  const [usuario, setUsuario] = useState<User | null>(null);
  const [cargando, setCargando] = useState<boolean>(true);

  useEffect(() => {
    const cargarSesion = async () => {
      const {
        data: { session },
      } = await supabase.auth.getSession();
      setUsuario(session?.user ?? null);
      setCargando(false);
    };

    cargarSesion();

    const { data: suscripcion } = supabase.auth.onAuthStateChange(
      (_evento, session) => {
        setUsuario(session?.user ?? null);
      }
    );

    return () => {
      suscripcion.subscription.unsubscribe();
    };
  }, []);

  const login = async (email: string, password: string) => {
    const { error } = await supabase.auth.signInWithPassword({
      email,
      password,
    });
    if (error) throw new Error(error.message);
  };

  const logout = async () => {
    const { error } = await supabase.auth.signOut();
    if (error) throw new Error(error.message);
  };

  return { usuario, cargando, login, logout };
};

export default useAuth;
```

---

# BLOQUE 2 — Actualizar Login y crear Register

Con `useAuth` listo, conectamos las páginas de autenticación. `Login.tsx` ya existe de D07 pero usaba un `alert` simulado — lo reemplazamos con auth real. `Register.tsx` es nuevo.

---

## Tarea 2.1 — Actualizar `Login.tsx`

El componente necesita tres cosas nuevas respecto a D07:

**Llamar a `login()` del hook.** Solo se destruye `login` de `useAuth`, no `usuario` — en esta página no necesitamos saber quién está logueado, solo ejecutar el inicio de sesión.

**Manejar el error.** `login()` lanza un error si las credenciales son incorrectas. Lo capturamos con `try/catch` y lo guardamos en el estado `error` para mostrarlo en pantalla. TypeScript exige verificar `err instanceof Error` antes de acceder a `.message` porque el tipo de `err` en un `catch` es `unknown`.

**Deshabilitar el botón mientras espera.** El estado `cargando` bloquea el botón durante la petición a Supabase. Sin esto el usuario podría hacer clic varias veces y enviar peticiones duplicadas. El bloque `finally` vuelve a habilitarlo siempre, haya error o no.

Reemplazá todo el contenido de `src/pages/Login.tsx`:

```tsx
import { useState } from "react";
import { useNavigate, Link } from "react-router";
import useAuth from "../hooks/useAuth";
import Navbar from "../components/Navbar";

const Login = () => {
  const navigate = useNavigate();
  const { login } = useAuth();

  const [email, setEmail] = useState<string>("");
  const [password, setPassword] = useState<string>("");
  const [error, setError] = useState<string | null>(null);
  const [cargando, setCargando] = useState<boolean>(false);

  const handleSubmit = async () => {
    if (!email.trim() || !password.trim()) {
      setError("Completá ambos campos.");
      return;
    }

    setCargando(true);
    setError(null);

    try {
      await login(email, password);
      navigate("/admin");
    } catch (err: unknown) {
      const mensaje =
        err instanceof Error ? err.message : "Error al iniciar sesión.";
      setError(mensaje);
    } finally {
      setCargando(false);
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

          {error && (
            <div className="bg-red-900 text-red-200 rounded-lg px-3 py-2 text-sm mb-4">
              {error}
            </div>
          )}

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
            <label className="block text-gray-300 text-sm mb-1">
              Contraseña
            </label>
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
            disabled={cargando}
            className="w-full bg-blue-600 hover:bg-blue-500 disabled:opacity-50
                       text-white font-semibold py-2 rounded-lg text-sm transition-colors"
          >
            {cargando ? "Ingresando..." : "Ingresar"}
          </button>

          <p className="text-gray-400 text-xs text-center mt-4">
            ¿No tenés cuenta?{" "}
            <Link to="/register" className="text-blue-400 hover:text-blue-300">
              Registrate
            </Link>
          </p>
        </div>
      </div>
    </div>
  );
};

export default Login;
```

---

## Tarea 2.2 — Crear `Register.tsx`

Este componente no usa `useAuth` — llama directamente a `supabase.auth.signUp`. La razón es que `useAuth` está diseñado para gestionar la sesión activa y escuchar sus cambios en tiempo real, algo que necesitás constantemente en Navbar, ProtectedRoute y Login. El registro es una operación de un solo uso: el usuario completa el formulario, se crea la cuenta y se termina. No hay estado de sesión que escuchar después, así que usar el hook completo sería innecesario.

Las validaciones se hacen en el cliente antes de llamar a Supabase para no gastar peticiones en errores detectables localmente. La validación de que las dos contraseñas coincidan no existe en Supabase — es responsabilidad del frontend implementarla.

El estado `exito` controla qué JSX renderizar: cuando el registro termina bien pasa a `true` y React muestra la pantalla de confirmación en lugar del formulario, sin necesidad de navegar a otra ruta.

Crear `src/pages/Register.tsx`:

```tsx
import { useState } from "react";
import { useNavigate, Link } from "react-router";
import { supabase } from "../lib/supabaseClient";
import Navbar from "../components/Navbar";

const Register = () => {
  const navigate = useNavigate();

  const [email, setEmail] = useState<string>("");
  const [password, setPassword] = useState<string>("");
  const [confirmPassword, setConfirmPassword] = useState<string>("");
  const [error, setError] = useState<string | null>(null);
  const [cargando, setCargando] = useState<boolean>(false);
  const [exito, setExito] = useState<boolean>(false);

  const handleRegistrar = async () => {
    setError(null);

    if (!email.trim() || !password.trim()) {
      setError("Completá todos los campos.");
      return;
    }
    if (password !== confirmPassword) {
      setError("Las contraseñas no coinciden.");
      return;
    }
    if (password.length < 6) {
      setError("La contraseña debe tener al menos 6 caracteres.");
      return;
    }

    setCargando(true);

    const { error: supabaseError } = await supabase.auth.signUp({
      email,
      password,
    });

    setCargando(false);

    if (supabaseError) {
      setError(supabaseError.message);
      return;
    }

    setExito(true);
  };

  if (exito) {
    return (
      <div className="min-h-screen bg-gray-950">
        <Navbar />
        <div className="flex items-center justify-center mt-20 px-4">
          <div className="bg-gray-800 rounded-xl p-8 w-full max-w-sm text-center">
            <div className="text-4xl mb-4">✅</div>
            <h2 className="text-white text-xl font-bold mb-2">
              ¡Cuenta creada!
            </h2>
            <p className="text-gray-400 text-sm mb-6">
              Tu cuenta fue creada correctamente. Ya podés iniciar sesión.
            </p>
            <button
              onClick={() => navigate("/login")}
              className="bg-blue-600 hover:bg-blue-500 text-white font-semibold
                         px-6 py-2 rounded-lg text-sm transition-colors"
            >
              Ir al login
            </button>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-950">
      <Navbar />
      <div className="flex items-center justify-center mt-20 px-4">
        <div className="bg-gray-800 rounded-xl p-8 w-full max-w-sm">
          <h2 className="text-white text-2xl font-bold mb-6 text-center">
            Crear cuenta
          </h2>

          {error && (
            <div className="bg-red-900 text-red-200 rounded-lg px-3 py-2 text-sm mb-4">
              {error}
            </div>
          )}

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

          <div className="mb-4">
            <label className="block text-gray-300 text-sm mb-1">
              Contraseña
            </label>
            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              className="w-full bg-gray-700 text-white rounded-lg px-3 py-2 text-sm
                         border border-gray-600 focus:outline-none focus:border-blue-400"
              placeholder="Mínimo 6 caracteres"
            />
          </div>

          <div className="mb-6">
            <label className="block text-gray-300 text-sm mb-1">
              Confirmar contraseña
            </label>
            <input
              type="password"
              value={confirmPassword}
              onChange={(e) => setConfirmPassword(e.target.value)}
              className="w-full bg-gray-700 text-white rounded-lg px-3 py-2 text-sm
                         border border-gray-600 focus:outline-none focus:border-blue-400"
              placeholder="••••••••"
            />
          </div>

          <button
            onClick={handleRegistrar}
            disabled={cargando}
            className="w-full bg-green-600 hover:bg-green-500 disabled:opacity-50
                       text-white font-semibold py-2 rounded-lg text-sm transition-colors"
          >
            {cargando ? "Registrando..." : "Crear cuenta"}
          </button>

          <p className="text-gray-400 text-xs text-center mt-4">
            ¿Ya tenés cuenta?{" "}
            <Link to="/login" className="text-blue-400 hover:text-blue-300">
              Iniciá sesión
            </Link>
          </p>
        </div>
      </div>
    </div>
  );
};

export default Register;
```

---

# BLOQUE 3 — Rutas protegidas reales

Con `useAuth` funcionando, actualizamos los dos archivos que dependían de `SESION_SIMULADA`: `ProtectedRoute.tsx` y `Navbar.tsx`. También agregamos la nueva ruta en `main.tsx`.

---

## Tarea 3.1 — Actualizar `ProtectedRoute.tsx`

La lógica del componente es la misma de D07: si hay usuario muestra el contenido, si no hay usuario redirige a `/login`. El cambio es que ahora la sesión viene de `useAuth` en lugar de una constante.

El detalle nuevo es el estado `cargando`. Cuando el usuario recarga la página estando en `/admin`, React monta el componente con `usuario = null` porque `getSession` todavía no terminó de leer localStorage. Sin el control de `cargando`, `ProtectedRoute` vería `null` y redirigiría a `/login` inmediatamente — aunque el usuario sí tiene sesión válida. Medio segundo después `getSession` termina, detecta la sesión, pero el usuario ya fue expulsado. Eso se llama **flash de redirección**.

Con `cargando = true` el componente muestra una pantalla de espera en lugar de redirigir. Cuando `getSession` termina y `cargando` pasa a `false`, React vuelve a evaluar: si hay `usuario` muestra el contenido, si no hay redirige. Así la redirección solo ocurre cuando realmente no hay sesión.

El `replace` en `<Navigate>` evita que `/admin` quede en el historial del navegador. Sin `replace`, al presionar "atrás" desde `/login` el usuario volvería a `/admin`, que lo redirigiría de nuevo a `/login` — un loop sin salida.

Reemplazá todo el contenido de `src/components/ProtectedRoute.tsx`:

```tsx
import { Navigate } from "react-router";
import useAuth from "../hooks/useAuth";

interface ProtectedRouteProps {
  children: React.ReactNode;
}

const ProtectedRoute = ({ children }: ProtectedRouteProps) => {
  const { usuario, cargando } = useAuth();

  if (cargando) {
    return (
      <div className="min-h-screen bg-gray-950 flex items-center justify-center">
        <p className="text-gray-400 text-sm">Verificando sesión...</p>
      </div>
    );
  }

  if (!usuario) {
    return <Navigate to="/login" replace />;
  }

  return <>{children}</>;
};

export default ProtectedRoute;
```

---

## Tarea 3.2 — Actualizar `Navbar.tsx`

La barra de navegación necesita mostrar contenido diferente según el estado de sesión: con sesión muestra el email, el enlace a Administración y el botón de cerrar sesión; sin sesión muestra solo el botón de iniciar sesión. Esto se resuelve con un renderizado condicional `usuario ? ... : ...`.

Cuando el usuario hace clic en "Cerrar sesión", `logout()` llama a `supabase.auth.signOut`. Supabase invalida el token y dispara `onAuthStateChange` con el evento `SIGNED_OUT`. El hook `useAuth` actualiza `usuario` a `null` y todos los componentes que lo usan — incluida la Navbar — se re-renderizan automáticamente sin necesidad de código extra. Después del logout se navega a `/login` para que el usuario no quede en una página protegida con sesión cerrada.

Reemplazá todo el contenido de `src/components/Navbar.tsx`:

```tsx
import { Link, useNavigate } from "react-router";
import useAuth from "../hooks/useAuth";

const Navbar = () => {
  const { usuario, logout } = useAuth();
  const navigate = useNavigate();

  const handleLogout = async () => {
    await logout();
    navigate("/login");
  };

  return (
    <nav
      className="bg-gray-900 border-b border-gray-700 px-6 py-3
                    flex items-center justify-between"
    >
      <Link to="/" className="text-white text-xl font-bold tracking-tight">
        🎬 CineApp
      </Link>

      <div className="flex items-center gap-6">
        <Link
          to="/"
          className="text-gray-300 hover:text-white text-sm transition-colors"
        >
          Catálogo
        </Link>

        {usuario ? (
          <>
            <Link
              to="/admin"
              className="text-gray-300 hover:text-white text-sm transition-colors"
            >
              Administración
            </Link>
            <span className="text-gray-500 text-xs hidden sm:block">
              {usuario.email}
            </span>
            <button
              onClick={handleLogout}
              className="bg-gray-700 hover:bg-gray-600 text-white text-sm
                         font-medium px-4 py-1.5 rounded-lg transition-colors"
            >
              Cerrar sesión
            </button>
          </>
        ) : (
          <Link
            to="/login"
            className="bg-blue-600 hover:bg-blue-500 text-white text-sm
                       font-semibold px-4 py-1.5 rounded-lg transition-colors"
          >
            Iniciar sesión
          </Link>
        )}
      </div>
    </nav>
  );
};

export default Navbar;
```

---

## Tarea 3.3 — Actualizar `main.tsx`

Solo hay que agregar la importación de `Register` y la ruta `/register`. Todo lo demás queda idéntico a D07.

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter, Routes, Route } from "react-router";
import Catalog from "./pages/Catalog";
import MovieDetail from "./pages/MovieDetail";
import Login from "./pages/Login";
import Register from "./pages/Register";
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
        <Route path="/register" element={<Register />} />

        {/* Rutas protegidas — redirigen a /login si no hay sesión */}
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

---

## Verificación final — Checklist completo

```bash
npm run dev
```

**Registro**
- [ ] `/register` muestra el formulario
- [ ] Campos vacíos → "Completá todos los campos."
- [ ] Contraseñas distintas → "Las contraseñas no coinciden."
- [ ] Contraseña menor a 6 caracteres → mensaje de error
- [ ] Registro exitoso → pantalla de confirmación con ✅
- [ ] "Ir al login" lleva a `/login`

**Login**
- [ ] Credenciales incorrectas → muestra el mensaje de error de Supabase
- [ ] Campos vacíos → "Completá ambos campos."
- [ ] Login exitoso → redirige a `/admin`
- [ ] La Navbar muestra el email del usuario logueado
- [ ] El botón "Cerrar sesión" reemplaza a "Iniciar sesión"

**Sesión persistente**
- [ ] Recargar con sesión activa → aparece "Verificando sesión..." brevemente y luego el contenido (sin flash de redirección)
- [ ] Recargar sin sesión → `/admin` redirige a `/login`

**Logout**
- [ ] "Cerrar sesión" en Navbar → redirige a `/login`
- [ ] Intentar acceder a `/admin` tras logout → redirige a `/login`
- [ ] La Navbar vuelve a mostrar "Iniciar sesión"

---

## ¿Qué aprendiste hoy?

| Concepto | Dónde lo aplicaste |
|----------|-------------------|
| `supabase.auth.signUp` | `Register.tsx` — crear usuario nuevo |
| `supabase.auth.signInWithPassword` | `useAuth` → llamado desde `Login.tsx` |
| `supabase.auth.signOut` | `useAuth` → llamado desde `Navbar.tsx` |
| `supabase.auth.getSession` | `useAuth` — leer sesión guardada en localStorage |
| `supabase.auth.onAuthStateChange` | `useAuth` — mantener el estado sincronizado |
| Tipo `User` de Supabase JS | `useAuth` — tipado del estado `usuario` |
| Custom hook `useAuth` | centraliza toda la lógica de auth |
| Estado `cargando` en `ProtectedRoute` | evitar flash de redirección al recargar |
| `throw new Error` desde un hook | propagar errores hacia el componente que llama |
| Renderizado condicional en `Navbar` | `usuario ? bloqueAdmin : botonLogin` |
| `finally` en try/catch | rehabilitar el botón siempre, haya error o no |

---

*SIS-0300 · Universidad Privada Domingo Savio · Tarija, Bolivia*
