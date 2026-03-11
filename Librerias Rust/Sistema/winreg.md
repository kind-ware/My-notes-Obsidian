## Descripción General

`winreg` proporciona bindings de Rust para la API del Registro de Windows. A diferencia de usar directamente `winapi` o `windows-sys`, esta librería abstrae la complejidad de los punteros y el manejo de memoria manual, permitiendo trabajar con tipos nativos de Rust (`String`, `u32`, `u64`) de forma casi transparente.

Es una librería madura que soporta tanto operaciones básicas como avanzadas (transacciones, iteradores y serialización con `serde`).

### Casos de Uso

*   **Configuración de Aplicaciones:** Almacenar preferencias persistentes del usuario a nivel de sistema o usuario.
*   **Detección de Software:** Verificar si una aplicación o componente está instalado (ej. buscar la ruta de instalación de Steam o VS Code).
*   **Modificación del Sistema:** Cambiar comportamientos del SO (como el tema oscuro/claro, aplicaciones de inicio o extensiones de archivo).
*   **Auditoría y Diagnóstico:** Leer información de hardware y versiones del sistema operativo almacenadas en el registro.

## Funciones y Estructuras Principales

*   **`RegKey`**: El tipo principal que representa una clave del registro.
*   **`RegKey::predef(HKEY_...)`**: Abre una de las claves raíz predefinidas (`HKEY_LOCAL_MACHINE`, `HKEY_CURRENT_USER`, etc.).
*   **`open_subkey` / `open_subkey_with_flags`**: Abre una subclave existente (por defecto en modo lectura).
*   **`create_subkey`**: Crea una nueva subclave o abre una existente con permisos de escritura.
*   **`get_value` / `set_value`**: Lee o escribe valores mapeando automáticamente tipos de Rust a tipos de Windows (`REG_SZ`, `REG_DWORD`, etc.).
*   **`enum_keys` / `enum_values`**: Devuelve iteradores para recorrer todas las subclaves o valores dentro de una clave.
*   **`delete_subkey` / `delete_value`**: Elimina entradas del registro.

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
winreg = "0.52"
```

### Ejemplo 1: Leer información del sistema
Cómo obtener datos de una ruta conocida del registro.

```rust
use winreg::enums::*;
use winreg::RegKey;

fn main() -> std::io::Result<()> {
    // 1. Abrir la raíz de la máquina local
    let hklm = RegKey::predef(HKEY_LOCAL_MACHINE);
    
    // 2. Abrir una subclave específica (en modo lectura por defecto)
    let subkey = hklm.open_subkey("SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion")?;

    // 3. Obtener valores con tipado automático
    let product_name: String = subkey.get_value("ProductName")?;
    let build_number: String = subkey.get_value("CurrentBuild")?;

    println!("SO: {}", product_name);
    println!("Build: {}", build_number);

    Ok(())
}
```

### Ejemplo 2: Crear una clave y escribir valores
Uso de `create_subkey` para persistir configuración propia.

```rust
use winreg::enums::*;
use winreg::RegKey;
use std::path::Path;

fn main() -> std::io::Result<()> {
    let hkcu = RegKey::predef(HKEY_CURRENT_USER);
    
    // Crear (o abrir si ya existe) la ruta "Software\MiAppRust"
    let path = Path::new("Software").join("MiAppRust");
    let (key, _disp) = hkcu.create_subkey(&path)?;

    // Escribir diferentes tipos de datos
    key.set_value("Version", &"1.0.2")?;      // REG_SZ
    key.set_value("Intentos", &5u32)?;        // REG_DWORD
    key.set_value("Habilitado", &1u32)?;      // REG_DWORD (usado como bool)

    println!("Configuración guardada en HKCU\\Software\\MiAppRust");
    Ok(())
}
```

### Ejemplo 3: Enumerar y eliminar subclaves
Recorrer todas las claves dentro de una carpeta y realizar una limpieza.

```rust
use winreg::enums::*;
use winreg::RegKey;

fn main() -> std::io::Result<()> {
    let hkcu = RegKey::predef(HKEY_CURRENT_USER);
    let subkey_path = "Software\\MiAppRust";

    // Intentar abrir la clave para borrar cosas dentro
    if let Ok(key) = hkcu.open_subkey_with_flags(subkey_path, KEY_ALL_ACCESS) {
        println!("Listando valores:");
        for (name, value) in key.enum_values().map(|x| x.unwrap()) {
            println!("  - {}: {:?}", name, value);
        }

        // Borrar un valor específico
        key.delete_value("Intentos")?;
        println!("Valor 'Intentos' eliminado.");
        
        // Para borrar la clave completa (y todo su contenido):
        // hkcu.delete_subkey_all(subkey_path)?;
    }

    Ok(())
}
```

## Buenas Prácticas y Consideraciones

1.  **Permisos de Administrador**: Escribir en `HKEY_LOCAL_MACHINE` requiere casi siempre privilegios de administrador. Si tu app no los tiene, las llamadas a `create_subkey` o `set_value` en esa raíz fallarán con un error de "Acceso Denegado". Usa `HKEY_CURRENT_USER` para datos de usuario normales.
2.  **Manejo de Errores**: El registro de Windows es un recurso compartido y volátil. Siempre asume que una clave puede no existir o haber sido borrada por otro proceso. Usa el operador `?` o `match` para manejar `std::io::Result`.
3.  **Rutas con Backslashes**: Al definir rutas en strings, recuerda usar "raw strings" `r"Software\App"` o escapar la barra invertida `"Software\\App"`.
4.  **Uso de `delete_subkey_all`**: Ten mucho cuidado con esta función, ya que es recursiva y eliminará toda la rama del registro sin preguntar. Es preferible usar `delete_subkey` (que falla si hay hijos) a menos que estés seguro de querer borrar todo el árbol.
5.  **Serialización con Serde**: Si tu configuración es compleja (structs anidados), activa la feature `serialization-serde`. Esto permite guardar y cargar structs enteros directamente en el registro de Windows como si fuera un archivo JSON o TOML.