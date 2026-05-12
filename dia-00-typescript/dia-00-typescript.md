# Laboratorio — Día 0: TypeScript Esencial

**Asignatura:** Programación Web I — SIS-0300  
**Carrera:** Ingeniería de Sistemas · UPDS Tarija  
**Entorno:** VSCode + ts-node · Windows

---

## Objetivo

Escribir y ejecutar TypeScript puro desde VSCode antes de arrancar con React. Al terminar este laboratorio vas a tener la base que necesitás para tipar componentes, props y datos durante todo el módulo.

---

## Regla del laboratorio

No uses `any` en ningún ejercicio. Si TypeScript muestra un error, léelo antes de buscar cómo evitarlo — el mensaje dice exactamente qué está fallando y por qué.

---

## Configuración inicial

Abrir VSCode. Luego abrir la terminal integrada con `Ctrl + J` y ejecutar los siguientes comandos **uno por uno**:

### Paso 1 — Crear la carpeta del laboratorio

```bash
mkdir lab-typescript
cd lab-typescript
```

### Paso 2 — Inicializar el proyecto

```bash
npm init -y
```

### Paso 3 — Instalar dependencias

```bash
npm install -D typescript ts-node @types/node
```

### Paso 4 — Crear el archivo tsconfig.json

Este archivo es **obligatorio**. Sin él, Node.js v18+ no sabe cómo compilar TypeScript y lanza un error `ERR_UNKNOWN_FILE_EXTENSION` antes de ejecutar una sola línea.

Crear el archivo `tsconfig.json` en la raíz del proyecto con este contenido:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "CommonJS",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "./dist"
  },
  "include": ["*.ts"]
}
```

> **¿Por qué este archivo?** `"module": "CommonJS"` le dice a ts-node que compile el código a CommonJS antes de ejecutarlo. Sin eso, Node.js intenta cargar el `.ts` directamente como módulo ES, lo cual no soporta. En el Día 1 vas a ver que el proyecto React ya trae su propio `tsconfig.json` generado automáticamente por Vite — es el mismo concepto.

### Paso 5 — Crear el archivo de trabajo

Crear el archivo `ejercicios.ts` desde VSCode: `File → New File → guardar como ejercicios.ts` dentro de la carpeta `lab-typescript`.

### Paso 6 — Verificar que todo funciona

Escribir esto en `ejercicios.ts`:

```ts
const mensaje: string = "Hola TypeScript";
console.log(mensaje);
```

Ejecutar en la terminal:

```bash
npx ts-node ejercicios.ts
```

Debe imprimir `Hola TypeScript` sin errores. **No avanzar al Ejercicio 1 hasta que esto funcione.**

### Si aparece este error

```
File cannot be loaded because running scripts is disabled on this system.
```

Ejecutar este comando una sola vez en PowerShell para habilitar scripts:

```bash
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Confirmar con `S` cuando pregunte. Luego volver a ejecutar `npx ts-node ejercicios.ts`.

---

## Ejercicio 1 — Tipos primitivos

### Ejemplo en clase

```ts
// Declaración explícita de tipo
let nombre: string = "Laptop HP";
let precio: number = 4500;
let activo: boolean = true;

// Inferencia — TypeScript deduce el tipo solo
let categoria = "Electrónica"; // TypeScript infiere: string
let stock = 12; // TypeScript infiere: number

// Arrays tipados
let productos: string[] = ["Laptop", "Mouse", "Teclado"];
let precios: number[] = [4500, 180, 350];
```

### Tu turno variables con tipo explícito que representen un producto: `nombre`, `precio`, `stock`, `disponible`. Usar el tipo correcto para cada una.

**2.** Crear un array `categorias: string[]` con al menos cuatro categorías de productos.

**3.** Crear un array `precios: number[]` con al menos cuatro precios.

**4.** Declarar tres variables sin especificar el tipo — dejar que TypeScript lo infiera. Luego intentar reasignarlas con un valor de tipo diferente. Anotar en un comentario qué error aparece.

**5.** Declarar `let codigo: number` y asignarle `"ABC-001"`. Leer el error y escribirlo en un comentario.

---

## Ejercicio 2 — Funciones tipadas

### Ejemplo en clase

```ts
// Parámetros y retorno tipados
function calcularTotal(precio: number, cantidad: number): number {
  return precio * cantidad;
}

// void: la función no devuelve nada
function mostrarMensaje(mensaje: string): void {
  console.log(mensaje);
}

// Parámetro opcional con ?
function formatearPrecio(precio: number, moneda?: string): string {
  return `${moneda ?? "Bs."} ${precio}`;
}

console.log(calcularTotal(4500, 3)); // 13500
console.log(formatearPrecio(180)); // Bs. 180
console.log(formatearPrecio(180, "USD")); // USD 180
```

### Tu turno

**1.** Escribir `calcularDescuento(precio: number, porcentaje: number): number` que devuelva el precio con el descuento aplicado.  
Ejemplo: `calcularDescuento(1000, 10)` → `900`.

**2.** Escribir `mostrarProducto(nombre: string, precio: number): void` que imprima en consola con el formato `"Laptop HP - Bs. 4500"`.

