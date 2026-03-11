## Descripción General

La librería estándar de Rust (`std`) es el conjunto de funcionalidades críticas que permiten interactuar con el sistema operativo y proporcionan tipos de datos fundamentales. A diferencia de otros lenguajes, `std` en Rust es relativamente pequeña pero extremadamente potente, diseñada bajo el principio de "abstracciones de coste cero".

Se asienta sobre `core` (funcionalidades que no requieren OS) y `alloc` (gestión de memoria dinámica), añadiendo soporte para entrada/salida (I/O), hilos, red y manejo de archivos.

### Casos de Uso

*   **Gestión de Colecciones:** Almacenamiento dinámico de datos (vectores, mapas, listas).
*   **Entrada/Salida (I/O):** Lectura y escritura de archivos, interacción con la consola.
*   **Concurrencia:** Creación de hilos (threads) y comunicación entre ellos.
*   **Redes:** Creación de servidores y clientes TCP/UDP.
*   **Manejo de Errores:** Tipos robustos para operaciones que pueden fallar.
-   **Entorno**: Variables de entorno, argumentos CLI, rutas del ejecutable.
-   **Rutas**: Manipulación portable de rutas de archivos/carpetas.
## Módulos y Funciones Principales

La librería se organiza en módulos. Los más importantes son:

*   **`std::vec::Vec`**: Un arreglo dinámico (crece en tiempo de ejecución).
*   **`std::string::String`**: Una cadena de caracteres UTF-8 de tamaño variable.
*   **`std::fs`**: Funciones para manipular el sistema de archivos (leer, escribir, borrar).
*   **`std::io`**: Traits y funciones para lectura/escritura (Read, Write, BufReader).
*   **`std::thread`**: Primitivas para la ejecución paralela.
*   **`std::collections`**: Estructuras de datos como `HashMap`, `BTreeMap`, `VecDeque`.
*   **`std::result` / `std::option`**: Los tipos fundamentales para el manejo de errores y valores nulos.
-   `std::env`: Variables de entorno, argumentos, current_exe(), env::var("LOCALAPPDATA"). 
-   `std::path`: Pahtbuf, Path para rutas portables 
-   ``std::net``:  Boost.Asio, común en std o **std::socket** (propuestas): Networking básico (TCP/UDP sockets, conexiones).
-   ``std::time``: Manejo de flujo del programa utilizando tiempos. 
-   ``std::sync``: Sincronización (Mutex, RwLock, Arc).
### Archivos (fs)

- `read_dir(path)` itera sobre contenidos de un directorio.
- `create_dir(path)` crea un directorio vacío.
- `create_dir_all(path)` crea directorios recursivamente.
- `remove_file(path)` elimina un archivo.
- `remove_dir(path)` elimina un directorio vacío.
- `copy(src, dst)` copia archivo preservando permisos.
- `rename(src, dst)` renombra/mueve archivo o directorio.

### Rutas (path)

- `Path::new(&str)` crea un `Path` desde una cadena.
- `is_dir()`, `is_file()` verifica si es directorio o archivo.
- `join(&Path)` une rutas de forma segura.
- `display()` obtiene representación como string.
- `parent()`, `file_name()`, `extension()` extrae componentes de la ruta.
- `canonicalize()` resuelve enlaces simbólicos y obtiene ruta absoluta
- `try_exists()` verifica la exitencia de una ruta

## Ejemplos de Código

### Ejemplo 1: Manejo de Vectores y Colecciones
Este ejemplo muestra cómo crear, modificar y filtrar una lista dinámica de datos.

```rust
fn main() {
    // Crear un vector mutable de enteros
    let mut numeros = vec![10, 20, 30, 40];

    // Añadir un elemento
    numeros.push(50);

    // Iterar usando programación funcional (iteradores)
    let mayores_a_25: Vec<i32> = numeros
        .iter()
        .filter(|&&x| x > 25)
        .map(|&x| x * 2)
        .collect();

    println!("Originales: {:?}", numeros);
    println!("Procesados (Mayores a 25 * 2): {:?}", mayores_a_25);
}
```

### Ejemplo 2: Lectura de Archivos y Manejo de Errores
Rust no utiliza excepciones. Se utiliza el tipo `Result` para manejar posibles fallos de I/O.

```rust
use std::fs::File;
use std::io::{self, Read};

fn leer_configuracion() -> io::Result<String> {
    let mut contenido = String::new();
    // El operador '?' propaga el error si ocurre un fallo al abrir o leer
    let mut archivo = File::open("config.txt")?;
    archivo.read_to_string(&mut contenido)?;
    
    Ok(contenido)
}

fn main() {
    match leer_configuracion() {
        Ok(texto) => println!("Contenido del archivo: {}", texto),
        Err(e) => eprintln!("Error al leer el archivo: {}", e),
    }
}
```

