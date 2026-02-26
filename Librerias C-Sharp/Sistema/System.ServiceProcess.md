## Descripción General

Este espacio de nombres proporciona las clases necesarias para implementar, instalar y controlar **Servicios de Windows**. Un servicio es una aplicación de larga duración que se ejecuta en su propia sesión de Windows, no tiene interfaz de usuario propia y suele configurarse para iniciarse automáticamente cuando arranca el sistema operativo.

#### Casos de Uso Comunes

*   **Servidores de Aplicaciones:** Programas que escuchan peticiones en red (como un servidor web o de base de datos).
*   **Agentes de Monitoreo:** Escanear constantemente carpetas, puertos o el rendimiento del sistema.
*   **Procesamiento de Colas:** Leer mensajes de una cola (RabbitMQ, Azure Service Bus) y procesarlos en segundo plano.
*   **Tareas Críticas:** Procesos que no deben cerrarse si el usuario cierra su sesión.

## Clases y Métodos Principales

La librería tiene dos caras: **Crear** un servicio y **Controlar** servicios existentes.

#### A. Para crear un Servicio:

*   **`ServiceBase`:** La clase de la que debes heredar. Es el "esqueleto" de tu servicio.
    *   `.OnStart(string[] args)`: El código que se ejecuta cuando el servicio arranca.
    *   `.OnStop()`: El código de limpieza cuando el servicio se detiene.
    *   `.OnPause()` / `.OnContinue()`: Control opcional del estado.

#### B. Para controlar otros Servicios:

*   **`ServiceController`:** Permite interactuar con cualquier servicio instalado en el PC (como el SCM - Service Control Manager).
    *   `.Start()`, `.Stop()`, `.Pause()`, `.Continue()`.
    *   `.Status`: Indica si el servicio está `Running`, `Stopped`, `Pending`, etc.
    *   `.WaitForStatus(status)`: Bloquea el código hasta que el servicio llegue al estado deseado (muy útil para reinicios).

## Ejemplos Prácticos

#### Ejemplo 1: Estructura básica de un Servicio de Windows

```csharp
using System;
using System.ServiceProcess;
using System.IO;

public class MiServicio : ServiceBase
{
    public MiServicio()
    {
        this.ServiceName = "MiServicioDeLog";
    }

    protected override void OnStart(string[] args)
    {
        // El código aquí debe ser rápido. Si el trabajo es largo, usa un hilo/Task.
        File.AppendAllText(@"C:\log_servicio.txt", $"Servicio iniciado a las: {DateTime.Now}\n");
    }

    protected override void OnStop()
    {
        File.AppendAllText(@"C:\log_servicio.txt", $"Servicio detenido a las: {DateTime.Now}\n");
    }
}

// En el Program.cs
static void Main()
{
    ServiceBase.Run(new MiServicio());
}
```

#### Ejemplo 2: Controlar un servicio existente (Reiniciar la cola de impresión)

```csharp
using System.ServiceProcess;

string nombreServicio = "Spooler"; // Cola de impresión de Windows
using (ServiceController sc = new ServiceController(nombreServicio))
{
    if (sc.Status == ServiceControllerStatus.Running)
    {
        sc.Stop();
        sc.WaitForStatus(ServiceControllerStatus.Stopped);
        Console.WriteLine("Servicio detenido.");
    }
    
    sc.Start();
    sc.WaitForStatus(ServiceControllerStatus.Running);
    Console.WriteLine("Servicio reiniciado.");
}
```

## Buenas Prácticas y Consideraciones

1.  **Aislamiento de la "Sesión 0":** Los servicios corren en una sesión especial llamada "Sesión 0". **No pueden mostrar ventanas, formularios ni MessageBox.** Si intentas abrir un `Form`, el servicio fallará o se quedará colgado porque no hay nadie para darle "Aceptar".
2.  **No bloquees el `OnStart`:** Windows espera que el método `OnStart` termine en pocos segundos. Si tienes una tarea pesada (como un bucle infinito), **lánzala en un hilo separado o en una `Task`** y deja que `OnStart` finalice. De lo contrario, Windows dirá: "El servicio no respondió a tiempo".
3.  **Registro de Errores (Logging):** Como no hay consola, debes usar el **Visor de Eventos de Windows** (`System.Diagnostics.EventLog`) o una librería como NLog/Serilog para saber qué está pasando.
4.  **Instalación:** Un `.exe` de servicio no se instala haciendo doble clic. Debes usar la herramienta `sc create` desde la consola de comandos o usar clases como `ServiceInstaller` dentro de tu proyecto.
5.  **Modernidad (.NET 6+):** En las versiones actuales de .NET, la forma recomendada de crear servicios es usar el **Worker Service Template** (clase `BackgroundService`), que es multiplataforma y mucho más fácil de testear, aunque internamente en Windows sigue apoyándose en estos conceptos.