**3.** Escribir `saludar(nombre: string, cargo?: string): string` que devuelva `"Hola Ana, Admin"` si se pasa cargo, o `"Hola Ana"` si no se pasa.

**4.** Llamar cada función con tipos incorrectos — por ejemplo `calcularDescuento("mil", 10)`. Anotar en comentarios los errores que aparecen.

---

## Ejercicio 3 — Interfaces

### Ejemplo en clase

```ts
// Interface: define la forma que debe tener un objeto
interface Producto {
  id: number;
  nombre: string;
  precio: number;
  stock: number;
}

// El objeto DEBE cumplir todos los campos
const laptop: Producto = {
  id: 1,
  nombre: "Laptop HP",
  precio: 4500,
  stock: 12,
};

// Función que recibe un Producto tipado
function mostrarProducto(p: Producto): void {
  console.log(`${p.nombre} - Bs. ${p.precio}`);
}

// Array tipado con la interface
const inventario: Producto[] = [
  { id: 1, nombre: "Laptop HP", precio: 4500, stock: 12 },
  { id: 2, nombre: "Mouse Logitech", precio: 180, stock: 0 },
];
```

### Tu turno

**1.** Definir la interface `Producto` con los campos: `id`, `nombre`, `precio`, `stock`, `categoria`. Usar el tipo correcto para cada uno.

**2.** Crear tres objetos que cumplan esa interface con datos reales de productos.

**3.** Crear un array `inventario: Producto[]` con esos tres objetos.

**4.** Escribir `buscarPorId(inventario: Producto[], id: number): Producto` que recorra el array y devuelva el producto con ese id. Usar `.find()`.

**5.** Intentar crear un objeto al que le falte el campo `precio`. Leer el error y anotarlo en un comentario.

---

## Ejercicio 4 — Props opcionales y union types

### Ejemplo en clase

```ts
// Campo opcional con ?
interface Producto {
  id: number;
  nombre: string;
  precio: number;
  stock: number;
  descripcion?: string; // puede o no estar
}

// Union type: acepta más de un tipo
interface ResultadoBusqueda {
  producto: Producto | null;
  encontrado: boolean;
}

// Ambos objetos son válidos
const conDesc: Producto = {
  id: 1,
  nombre: "Laptop HP",
  precio: 4500,
  stock: 12,
  descripcion: "Procesador i7, 16GB RAM",
};

const sinDesc: Producto = {
  id: 2,
  nombre: "Mouse",
  precio: 180,
  stock: 5,
  // descripcion no es obligatoria
};
```

### Tu turno

**1.** Actualizar la interface `Producto` del Ejercicio 3 para que `descripcion` y `categoria` sean opcionales con `?`.

**2.** Crear dos objetos: uno con los campos opcionales completos y otro sin ellos. Verificar que TypeScript acepta ambos sin errores.

**3.** Definir la interface `ResultadoBusqueda` con `producto: Producto | null` y `encontrado: boolean`.

**4.** Escribir `buscarProducto(inventario: Producto[], nombre: string): ResultadoBusqueda` que devuelva el resultado correcto si encuentra o no el producto.

**5.** Llamar a `buscarProducto` con un nombre que exista y con uno que no exista. Imprimir ambos resultados en consola.

---

## Ejercicio 5 — Integrador

Sin ejemplo previo. Usá todo lo aprendido en los ejercicios anteriores. El docente asiste individualmente.

### Consigna

Construir un mini sistema de gestión de productos en TypeScript puro.

```ts
// 1. Definir la interface Producto
//    Campos requeridos: id, nombre, precio, stock
//    Campos opcionales: categoria, descripcion

// 2. Crear inventario: Producto[] con al menos 5 productos
//    Al menos 2 sin stock, algunos con campos opcionales

// 3. agregarProducto(inventario: Producto[], producto: Producto): Producto[]
//    Devuelve el inventario actualizado con el nuevo producto

// 4. filtrarPorStock(inventario: Producto[]): Producto[]
//    Devuelve solo los productos con stock mayor a 0

// 5. calcularValorTotal(inventario: Producto[]): number
//    Devuelve la suma de (precio × stock) de todos los productos

// 6. buscarPorNombre(inventario: Producto[], nombre: string): Producto | null
//    Devuelve el producto encontrado o null si no existe

// 7. productoMasCaro(inventario: Producto[]): Producto | null
//    Devuelve el producto con el precio más alto
//    Si el inventario está vacío devuelve null

// 8. actualizarStock(inventario: Producto[], id: number, nuevoStock: number): Producto[]
//    Devuelve el inventario con el stock del producto actualizado
//    Si el id no existe devuelve el inventario sin cambios

// 9. Ejecutar todo y mostrar resultados en consola
```

### Resultado esperado en consola

```
Inventario inicial: 5 productos
Después de agregar: 6 productos
Con stock disponible: 3 productos
Valor total del inventario: Bs. 58450
Buscar "Laptop": Laptop HP - Bs. 4500
Buscar "Impresora": No encontrado
Producto más caro: Laptop HP - Bs. 4500
Stock actualizado de id 2: 15 unidades
```