### Ejemplo 3: Concurrencia Básica con Hilos
Uso de `std::thread` para ejecutar tareas en paralelo.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..5 {
            println!("Hilo secundario: mensaje {}", i);
            thread::sleep(Duration::from_millis(500));
        }
    });

    for i in 1..3 {
        println!("Hilo principal: mensaje {}", i);
        thread::sleep(Duration::from_millis(500));
    }

    // Esperar a que el hilo secundario termine
    handle.join().unwrap();
    println!("Simulación finalizada.");
}
```

### Ejemplo 4: Uso de path y env

```rust
use std::env;
use std::fs;
use std::path::Path;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Obtener la variable LOCALAPPDATA
    let localappdata = env::var("LOCALAPPDATA")
        .map_err(|_| "No se encontró LOCALAPPDATA")?;

    // Crear ruta de destino: LOCALAPPDATA\RustTest
    let target_dir = Path::new(&localappdata).join("RustTest");
    let target_path = target_dir.join("mi_ejecutable.exe"); // Ajusta el nombre según tu ejecutable

    // Crear directorio si no existe
    fs::create_dir_all(&target_dir)?;

    // Obtener ruta del ejecutable actual
    let current_exe = env::current_exe()?;

    // Copiar el ejecutable
    fs::copy(&current_exe, &target_path)?;
    
    println!("Ejecutable copiado a: {}", target_path.display());
    println!("LOCALAPPDATA está en: {}", localappdata);

    Ok(())
}

```

### Ejemplo 5: Port Scanner

```rust
use std::net::{TcpStream, ToSocketAddrs};
use std::time::Duration;
use std::thread;
use std::io::ErrorKind;

fn scan_port(host: &str, port: u16, timeout: Duration) -> Result<(), ()> {
    let address = format!("{}:{}", host, port);
    match address.to_socket_addrs() {
        Ok(mut addrs) => {
            if let Some(addr) = addrs.next() {
                match TcpStream::connect_timeout(&addr, timeout) {
                    Ok(_) => {
                        println!("[+] Puerto {}: ABIERTO", port);
                        Ok(())
                    }
                    Err(e) => {
                        if e.kind() == ErrorKind::ConnectionRefused || 
                           e.kind() == ErrorKind::TimedOut {
                            println!("[X] Puerto {}: CERRADO/TIMEOUT", port);
                        }
                        Err(())
                    }
                }
            } else {
                Err(())
            }
        }
        Err(_) => {
            println!("[X] Puerto {}: HOST NO RESOLUCIONABLE", port);
            Err(())
        }
    }
}

fn main() {
    let host = "127.0.0.1"; // Cambia por la IP objetivo
    let start_port = 1;
    let end_port = 100;
    let timeout = Duration::from_millis(100); // 100ms por puerto

    println!("Escaneando {} (puertos {}-{})...", host, start_port, end_port);

    let mut handles = vec![];

    // Escaneo multi-hilo para mayor velocidad
    for port in start_port..=end_port {
        let host_clone = host.to_string();
        let handle = thread::spawn(move || {
            scan_port(&host_clone, port, timeout).ok();
        });
        handles.push(handle);

        // Limitar hilos concurrentes (evita saturar sistema)
        if handles.len() >= 200 {
            for handle in handles.drain(..) {
                handle.join().unwrap();
            }
        }
    }

    // Esperar hilos restantes
    for handle in handles {
        handle.join().unwrap();
    }

    println!("Escaneo completado.");
}
```

## Buenas Prácticas y Consideraciones

1.  **Evitar el uso de `unwrap()`**: En producción, llamar a `.unwrap()` puede causar que el programa se detenga abruptamente (panic). Es preferible usar `match`, `if let` o el operador `?` para manejar errores de forma segura.
2.  **Aprovechar los Iteradores**: Los iteradores en Rust son "lazy" y suelen ser tan rápidos o más que los bucles `for` tradicionales debido a las optimizaciones del compilador (Zero-cost abstractions).
3.  **Strings vs &str**: Recuerda que `String` es una cadena que posees (dueño) y puede crecer, mientras que `&str` es una referencia (slice) a una cadena. Usa `&str` como argumento en funciones para mayor flexibilidad.
4.  **Ownership y Borrowing**: La librería estándar está diseñada para seguir las reglas de pertenencia. Si pasas un objeto a una función de `std`, podrías estar perdiendo su propiedad a menos que pases una referencia (`&`).
5.  **Memoria**: Al usar `std::collections`, ten en cuenta que estructuras como `HashMap` tienen un gasto extra de memoria por el hashing. Evalúa si un simple `Vec` de tuplas es suficiente para pocos datos.