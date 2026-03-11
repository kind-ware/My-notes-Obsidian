## Descripción General

`indicatif` es una librería diseñada para indicar el progreso de tareas en aplicaciones de consola. Permite crear barras de progreso altamente personalizables y *spinners* (indicadores de carga) que no bloquean la lógica principal del programa. 

Es extremadamente ligera, se integra bien con hilos y entornos asíncronos, y maneja automáticamente la limpieza de la pantalla para que los mensajes no se "amontonen" mientras la barra avanza.

### Casos de Uso
*   **Descargas de Archivos:** Mostrar el porcentaje, la velocidad y el tiempo estimado (ETA).
*   **Procesamiento de Datos:** Visualizar el avance de un bucle que procesa miles de registros.
*   **Instaladores/Scripts de Setup:** Usar *spinners* para tareas donde no conoces el tiempo exacto de finalización.
*   **Tareas Concurrentes:** Mostrar múltiples barras de progreso al mismo tiempo (ej: bajando 3 archivos a la vez).

## Estructuras y Funciones Principales

*   **`ProgressBar`**: El objeto principal que representa una barra o un spinner.
*   **`ProgressStyle`**: Define la apariencia (estilo, colores, caracteres) mediante plantillas (*templates*).
*   **`MultiProgress`**: Un contenedor para gestionar y dibujar varias barras simultáneamente sin que se pisen entre sí.
*   **`ProgressIterator`**: Un trait de conveniencia que permite añadir una barra de progreso a cualquier iterador de Rust simplemente llamando a `.progress()`.

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
indicatif = "0.17"
rand = "0.8" # Para simular delays
```

### Ejemplo 1: Barra de Progreso Clásica
Configuración de estilo y actualización manual.

```rust
use indicatif::{ProgressBar, ProgressStyle};
use std::thread;
use std::time::Duration;

fn main() {
    let pasos = 100;
    let pb = ProgressBar::new(pasos);
    
    // Personalizar el estilo: {bar} es la barra, {pos}/{len} la posición, {msg} el mensaje
    pb.set_style(ProgressStyle::with_template("{spinner:.green} [{elapsed_precise}] [{bar:40.cyan/blue}] {pos}/{len} ({eta})")
        .unwrap()
        .progress_chars("#>-"));

    for i in 0..pasos {
        pb.set_message(format!("Procesando archivo_{}.txt", i));
        pb.inc(1); // Incrementar la barra
        thread::sleep(Duration::from_millis(30));
    }

    pb.finish_with_message("¡Todo listo!");
}
```

### Ejemplo 2: Spinner para Tareas de Duración Desconocida
Ideal para cuando esperas una respuesta de una API o una base de datos.

```rust
use indicatif::{ProgressBar, ProgressStyle};
use std::thread;
use std::time::Duration;

fn main() {
    let pb = ProgressBar::new_spinner();
    pb.set_style(
        ProgressStyle::with_template("{spinner:.magenta} {msg}")
            .unwrap()
            .tick_strings(&["⠋", "⠙", "⠹", "⠸", "⠼", "⠴", "⠦", "⠧", "⠇", "⠏"]),
    );

    pb.set_message("Conectando con el servidor...");
    
    // Simular trabajo
    for _ in 0..20 {
        pb.tick(); // Hacer que el spinner gire
        thread::sleep(Duration::from_millis(100));
    }

    pb.finish_and_clear(); // Elimina el spinner de la pantalla al terminar
    println!("✔ Conexión establecida.");
}
```

### Ejemplo 3: Barras de Progreso Concurrentes (`MultiProgress`)
Manejo de varias tareas paralelas.

```rust
use indicatif::{MultiProgress, ProgressBar, ProgressStyle};
use std::thread;
use std::time::Duration;

fn main() {
    let m = MultiProgress::new();
    let sty = ProgressStyle::with_template("[{elapsed_precise}] {bar:40.yellow/red} {pos:>7}/{len:7} {msg}")
        .unwrap();

    let pb1 = m.add(ProgressBar::new(100));
    pb1.set_style(sty.clone());
    pb1.set_message("Tarea A");

    let pb2 = m.add(ProgressBar::new(100));
    pb2.set_style(sty);
    pb2.set_message("Tarea B");

    // Ejecutar en hilos separados
    let h1 = thread::spawn(move || {
        for _ in 0..100 {
            pb1.inc(1);
            thread::sleep(Duration::from_millis(20));
        }
        pb1.finish_with_message("A finalizada");
    });

    let h2 = thread::spawn(move || {
        for _ in 0..100 {
            pb2.inc(1);
            thread::sleep(Duration::from_millis(40));
        }
        pb2.finish_with_message("B finalizada");
    });

    h1.join().unwrap();
    h2.join().unwrap();
}
```

## Buenas Prácticas y Consideraciones

1.  **Detección de TTY**: `indicatif` es inteligente, pero si tu programa se ejecuta en un entorno sin terminal (como un pipeline de CI/CD o redirigiendo la salida a un archivo), las barras de progreso pueden ensuciar el log. Puedes usar `.is_hidden()` o detectar si la salida es una terminal antes de mostrar la barra.
2.  **Mensajes Estáticos vs Dinámicos**: Evita usar `println!` mientras una barra de progreso está activa, ya que romperá el dibujo de la consola. En su lugar, usa `pb.println("mensaje")` para que `indicatif` lo imprima correctamente encima de la barra.
3.  **Rendimiento**: No actualices la barra en cada micro-paso si el bucle es extremadamente rápido (millones de iteraciones por segundo). Actualizar la consola es costoso. Considera usar un paso de actualización mayor (ej: cada 100 iteraciones).
4.  **Templates Flexibles**: Aprovecha los modificadores de plantillas. Por ejemplo, `{bytes}/{total_bytes}` formatea automáticamente números grandes a formatos legibles como "1.2 MB".
5.  **Finish**: Siempre llama a un método `finish` (`finish()`, `finish_with_message()`, `finish_and_clear()`). Esto asegura que el cursor de la terminal regrese a su posición correcta y la barra deje de consumir recursos de dibujo.