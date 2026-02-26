## Descripción General

Es un *wrapper* (envoltorio) de alto nivel sobre la API de Programación de Tareas de Windows (versiones 1.0 y 2.0). Permite a los desarrolladores de C# crear, modificar, eliminar y leer tareas programadas de forma sencilla, sin tener que lidiar con la compleja comunicación COM (Component Object Model) que Windows requiere internamente.

#### Casos de Uso Comunes

*   **Mantenimiento Automático:** Programar un script de limpieza de base de datos que se ejecute todos los domingos a las 3:00 AM.
*   **Actualizaciones de Software:** Crear una tarea que verifique si hay nuevas versiones al iniciar sesión el usuario.
*   **Recordatorios y Notificaciones:** Lanzar un mensaje o una aplicación cuando el sistema detecte que el usuario ha estado inactivo (Idle) por 15 minutos.
*   **Persistencia:** Asegurar que un servicio o proceso se reinicie automáticamente si el PC se apaga o se reinicia.

## Clases y Componentes Principales

El modelo de esta librería sigue la lógica de Windows: una tarea se compone de una **Definición**, **Disparadores** y **Acciones**.

*   **`TaskService`:** Es la clase principal. Representa la conexión con el servicio de programación de tareas del PC (local o remoto).
*   **`TaskDefinition`:** Es el "plano" o configuración de la tarea. Aquí defines la descripción, el autor y los ajustes de seguridad.
*   **`Trigger` (Disparadores):** Definen **cuándo** se ejecuta la tarea.
    *   `DailyTrigger`, `WeeklyTrigger`, `MonthlyTrigger`.
    *   `LogonTrigger` (Al iniciar sesión).
    *   `BootTrigger` (Al encender el PC).
    *   `IdleTrigger` (Cuando el PC está inactivo).
*   **`Action` (Acciones):** Definen **qué** hace la tarea.
    *   `ExecAction`: La más común, ejecuta un programa o script (`.exe`, `.bat`, `.ps1`).
    *   `EmailAction`: Envía un correo (Nota: obsoleta en versiones modernas de Windows).
    *   `ShowMessageAction`: Muestra un mensaje (Nota: obsoleta en versiones modernas de Windows).

## Ejemplos Prácticos

> **Nota:** Para usar esto, debes instalar el paquete NuGet: `TaskScheduler`.

#### Ejemplo 1: Crear una tarea diaria sencilla

```csharp
using System;
using Microsoft.Win32.TaskScheduler;

class Program
{
    static void Main()
    {
        // 1. Conectar al servicio de tareas local
        using (TaskService ts = new TaskService())
        {
            // 2. Crear una nueva definición de tarea
            TaskDefinition td = ts.NewTask();
            td.RegistrationInfo.Description = "Mi tarea de respaldo diaria";

            // 3. Crear un disparador (Trigger): Todos los días a las 10:00 PM
            td.Triggers.Add(new DailyTrigger { StartBoundary = DateTime.Today.AddHours(22) });

            // 4. Crear la acción (Action): Ejecutar un bloc de notas (como ejemplo)
            td.Actions.Add(new ExecAction("notepad.exe", "c:\\test.txt", null));

            // 5. Registrar la tarea en la carpeta raíz del programador
            ts.RootFolder.RegisterTaskDefinition(@"MantenimientoAlex", td);
            
            Console.WriteLine("Tarea programada correctamente.");
        }
    }
}
```

#### Ejemplo 2: Listar todas las tareas activas en el sistema

```csharp
using (TaskService ts = new TaskService())
{
    foreach (Task t in ts.AllTasks)
    {
        Console.WriteLine($"Nombre: {t.Name} | Estado: {t.State} | Última ejecución: {t.LastRunTime}");
    }
}
```

#### Ejemplo 3: Eliminar una tarea

```csharp
using (TaskService ts = new TaskService())
{
    ts.RootFolder.DeleteTask("MantenimientoAlex");
}
```

## Buenas Prácticas y Consideraciones

1.  **Privilegios de Administrador:** Para registrar tareas que se ejecuten "con los privilegios más altos" o para acceder a carpetas críticas del sistema, tu aplicación de C# debe ejecutarse como **Administrador**.
2.  **Uso de `using`:** La clase `TaskService` utiliza recursos COM del sistema. Siempre envuélvela en un bloque `using` para asegurar que la conexión se cierre correctamente.
3.  **Compatibilidad de Versiones:** Si desarrollas para sistemas muy antiguos (como Windows XP), debes especificar en el constructor del `TaskService` que use la versión 1.0 de la API. Para Windows 7 en adelante, la 2.0 es el estándar.
4.  **Rutas Absolutas:** Al configurar una `ExecAction`, usa siempre rutas completas (ej: `C:\Apps\Miprograma.exe`). El programador de tareas no siempre sabe cuál es el "directorio actual".
5.  **Manejo de Errores:** Registrar tareas puede fallar por falta de permisos o por nombres duplicados. Usa bloques `try-catch` específicos para capturar excepciones de COM.

