## Descripción General

`taskschd` permite a los desarrolladores de Rust crear, enumerar, modificar y eliminar tareas programadas en Windows de forma programática. En lugar de lidiar con las complejas llamadas a la API COM `ITaskService` de forma manual (como harías con `winapi`), esta crate ofrece una abstracción mucho más limpia y segura.

Con ella, puedes definir cuándo debe ejecutarse un binario (triggers), bajo qué condiciones (batería, red) y con qué privilegios (usuario normal o sistema).

### Casos de Uso

*   **Instaladores:** Configurar una tarea de actualización automática para una aplicación.
*   **Herramientas de Mantenimiento:** Programar limpiezas de logs o backups periódicos.
*   **Persistencia de Servicios:** Asegurar que un agente o monitor se inicie automáticamente al iniciar sesión el usuario (sin usar la carpeta de inicio).
*   **Automatización de IT:** Crear scripts que programen tareas en máquinas remotas o locales.

## Estructuras y Flujo Principal

El flujo de trabajo sigue la jerarquía oficial de Windows:
1.  **`TaskService`**: La conexión principal al servicio del programador.
2.  **`TaskFolder`**: Las tareas se organizan en carpetas (la raíz es `\`).
3.  **`TaskDefinition`**: El "plano" de la tarea donde defines:
    *   **`RegistrationInfo`**: Autor y descripción.
    *   **`Triggers`**: Cuándo inicia (por tiempo, al iniciar sesión, al arrancar el PC).
    *   **`Actions`**: Qué hace (ejecutar un `.exe`).
    *   **`Settings`**: Comportamientos (detener si dura mucho, permitir ejecución bajo batería, etc.).

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
taskschd = "0.2"
```

### Ejemplo 1: Conectar y Listar Tareas en la Raíz
Un ejemplo sencillo para ver qué tareas tiene el sistema actualmente.

```rust
use taskschd::TaskService;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 1. Conectar al servicio de tareas
    let service = TaskService::new()?;
    service.connect()?;

    // 2. Obtener la carpeta raíz
    let root_folder = service.get_root_folder()?;

    // 3. Enumerar tareas registradas
    println!("Tareas encontradas en la raíz:");
    for task in root_folder.get_tasks()?.iter() {
        println!(" - Nombre: {}", task.get_name()?);
        println!("   Estado: {:?}", task.get_state()?);
    }

    Ok(())
}
```

### Ejemplo 2: Crear una Tarea al Iniciar Sesión (Logon)
Este es el caso típico para aplicaciones que deben arrancar con el usuario.

```rust
use taskschd::{TaskService, Action, Trigger};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let service = TaskService::new()?;
    service.connect()?;
    let folder = service.get_root_folder()?;

    // Crear la definición de la tarea
    let mut definition = service.new_task_definition()?;
    
    // 1. Configurar la Acción (Ejecutar un programa)
    let mut actions = definition.get_actions()?;
    let action = actions.create_exec_action()?;
    action.set_path(r"C:\Windows\System32\notepad.exe")?;
    
    // 2. Configurar el Trigger (Al iniciar sesión)
    let mut triggers = definition.get_triggers()?;
    triggers.create_logon_trigger()?;

    // 3. Registrar la tarea (Si ya existe, la sobreescribe)
    folder.register_task_definition(
        "MiTareaRust",
        &definition,
        taskschd::TASK_CREATE_OR_UPDATE,
        None, // Usuario actual
        None, // Sin contraseña
        taskschd::TASK_LOGON_INTERACTIVE_TOKEN,
    )?;

    println!("Tarea 'MiTareaRust' creada con éxito.");
    Ok(())
}
```

### Ejemplo 3: Eliminar una Tarea Existente
Gestión de limpieza al desinstalar o actualizar tu software.

```rust
use taskschd::TaskService;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let service = TaskService::new()?;
    service.connect()?;
    let folder = service.get_root_folder()?;

    // Eliminar la tarea por su nombre
    match folder.delete_task("MiTareaRust") {
        Ok(_) => println!("Tarea eliminada."),
        Err(e) => eprintln!("No se pudo eliminar la tarea: {}", e),
    }

    Ok(())
}
```

## Buenas Prácticas y Consideraciones

1.  **Privilegios de Administrador:** Para crear tareas que se ejecuten a nivel de sistema (`TASK_LOGON_SERVICE_ACCOUNT`) o para tocar carpetas protegidas, tu aplicación de Rust **debe ejecutarse con privilegios de administrador**. De lo contrario, recibirás un error de "Acceso Denegado".
2.  **Rutas Absolutas:** El Programador de Tareas no entiende de "rutas relativas" bien. Siempre proporciona la ruta completa al ejecutable (ej: `C:\Program Files\MyApp\app.exe`) y asegúrate de configurar el "Directorio de trabajo" si tu app lo necesita.
3.  **Manejo de COM:** Al igual que `wmi`, esta librería depende de COM. Si la usas en hilos secundarios, asegúrate de inicializar el runtime de COM en ese hilo.
4.  **Settings de Energía:** Por defecto, muchas tareas en Windows están configuradas para "No ejecutarse si el equipo está usando batería". Si tu tarea es crítica, asegúrate de modificar los `Settings` de la `TaskDefinition` para desactivar esa restricción.
5.  **Sobreescritura:** Al registrar una tarea, usa la flag `TASK_CREATE_OR_UPDATE`. Esto evita errores si intentas crear una tarea que ya existe, permitiéndote actualizar la configuración (por ejemplo, cambiar la ruta del ejecutable en una actualización).