## Descripción General

`System.Threading` proporciona las clases e interfaces que permiten la programación multihilo de bajo nivel. Mientras que las `Tasks` se encargan de *qué* se debe hacer de forma asíncrona, esta librería se encarga del *cómo* se ejecutan los hilos (`Thread`) y de las herramientas de sincronización para evitar condiciones de carrera (*race conditions*).

## 3. Casos de Uso Comunes

*   **Control Total del Hilo:** Crear hilos persistentes que vivan durante toda la ejecución de la app (ej. un hilo de renderizado o de monitoreo).
*   **Sincronización de Recursos:** Bloquear variables o archivos para que solo un hilo pueda modificarlos a la vez.
*   **Operaciones Atómicas:** Realizar operaciones matemáticas simples (sumar, incrementar) de forma segura entre hilos sin usar bloqueos pesados.
*   **Señalización:** Hacer que un hilo espere hasta que otro le avise que puede continuar.

## Clases y Métodos Principales

#### A. Ejecución

*   **`Thread`:** La clase fundamental para crear y controlar un hilo de ejecución.
    *   `.Start()`: Inicia el hilo.
    *   `.Join()`: Bloquea el hilo actual hasta que el hilo en cuestión termine (espera).
    *   `.Sleep(ms)`: Pausa el hilo actual (esta es una pausa **bloqueante**, a diferencia de `Task.Delay`).
*   **`ThreadPool`:** Un grupo de hilos reciclables manejados por el sistema para tareas cortas.

#### B. Sincronización (Los "Semáforos")

*   **`Monitor` / `lock`:** La herramienta más común para asegurar que solo un hilo entre a una sección de código. (En C# usamos la palabra clave `lock` que es un atajo para `Monitor`).
*   **`Interlocked`:** Operaciones atómicas ultra rápidas (incrementar, intercambiar valores) que no requieren bloqueos de memoria complejos.
*   **`Mutex`:** Un bloqueo que funciona incluso **entre diferentes aplicaciones/procesos** del sistema operativo.
*   **`SemaphoreSlim`:** Una versión ligera de un semáforo que permite que un número X de hilos entren a una sección (ej. "solo permitir 3 descargas simultáneas").

## Ejemplos Prácticos

#### Ejemplo 1: Creación de un hilo manual

```csharp
using System;
using System.Threading;

class Program
{
    static void Main()
    {
        // Creamos el hilo y le decimos qué función ejecutar
        Thread nuevoHilo = new Thread(TrabajoPesado);
        
        nuevoHilo.Start(); // Inicia el hilo secundario
        
        Console.WriteLine("El hilo principal sigue trabajando...");
        
        nuevoHilo.Join(); // Esperamos a que el hilo secundario termine
        Console.WriteLine("Hilo secundario finalizado.");
    }

    static void TrabajoPesado()
    {
        Thread.Sleep(2000); // Pausa de 2 segundos
        Console.WriteLine("Trabajo desde el hilo secundario completado.");
    }
}
```

**Ejemplo 2: Evitando el caos con `lock` (Sincronización)**

Si dos hilos intentan sumar a la misma variable al mismo tiempo sin protección, el resultado será incorrecto.

```csharp
using System;
using System.Threading;

class Contador
{
    public int Valor = 0;
    private readonly object _bloqueo = new object(); // Objeto para el candado

    public void Incrementar()
    {
        lock (_bloqueo) // Solo un hilo puede entrar aquí a la vez
        {
            Valor++;
        }
    }
}
```

**Ejemplo 3: Operaciones Atómicas con `Interlocked`**
Mucho más rápido que un `lock` para casos simples.

```csharp
int contador = 0;

// Incrementa la variable de forma segura sin usar 'lock'
Interlocked.Increment(ref contador); 

// Intercambia el valor de forma segura
Interlocked.Exchange(ref contador, 100);
```

## 6. Buenas Prácticas y Consideraciones

1.  **Prefiere `Tasks` sobre `Thread`:** Crear un `Thread` manualmente es costoso en términos de memoria y CPU. Para la mayoría de las tareas, usa `Task.Run` (TPL), que utiliza el `ThreadPool` de forma eficiente. Usa `Thread` solo si necesitas control absoluto.
2.  **Cuidado con los Deadlocks (Interbloqueos):** Ocurren cuando el Hilo A espera al Hilo B, y el Hilo B espera al Hilo A. Ninguno avanza y el programa se congela. Siempre bloquea los objetos en el mismo orden.
3.  **No bloquees el objeto `this` ni tipos `string`:** Para el `lock`, siempre crea un objeto privado dedicado (como `private readonly object _lock = new object();`). Bloquear `this` o un `string` puede causar conflictos con otras partes del código que no controlas.
4.  **Usa `CancellationToken`:** En lugar de intentar "matar" un hilo (lo cual es peligroso y puede dejar archivos corruptos), usa un `CancellationToken` para pedirle al hilo que termine de forma educada y segura.
5.  **Volatile:** Si tienes una variable que es leída y escrita por diferentes hilos, marcarla como `volatile` asegura que el valor leído sea siempre el más reciente de la memoria RAM y no una copia guardada en el caché del procesador.

