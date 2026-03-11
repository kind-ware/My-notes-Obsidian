## Descripción General

La crate `wmi` proporciona una interfaz de alto nivel para realizar consultas sobre la infraestructura de gestión de Windows. Utiliza **COM** (*Component Object Model*) por debajo para comunicarse con el servicio WMI. 

Su característica más potente es la integración con **Serde**, lo que permite mapear los resultados de consultas WMI (que son tablas de datos) directamente a estructuras de Rust de forma segura y tipada.

### Casos de Uso

*   **Inventario de Hardware Detallado:** Obtener el número de serie de la placa base, el modelo exacto de la memoria RAM o la versión de la BIOS.
*   **Monitorización de Procesos:** Consultar el uso de recursos de procesos específicos con filtros avanzados.
*   **Estado de Red:** Listar adaptadores de red físicos y virtuales con sus direcciones MAC y estados.
*   **Información del Sistema Operativo:** Obtener fechas de instalación, versiones de parches y configuraciones de arranque.

## Funciones y Estructuras Principales

*   **`COMLib`**: Estructura que gestiona la inicialización de la librería COM. Debe estar viva mientras se use WMI.
*   **`WMIConnection`**: El punto de entrada para realizar consultas. Se puede conectar al namespace local (`ROOT\CIMV2`) o a uno remoto.
*   **`query` / `raw_query`**: Ejecuta sentencias en lenguaje **WQL** (WMI Query Language), que es muy similar a SQL.
*   **`#[derive(Deserialize)]`**: Se usa en tus structs para que `wmi` sepa cómo llenar los datos automáticamente.

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
wmi = "0.13"
serde = { version = "1.0", features = ["derive"] }
```

### Ejemplo 1: Obtener información de la BIOS
Este ejemplo muestra cómo inicializar COM y traer datos de una clase específica.

```rust
use wmi::{COMLib, WMIConnection};
use serde::Deserialize;

#[derive(Deserialize, Debug)]
#[serde(rename_all = "PascalCase")] // WMI usa PascalCase (ej: SerialNumber)
struct Win32_BIOS {
    manufacturer: String,
    serial_number: String,
    release_date: String,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 1. Inicializar COM (Obligatorio)
    let com_con = COMLib::new()?;
    
    // 2. Conectar al namespace por defecto
    let wmi_con = WMIConnection::new(com_con.into())?;

    // 3. Ejecutar consulta SELECT
    let resultados: Vec<Win32_BIOS> = wmi_con.query()?;

    for bios in resultados {
        println!("Fabricante: {}, S/N: {}", bios.manufacturer, bios.serial_number);
    }

    Ok(())
}
```

### Ejemplo 2: Listar Procesos con Filtro (WQL)
Podemos usar filtros para no traer todos los datos del sistema, mejorando el rendimiento.

```rust
use wmi::{COMLib, WMIConnection};
use serde::Deserialize;

#[derive(Deserialize, Debug)]
#[serde(rename_all = "PascalCase")]
struct Win32_Process {
    process_id: u32,
    name: String,
    working_set_size: u64, // Memoria usada
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let com_con = COMLib::new()?;
    let wmi_con = WMIConnection::new(com_con.into())?;

    // Consulta con WHERE para buscar un proceso específico
    let query = "SELECT * FROM Win32_Process WHERE Name = 'explorer.exe'";
    let procesos: Vec<Win32_Process> = wmi_con.raw_query(query)?;

    for p in procesos {
        println!("PID: {} | RAM: {} MB", p.process_id, p.working_set_size / 1024 / 1024);
    }

    Ok(())
}
```

### Ejemplo 3: Información de Discos Físicos
Extraer datos que normalmente no están disponibles en la librería estándar.

```rust
use wmi::{COMLib, WMIConnection};
use serde::Deserialize;

#[derive(Deserialize, Debug)]
#[serde(rename_all = "PascalCase")]
struct Win32_DiskDrive {
    model: String,
    size: u64,
    media_type: String,
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let com_con = COMLib::new()?;
    let wmi_con = WMIConnection::new(com_con.into())?;

    let discos: Vec<Win32_DiskDrive> = wmi_con.query()?;

    for d in discos {
        println!("Modelo: {} ({} GB)", d.model, d.size / 1024 / 1024 / 1024);
    }

    Ok(())
}
```

## Buenas Prácticas y Consideraciones

1.  **Ciclo de vida de `COMLib`**: COM debe inicializarse una vez por hilo. Si intentas usar WMI sin una instancia activa de `COMLib` en el hilo actual, el programa fallará. Normalmente se crea una sola vez al inicio del `main`.
2.  **Rendimiento (WMI es lento)**: Consultar WMI es significativamente más lento que leer archivos o usar APIs directas de Windows. Evita realizar consultas WMI en bucles de alta frecuencia (como el frame de un juego o un loop de 1ms).
3.  **Renombrado con Serde**: Las propiedades de WMI suelen usar `PascalCase` (ej: `Caption`, `Model`). Usa `#[serde(rename_all = "PascalCase")]` en tus structs para que coincidan con los nombres de Rust (`caption`, `model`) de forma automática.
4.  **Cuidado con `SELECT *`**: Al igual que en SQL, es mejor pedir solo las columnas que necesitas (`SELECT Name FROM ...`) en lugar de `*`, especialmente en clases con muchas propiedades como `Win32_Process`.
5.  **Namespaces**: Por defecto se usa `ROOT\CIMV2`, pero para algunas cosas (como temperatura del procesador o datos de Hyper-V), podrías necesitar conectar con otros namespaces como `ROOT\WMI` o `ROOT\Virtualization\V2`.
