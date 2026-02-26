## Descripción General

Este espacio de nombres es la base de la **TPL (Task Parallel Library)**. Proporciona tipos que permiten escribir código asíncrono y paralelo de forma sencilla. El concepto central es la **`Task`** (Tarea), que representa una operación que se está ejecutando y que eventualmente terminará, devolviendo o no un resultado.

A diferencia de los "Threads" (hilos) manuales, las `Tasks` son gestionadas por un planificador interno que optimiza el uso de la CPU.

#### Casos de Uso Comunes

*   **Operaciones de E/S (I/O):** Realizar peticiones web, leer archivos o consultar bases de datos sin bloquear la interfaz de usuario o el hilo principal.
*   **Paralelismo de datos:** Realizar cálculos intensivos en múltiples núcleos de la CPU simultáneamente.
*   **Operaciones con retardo:** Ejecutar código después de un tiempo determinado sin detener el programa.
*   **Tareas en segundo plano:** Ejecutar procesos largos mientras el usuario sigue interactuando con la aplicación.

## Clases y Métodos Principales

*   **`Task`:** Representa una operación que no devuelve valor (equivalente a un `void` asíncrono).
*   **`Task<TResult>`:** Representa una operación que devuelve un valor de tipo `TResult`.
*   **`Task.Run()`:** Envía un trabajo para que se ejecute en un hilo distinto (en el *Thread Pool*).
*   **`Task.WhenAll(tasks)`:** Crea una tarea que se completa cuando **todas** las tareas proporcionadas han terminado.
*   **`Task.WhenAny(tasks)`:** Se completa cuando **al menos una** de las tareas termina.
*   **`Task.Delay(ms)`:** Pausa asíncrona (equivalente a `await asyncio.sleep()` en Python). No bloquea el hilo, solo espera.

## Ejemplos Prácticos

#### Ejemplo 1: El patrón `async` / `await` básico

```C#
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main() // El método principal también puede ser asíncrono
    {
        Console.WriteLine("Iniciando descarga...");
        
        // Esperamos a que la tarea termine sin bloquear el programa
        string resultado = await DescargarDatosSimuladoAsync();
        
        Console.WriteLine($"Resultado: {resultado}");
    }

    static async Task<string> DescargarDatosSimuladoAsync()
    {
        // Simulamos una espera de red de 2 segundos
        await Task.Delay(2000); 
        return "Datos de la web";
    }
}
```

#### Ejemplo 2: Ejecutar varias tareas en paralelo (`WhenAll`)

```csharp
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

// ... dentro de un método async ...

var tareas = new List<Task<int>>();

for (int i = 1; i <= 3; i++)
{
    int id = i; // Capturamos la variable para el closure
    tareas.Add(Task.Run(async () => {
        await Task.Delay(1000); // Simulamos trabajo
        return id * 10;
    }));
}

// Esperamos a que las 3 tareas terminen al mismo tiempo
int[] resultados = await Task.WhenAll(tareas);

// Salida: 10, 20, 30 (obtenidos en ~1 segundo, no en 3)
```

#### Ejemplo 3: Tarea con cancelación (`CancellationToken`)

C# tiene un sistema estándar para "arrepentirse" de una tarea en curso.

```csharp
using System.Threading;

CancellationTokenSource cts = new CancellationTokenSource();

// Si pasan 5 segundos y no ha terminado, cancelamos
cts.CancelAfter(5000);

try {
    await OperacionLargaAsync(cts.Token);
} catch (OperationCanceledException) {
    Console.WriteLine("La tarea fue cancelada por exceso de tiempo.");
}
```

## Buenas Prácticas y Consideraciones

1.  **"Async all the way" (Asíncrono hasta el final):** No mezcles código síncrono y asíncrono. Si un método llama a algo `async`, ese método también debería ser `async` y ser esperado con `await`.
2.  **Evita `.Result` y `.Wait()`:** (Muy importante) Usar estos métodos bloquea el hilo actual hasta que la tarea termine. En aplicaciones de escritorio o web, esto suele causar **Deadlocks** (el programa se congela por completo). Usa siempre `await`.
3.  **Sufijo "Async":** Por convención en .NET, todos los métodos que devuelven una `Task` deben terminar con el nombre `Async` (ej. `LeerArchivoAsync`).
4.  **Usa `Task.Run` solo para CPU:** No uses `Task.Run` para operaciones de entrada/salida (como bases de datos). Usa las versiones asíncronas nativas del framework (como `ExecuteReaderAsync`). Reservamos `Task.Run` para cálculos matemáticos o procesamiento de datos pesado.
5.  **ValueTask:** Para métodos que se llaman millones de veces y que a veces ya tienen el resultado listo (sin esperar), existe `ValueTask<T>`, que es más ligero para la memoria RAM que una `Task` normal.
