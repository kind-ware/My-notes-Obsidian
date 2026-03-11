## Descripción General

`rand` es un framework completo para la generación de números aleatorios. Se basa en tres conceptos clave:
1.  **Generadores (RNGs):** La fuente de aleatoriedad (puede ser rápida, segura criptográficamente o reproducible mediante semillas).
2.  **Distribuciones:** Determinan cómo se reparten los números (uniforme, normal, booleana, etc.).
3.  **Traits (`Rng`):** Definen los métodos para extraer valores del generador.

### Casos de Uso

*   **Videojuegos:** Generación de niveles, probabilidad de crítico o daño aleatorio.
*   **Simulaciones:** Métodos de Monte Carlo o modelos estadísticos.
*   **Seguridad:** Generación de tokens, sales (salts) o identificadores únicos (aunque para criptografía pura se suele usar `ring`, `rand` ofrece generadores seguros).
*   **Pruebas (Testing):** Generación de datos de prueba variados.

## Funciones y Traits Principales

*   **`thread_rng()`**: La función más usada; devuelve un generador de números aleatorios que es eficiente y seguro para el hilo actual.
*   **`Rng` (Trait)**: Proporciona métodos como `gen()`, `gen_range()`, `gen_bool()`.
*   **`random::<T>()`**: Una función de atajo para generar un valor aleatorio de tipo `T`.
*   **`SliceRandom` (Trait)**: Añade métodos a los arrays/vectores para elegir elementos al azar o mezclarlos (`shuffle`).
*   **`StdRng` / `SmallRng`**: Generadores específicos que permiten usar una **semilla (seed)** para que los resultados sean reproducibles.

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
rand = "0.8"
```

### Ejemplo 1: Generación Básica y Rangos
Cómo obtener números, booleanos y valores dentro de un rango específico.

```rust
use rand::prelude::*; // Importa los traits necesarios

fn main() {
    let mut rng = thread_rng();

    // 1. Número entero aleatorio (cualquier valor del tipo u32)
    let n: u32 = rng.gen();
    println!("Número aleatorio: {}", n);

    // 2. Número en un rango [min, max) -> incluye el 1, no el 101
    let puntuacion = rng.gen_range(1..=100);
    println!("Puntuación (1-100): {}", puntuacion);

    // 3. Probabilidad (ej: 75% de probabilidad de éxito)
    if rng.gen_bool(0.75) {
        println!("¡Evento exitoso!");
    }
}
```

### Ejemplo 2: Trabajando con Colecciones (Slices)
Elegir elementos aleatorios de una lista o desordenarla.

```rust
use rand::seq::SliceRandom; // Trait necesario para listas
use rand::thread_rng;

fn main() {
    let mut rng = thread_rng();
    let mut frutas = vec!["Manzana", "Banana", "Naranja", "Pera", "Uva"];

    // 1. Elegir un elemento al azar (devuelve Option porque la lista podría estar vacía)
    if let Some(fruta_del_dia) = frutas.choose(&mut rng) {
        println!("Hoy comeré: {}", fruta_del_dia);
    }

    // 2. Mezclar (Shuffle) la lista original
    frutas.shuffle(&mut rng);
    println!("Lista mezclada: {:?}", frutas);
}
```

### Ejemplo 3: Aleatoriedad Reproducible (Semillas)
Útil para debugging o juegos donde quieres que un "mundo" sea igual si usas el mismo código de semilla.

```rust
use rand::prelude::*;
use rand_chacha::ChaCha8Rng; // Requiere la crate 'rand_chacha'

fn main() {
    let semilla = [42u8; 32]; // Una semilla fija de 32 bytes
    
    // Crear un generador basado en la semilla
    let mut rng = ChaCha8Rng::from_seed(semilla);

    let n1: u32 = rng.gen();
    let n2: u32 = rng.gen();

    // Estos números serán siempre los mismos cada vez que corras el programa
    println!("Valores deterministas: {} y {}", n1, n2);
}
```

## Buenas Prácticas y Consideraciones

1.  **Reutiliza `thread_rng()`**: No es necesario llamar a `thread_rng()` en cada iteración de un bucle. Obtén el manejador una vez fuera del bucle y úsalo dentro para mayor rendimiento.
2.  **Seguridad Criptográfica**: `thread_rng()` utiliza algoritmos que son seguros para la mayoría de los casos de seguridad. Sin embargo, si trabajas en algo extremadamente sensible, asegúrate de que el generador implemente el trait `CryptoRng`.
3.  **Rendimiento vs Calidad**: Si necesitas millones de números aleatorios por segundo y no te importa la seguridad (por ejemplo, para una simulación física), puedes usar `SmallRng`, que es mucho más rápido pero menos "impredecible".
4.  **Uso de Rangos**: Prefiere `gen_range(1..=10)` (inclusivo) o `gen_range(1..10)` (exclusivo) en lugar de usar el operador módulo (`%`), ya que el módulo puede introducir sesgos estadísticos (hacer que unos números salgan ligeramente más que otros).
5.  **Crates complementarias**: Para distribuciones estadísticas avanzadas (Normal, Poisson, etc.), se recomienda usar la crate hermana **`rand_distr`**.
