## Descripción General

`System.Collections.Generic` contiene interfaces y clases que definen **colecciones fuertemente tipadas** (genéricas). ¿Qué significa esto? Que cuando creas una lista o un diccionario, debes especificar qué **tipo de dato** (`<T>`) va a contener (por ejemplo, una lista solo de enteros, o un diccionario de *string* a *booleano*). Esto mejora drásticamente el rendimiento y evita errores de tipo en tiempo de ejecución (Type Safety).

#### Casos de Uso Comunes

*   **Almacenamiento dinámico de datos:** Cuando no sabes cuántos elementos vas a guardar (a diferencia de los Arreglos/Arrays tradicionales `[]` que tienen un tamaño fijo).
*   **Búsquedas ultra rápidas:** Almacenar datos en pares de *Clave-Valor* para acceder a ellos instantáneamente sin tener que recorrer toda la lista.
*   **Garantizar elementos únicos:** Filtrar datos eliminando duplicados automáticamente.
*   **Estructuras LIFO/FIFO:** Manejar turnos, historiales de navegación, o procesamiento de tareas (Pilas y Colas).

## Clases y Métodos Principales

*   **`List<T>`** (Equivalente a `list` en Python): Una colección ordenada cuyo tamaño aumenta dinámicamente.
	
    *   `.Add(item)` / `.AddRange(collection)`: Agrega uno o varios elementos.
    *   `.Remove(item)` / `.RemoveAt(index)`: Elimina elementos.
    *   `.Contains(item)`: Verifica si un elemento existe (devuelve *booleano*).
    *   `.Count`: Propiedad que devuelve la cantidad de elementos (como el `len()` de Python).
    
*   **`Dictionary<TKey, TValue>`** (Equivalente a `dict` en Python): Colección de pares clave-valor (basada en tablas Hash).
    
	*   `.Add(key, value)`: Agrega un nuevo par.
    *   `.ContainsKey(key)`: Verifica si una clave existe.
    *   `.TryGetValue(key, out value)`: Intenta obtener un valor de forma segura sin que el programa se caiga si la clave no existe.
    
*   **`HashSet<T>`** (Equivalente a `set` en Python): Colección que **no permite elementos duplicados** y está optimizada para búsquedas matemáticas.
    
	*   `.Add(item)`: Agrega el elemento (si ya existe, simplemente lo ignora y devuelve `false`).
	
*   **`Queue<T>`** (Cola - FIFO: *First In, First Out*): El primero en llegar es el primero en salir.
    
	*   `.Enqueue(item)`: Agrega al final de la cola.
    *   `.Dequeue()`: Saca y devuelve el primer elemento.
    
*   **`Stack<T>`** (Pila - LIFO: *Last In, First Out*): El último en entrar es el primero en salir (como el botón "Atrás" del navegador).
	
    *   `.Push(item)`: Agrega a la cima de la pila.
    *   `.Pop()`: Saca y devuelve el elemento de la cima.
	
## Ejemplos Prácticos

#### Ejemplo 1: Uso básico de Listas (`List<T>`)

```csharp
using System;
using System.Collections.Generic; // Obligatorio para usar estas clases

class Program
{
    static void Main()
    {
        // Creamos una lista que SOLO acepta strings
        List<string> frutas = new List<string>();
        
        frutas.Add("Manzana");
        frutas.Add("Banana");
        frutas.Add("Naranja");

        // Podemos acceder por índice
        Console.WriteLine($"La primera fruta es: {frutas[0]}");

        // Iterar sobre la lista
        foreach (string fruta in frutas)
        {
            Console.WriteLine(fruta);
        }
    }
}
```

#### Ejemplo 2: Diccionarios y recuperación segura (Dictionary<TKey, TValue>)

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        // Diccionario: Clave de tipo string, Valor de tipo int
        Dictionary<string, int> edades = new Dictionary<string, int>
        {
            { "Juan", 25 },
            { "Maria", 30 }
        };

        edades.Add("Pedro", 28); // Agregar un elemento nuevo

        // Forma segura de buscar (Equivalente a edades.get("Maria") en Python)
        if (edades.TryGetValue("Maria", out int edadMaria))
        {
            Console.WriteLine($"Maria tiene {edadMaria} años.");
        }
        else
        {
            Console.WriteLine("Maria no está en el diccionario.");
        }
    }
}
```

#### Ejemplo 3: Eliminar duplicados con `HashSet<T>'

```csharp
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
        List<int> numerosConDuplicados = new List<int> { 1, 2, 2, 3, 4, 4, 5 };
        
        // Al pasar la lista al HashSet, los duplicados desaparecen automáticamente
        HashSet<int> numerosUnicos = new HashSet<int>(numerosConDuplicados);

        Console.WriteLine(string.Join(", ", numerosUnicos)); 
        // Salida: 1, 2, 3, 4, 5
    }
}
```

## Buenas Prácticas y Consideraciones

1.  **Dile ADIÓS a las colecciones NO genéricas:** Existe un namespace antiguo llamado `System.Collections` (sin el `.Generic`) que tiene clases como `ArrayList` o `Hashtable`. **No las uses nunca en código moderno**. Guardan todo como tipo `object`, lo que causa lentitud (por un proceso llamado *boxing/unboxing*) y posibles caídas del sistema. Usa siempre las versiones genéricas (`List<T>`, etc.).

2.  **Define la capacidad inicial si la conoces:** Si sabes que vas a guardar exactamente 1000 elementos en una lista, inicialízala así: `new List<int>(1000)`. Esto evita que la lista tenga que redimensionar su memoria internamente varias veces, mejorando mucho el rendimiento.

3.  **Usa `TryGetValue` en lugar de indexación directa:** Si haces `int edad = edades["Carlos"]` y "Carlos" no existe, C# lanzará una excepción (`KeyNotFoundException`) que detendrá tu programa. Es mejor práctica usar `TryGetValue()` o comprobar primero con `ContainsKey()`.

4.  **Usa `Count` en lugar de `.Length`**: En C#, los arreglos fijos `(como int[])` usan la propiedad .Length, mientras que todas las colecciones genéricas de este namespace `(como List<T> o Dictionary<TKey, TValue>)` usan la propiedad **.Count**. Es el error más común al saltar de un tipo de colección a otro.