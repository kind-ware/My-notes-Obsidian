## Descripción General

`windows-sys` proporciona **bindings crudos (FFI)** para las APIs de Windows (Win32). Es parte del proyecto oficial "Rust for Windows" de Microsoft. A diferencia de la librería `windows` (a secas), `windows-sys` no ofrece abstracciones seguras, ni soporte para COM o WinRT; su único objetivo es proporcionar las declaraciones de funciones, constantes y estructuras tal cual existen en C.

Es la opción preferida cuando el **tiempo de compilación** es crítico o cuando necesitas un control absoluto sobre la memoria sin el "overhead" de las capas de seguridad adicionales de Rust.

### Casos de Uso

*   **Desarrollo de Sistemas:** Creación de servicios de Windows o herramientas de diagnóstico.
*   **Manipulación de Procesos:** Inyección de DLLs, lectura de memoria de otros procesos o enumeración de hilos.
*   **Interfaces Nativas:** Creación de ventanas desde cero sin usar frameworks pesados.
*   **Optimización de Compilación:** Proyectos que necesitan funciones de Windows pero quieren evitar los largos tiempos de compilación de la librería `windows` completa.
*   **Sistemas `no_std`:** Ideal para entornos restringidos donde no hay librería estándar.

## Módulos y Tipos Principales

La librería está dividida en módulos basados en la jerarquía oficial de Windows:
*   **`Win32::Foundation`**: Tipos básicos como `HWND`, `HANDLE`, `BOOL`, `PSTR`, `PWSTR`.
*   **`Win32::System::Threading`**: Funciones para hilos y procesos (ej. `CreateProcessW`, `Sleep`).
*   **`Win32::UI::WindowsAndMessaging`**: Interacción con el usuario (ej. `MessageBoxW`, `CreateWindowExW`).
*   **`Win32::Security`**: Gestión de permisos y tokens de seguridad.
*   **Macros `s!` y `w!`**: Utilizadas para crear cadenas de texto compatibles con ANSI (`PSTR`) y Wide/Unicode (`PWSTR`) directamente desde Rust.

## Ejemplos de Código

### Preparación (Cargo.toml)
Debes activar específicamente las "features" que vas a usar, de lo contrario no tendrás acceso a ninguna función.

```toml
[dependencies]
windows-sys = { version = "0.52", features = [
    "Win32_UI_WindowsAndMessaging",
    "Win32_Foundation",
    "Win32_System_SystemInformation"
] }
```

### Ejemplo 1: Mostrar un Message Box (UI Básica)
Interacción directa con el usuario mediante funciones nativas de Win32.

```rust
use windows_sys::Win32::UI::WindowsAndMessaging::*;

fn main() {
    unsafe {
        // MessageBoxW usa UTF-16 (Wide strings)
        // El prefijo 'w!' ayuda a crear la cadena terminada en nulo para Windows
        MessageBoxW(
            0, 
            windows_sys::w!("Hola desde Rust usando windows-sys"), 
            windows_sys::w!("Título Nativo"), 
            MB_OK | MB_ICONINFORMATION
        );
    }
}
```

### Ejemplo 2: Obtener Información del Sistema
Uso de funciones para leer datos del hardware o del sistema operativo.

```rust
use windows_sys::Win32::System::SystemInformation::*;

fn main() {
    unsafe {
        let ticks = GetTickCount();
        println!("Milisegundos transcurridos desde que arrancó el sistema: {} ms", ticks);
        
        let mut system_info = std::mem::zeroed();
        GetSystemInfo(&mut system_info);
        
        println!("Número de procesadores: {}", system_info.dwNumberOfProcessors);
    }
}
```

### Ejemplo 3: Gestión de Eventos y Sincronización
Uso de handles nativos de Windows para sincronización de bajo nivel.

```rust
use windows_sys::Win32::Foundation::*;
use windows_sys::Win32::System::Threading::*;

fn main() {
    unsafe {
        // Crear un evento manual en el sistema
        let event = CreateEventW(std::ptr::null(), 1, 0, std::ptr::null());
        
        if event == 0 {
            panic!("No se pudo crear el evento");
        }

        println!("Evento creado. Esperando 1 segundo...");
        SetEvent(event); // Señalizar el evento
        
        let result = WaitForSingleObject(event, 1000);
        if result == WAIT_OBJECT_0 {
            println!("El evento fue señalizado correctamente.");
        }

        CloseHandle(event); // ¡Es vital cerrar los handles manualmente!
    }
}
```

## Buenas Prácticas y Consideraciones

1.  **Todo es `unsafe`**: Casi cualquier llamada a `windows-sys` requiere un bloque `unsafe`. Es tu responsabilidad asegurar que los punteros sean válidos y que los buffers tengan el tamaño correcto.
2.  **Gestión de Memoria Manual**: Al ser una capa cruda, Rust no te ayudará con el "Drop". Si abres un `HANDLE`, debes llamar a `CloseHandle`. Si asignas memoria con `HeapAlloc`, debes liberarla con `HeapFree`.
3.  **Strings (ANSI vs Wide)**: 
    *   Las funciones que terminan en **`A`** (ej. `MessageBoxA`) esperan `PSTR` (ANSI, terminadas en `\0`).
    *   Las funciones que terminan en **`W`** (ej. `MessageBoxW`) esperan `PWSTR` (UTF-16, terminadas en `\0\0`). Se recomienda usar siempre las versiones `W` para compatibilidad moderna.
4.  **Cero Overhead**: Usa esta librería solo si necesitas la máxima velocidad de compilación o si estás escribiendo un wrapper seguro por encima. Si prefieres algo más cómodo (que use `Result` en lugar de códigos de error numéricos), usa la crate `windows`.
5.  **Manejo de Errores**: Windows suele devolver `0` o `FALSE` cuando falla. Para saber qué pasó realmente, a menudo tendrás que llamar a `GetLastError()` (del módulo `Win32::Foundation`) justo después del fallo.