## Descripción General

`sysinfo` es una crate que permite interactuar con el sistema operativo para extraer datos en tiempo real sobre el hardware y el software. Proporciona una interfaz unificada para Windows, Linux, macOS, Android e iOS, abstrayendo las complejidades de leer archivos como `/proc` en Linux o usar APIs nativas en Windows.

Su diseño se basa en un modelo de **"snapshot"**: los datos no se actualizan solos de forma pasiva; el programador debe solicitar explícitamente un "refresh" para que la librería consulte al sistema operativo.

## Casos de Uso

*   **Monitores de Sistema:** Creación de herramientas tipo `htop` o administradores de tareas.
*   **Logging y Diagnóstico:** Registrar el estado del servidor (RAM libre, carga de CPU) junto con los errores de la aplicación.
*   **Gestión de Procesos:** Buscar procesos por nombre, obtener su PID o matarlos desde Rust.
*   **Auditoría de Hardware:** Detectar el número de núcleos, temperatura de componentes y capacidad de discos.

## Estructuras y Métodos Principales

*   **`System`**: La estructura central que contiene toda la información (CPU, memoria, procesos).
*   **`Process`**: Representa un proceso individual (uso de memoria, CPU, ruta del ejecutable).
*   **`Cpu`**: Información detallada de cada núcleo (frecuencia, marca, uso porcentual).
*   **`Disks` / `Networks`**: Colecciones para manejar almacenamiento e interfaces de red.
*   **Métodos `refresh_*`**: Vitales para obtener datos actuales (`refresh_all`, `refresh_cpu_usage`, `refresh_memory`, etc.).

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
sysinfo = "0.31" # Versión actual estable
```

### Ejemplo 1: Estadísticas de Memoria y CPU Global
Para obtener el uso de CPU, es obligatorio llamar a la actualización **dos veces** con un pequeño intervalo, ya que el cálculo se basa en la diferencia de tiempo.

```rust
use sysinfo::{System, MINIMUM_CPU_UPDATE_INTERVAL};
use std::thread;

fn main() {
    let mut sys = System::new_all();

    // Primera lectura (necesaria para establecer el punto de partida)
    sys.refresh_cpu_usage();
    
    // Esperar un poco para que haya una diferencia medible
    thread::sleep(MINIMUM_CPU_UPDATE_INTERVAL);
    
    // Segunda lectura
    sys.refresh_cpu_usage();
    sys.refresh_memory();

    println!("=> Sistema:");
    println!("Nombre del SO:      {:?}", System::name());
    println!("Memoria Total:      {} KB", sys.total_memory() / 1024);
    println!("Memoria Usada:      {} KB", sys.used_memory() / 1024);
    println!("Uso Global CPU:     {:.2}%", sys.global_cpu_info().cpu_usage());
}
```

### Ejemplo 2: Listar y Filtrar Procesos
Cómo buscar procesos específicos y obtener su consumo de recursos.
	
```rust
use sysinfo::{System, Pid};

fn main() {
    let mut sys = System::new_all();
    sys.refresh_processes(sysinfo::ProcessesToUpdate::All, true);

    println!("=> Top procesos por uso de memoria:");
    
    // Convertir a vector para ordenar por memoria
    let mut procesos: Vec<_> = sys.processes().values().collect();
    procesos.sort_by(|a, b| b.memory().cmp(&a.memory()));

    for p in procesos.iter().take(5) {
        println!(
            "PID: {:<10} | Nombre: {:<20} | Memoria: {} MB",
            p.pid(),
            p.name().to_string_lossy(),
            p.memory() / 1024 / 1024
        );
    }
}
```

### Ejemplo 3: Información de Discos y Red
Ideal para herramientas que monitorean el almacenamiento o el tráfico.

```rust
use sysinfo::{Disks, Networks};

fn main() {
    // Los discos y redes se manejan ahora a menudo con sus propias structs dedicadas
    let disks = Disks::new_with_refreshed_list();
    println!("=> Discos:");
    for disk in &disks {
        println!(
            "Punto de montaje: {:?} | Espacio libre: {} GB",
            disk.mount_point(),
            disk.available_space() / 1024 / 1024 / 1024
        );
    }

    let networks = Networks::new_with_refreshed_list();
    println!("\n=> Redes:");
    for (interface_name, data) in &networks {
        println!(
            "Interfaz: {:<10} | Recibido: {} B | Transmitido: {} B",
            interface_name,
            data.received(),
            data.transmitted()
        );
    }
}
```

## Buenas Prácticas y Consideraciones

1.  **Reutiliza la instancia de `System`**: No crees un `System::new()` dentro de un bucle. Instáncialo una vez y usa los métodos `refresh` sobre esa misma instancia. Esto ahorra muchas asignaciones de memoria y mejora drásticamente el rendimiento.
2.  **Refresca solo lo necesario**: `refresh_all()` es costoso. Si solo necesitas el uso de memoria, usa `refresh_memory()`. Si solo necesitas un proceso por su PID, usa `refresh_process(pid)`.
3.  **El dilema de la CPU**: Recuerda que `cpu_usage()` devolverá `0.0` la primera vez que se llame. Siempre requiere dos llamadas con un tiempo de espera (mínimo unos 200ms) entre ellas para calcular el diferencial.
4.  **Permisos**: En sistemas como Linux, obtener ciertos detalles de procesos de otros usuarios requiere permisos de root. Si no los tienes, algunos campos podrían aparecer vacíos o como `0`.
5.  **Multi-threading**: Por defecto, `sysinfo` usa múltiples hilos para acelerar el refresco de datos. Si tu aplicación es muy sensible al uso de memoria o hilos (ej. sistemas embebidos), puedes desactivar la feature `multithread` en tu `Cargo.toml`.
