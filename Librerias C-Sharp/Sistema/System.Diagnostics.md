## Descripción General

`System.Diagnostics` proporciona clases que te permiten interactuar con procesos del sistema, registros de eventos, contadores de rendimiento y herramientas de depuración. Es la librería esencial para medir cuánto tarda en ejecutarse un código, lanzar aplicaciones externas, o escribir logs que solo se vean durante el desarrollo.

#### Casos de Uso Comunes

*   **Gestión de Procesos:** Abrir el navegador, ejecutar un comando en la terminal o listar las aplicaciones que están corriendo en el PC.
*   **Perfilado de Rendimiento:** Medir con precisión de nanosegundos cuánto tiempo toma una función.
*   **Depuración Avanzada:** Escribir mensajes en la consola de salida que solo se activan cuando estás en modo "Debug".
*   **Detección de Crash:** Obtener el "Stack Trace" (pila de llamadas) cuando ocurre un error para saber exactamente en qué línea y función falló todo.

## Clases y Métodos Principales

*   **`Process`:** La clase más potente para interactuar con el SO.
    *   `.Start("ruta_o_comando")`: Lanza una aplicación o abre un archivo/URL.
    *   `.GetProcesses()`: Lista todos los procesos activos en la máquina.
    *   `.Kill()`: Detiene un proceso forzosamente.
*   **`Stopwatch`:** Un cronómetro de alta precisión (mucho mejor que usar `DateTime`).
    *   `.Start()`, `.Stop()`, `.Restart()`.
    *   `.ElapsedMilliseconds` o `.Elapsed`: Tiempo transcurrido.
*   **`Debug` y `Trace`:**
    *   `Debug.WriteLine()`: Escribe mensajes que solo existen en la versión de desarrollo.
    *   `Trace.WriteLine()`: Similar a Debug, pero puede configurarse para que también funcione en la versión final (producción).
*   **`StackTrace` y `StackFrame`:**
    *   Permiten inspeccionar el camino que siguió el código hasta llegar al punto actual.

## Ejemplos Prácticos

#### Ejemplo 1: Lanzar un proceso externo (Abrir una web)

```csharp
using System.Diagnostics;

// Abrir el navegador en una página específica
// Nota: En .NET Core/5+ se usa UseShellExecute = true
ProcessStartInfo psi = new ProcessStartInfo
{
    FileName = "https://www.google.com",
    UseShellExecute = true 
};
Process.Start(psi);
```

#### Ejemplo 2: Medir el rendimiento de un bloque de código (`Stopwatch`)

```csharp
using System;
using System.Diagnostics;
using System.Threading;

class Program
{
    static void Main()
    {
        Stopwatch cronometro = new Stopwatch();

        cronometro.Start();
        
        // Simulamos un trabajo pesado
        Thread.Sleep(1250); 
        
        cronometro.Stop();

        Console.WriteLine($"Tiempo transcurrido: {cronometro.ElapsedMilliseconds} ms");
        Console.WriteLine($"Tiempo exacto (Ticks): {cronometro.Elapsed.Ticks}");
    }
}
```

#### Ejemplo 3: Listar procesos que consumen mucha memoria

```csharp
using System;
using System.Diagnostics;
using System.Linq;

var procesosPesados = Process.GetProcesses()
    .Where(p => p.WorkingSet64 > 100 * 1024 * 1024) // Más de 100 MB
    .OrderByDescending(p => p.WorkingSet64);

foreach (var p in procesosPesados)
{
    Console.WriteLine($"Proceso: {p.ProcessName} | RAM: {p.WorkingSet64 / 1024 / 1024} MB");
}
```

#### Ejemplo 4: Ejecutar comandos en consola

```csharp
using System;
using System.Diagnostics;

var process = new Process();
process.StartInfo.FileName = "cmd.exe";  // o cualquier ejecutable
process.StartInfo.Arguments = "/c dir";  // argumentos del comando
process.StartInfo.UseShellExecute = false;
process.StartInfo.RedirectStandardOutput = true;
process.Start();

string output = process.StandardOutput.ReadToEnd();
process.WaitForExit();

Console.WriteLine(output);
```

## Buenas Prácticas y Consideraciones

1.  **`Stopwatch` sobre `DateTime`:** Nunca uses `DateTime.Now` para medir rendimiento. `DateTime` depende del reloj del sistema y puede verse afectado por actualizaciones de hora externas. `Stopwatch` usa los contadores de alta resolución del procesador.
2.  **Cuidado al cerrar Procesos:** Si lanzas un proceso con `Process.Start`, intenta guardarlo en una variable y usar `process.Dispose()` o cerrarlo adecuadamente para no dejar "procesos huérfanos".
3.  **Uso de `Conditional("DEBUG")`:** Si creas funciones de ayuda para depuración, puedes decorarlas con el atributo `[Conditional("DEBUG")]`. Esto hará que el compilador borre automáticamente todas las llamadas a esa función cuando compiles tu aplicación para publicarla (modo Release).
4.  **No abuses de `GetProcesses()`:** Obtener la lista de todos los procesos del sistema es una operación costosa para el procesador. No la llames dentro de un bucle que se ejecute muchas veces por segundo.
5.  **Privilegios:** Algunas funciones de esta librería (como matar procesos de otros usuarios) requieren que tu aplicación se ejecute como Administrador/Root.
