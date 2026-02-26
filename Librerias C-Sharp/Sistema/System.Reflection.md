## Descripción General

`System.Reflection` permite obtener información sobre los ensamblados (`.dll` o `.exe`), módulos y tipos (clases, interfaces, estructuras) en tiempo de ejecución. Además, permite crear instancias de tipos, vincular tipos a objetos existentes y obtener o invocar métodos, propiedades, campos y eventos de forma dinámica.

Es, básicamente, la capacidad de un programa de **inspeccionarse a sí mismo**.

#### Casos de Uso Comunes

*   **Sistemas de Plugins:** Cargar una `.dll` externa de la que no sabes nada y ejecutar sus métodos.
*   **Serializadores y ORMs:** Como vimos en `System.Text.Json` o Entity Framework; estas librerías leen las propiedades de tus clases usando reflexión para saber qué guardar en un JSON o una base de datos.
*   **Inyección de Dependencias:** Instanciar clases automáticamente analizando sus constructores.
*   **Herramientas de Test Unitario:** Encontrar todos los métodos que tienen un atributo `[Test]` y ejecutarlos.
*   **Inspección de Atributos:** Leer metadatos personalizados que hayas puesto sobre clases o métodos.

## Clases y Métodos Principales

*   **`Type`:** La clase central. Representa la declaración de un tipo (clase, interfaz, etc.).
    *   `typeof(MiClase)` u `objeto.GetType()`: Obtienen el objeto `Type`.
    *   `.GetProperties()`, `.GetMethods()`, `.GetConstructors()`: Devuelven arreglos con la estructura del tipo.
*   **`Assembly`:** Representa un archivo compilado de .NET.
    *   `Assembly.LoadFrom("ruta.dll")`: Carga un archivo externo.
    *   `.GetTypes()`: Obtiene todas las clases definidas en ese archivo.
*   **`PropertyInfo` / `MethodInfo` / `FieldInfo`:** Representan un miembro específico de una clase.
    *   `methodInfo.Invoke(objeto, parametros)`: Ejecuta un método dinámicamente.
    *   `propertyInfo.GetValue(objeto)` / `SetValue(...)`: Lee o escribe en una propiedad.
*   **`Activator`:** Clase de utilidad para crear objetos.
    *   `Activator.CreateInstance(tipo)`: Crea una instancia de una clase a partir de su objeto `Type`.

## Ejemplos Prácticos

#### Ejemplo 1: Inspeccionar una clase en tiempo de ejecución**

```csharp
using System;
using System.Reflection;

public class Persona
{
    public string Nombre { get; set; }
    public void Saludar() => Console.WriteLine("¡Hola!");
}

class Program
{
    static void Main()
    {
        Type t = typeof(Persona);

        Console.WriteLine($"Clase: {t.Name}");

        Console.WriteLine("Propiedades:");
        foreach (var prop in t.GetProperties())
            Console.WriteLine($"- {prop.Name} (Tipo: {prop.PropertyType.Name})");

        Console.WriteLine("Métodos:");
        foreach (var metodo in t.GetMethods(BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly))
            Console.WriteLine($"- {metodo.Name}");
    }
}
```

#### Ejemplo 2: Invocar un método por su nombre (Dinámico)

Imagina que recibes un string con el nombre de un método y quieres ejecutarlo.

```csharp
Persona alguien = new Persona();
Type t = alguien.GetType();

// Buscamos el método por nombre
MethodInfo metodoSaludar = t.GetMethod("Saludar");

// Lo ejecutamos sobre la instancia 'alguien'
metodoSaludar.Invoke(alguien, null);
```

#### Ejemplo 3: Leer Atributos Personalizados

Esta es la base de frameworks como ASP.NET o herramientas de validación.

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class InfoAttribute : Attribute { public string Descripcion { get; set; } }

[Info(Descripcion = "Esta es una clase de prueba")]
public class MiClase { }

// Para leerlo:
var atributo = typeof(MiClase).GetCustomAttribute<InfoAttribute>();
Console.WriteLine(atributo?.Descripcion);
```

## Buenas Prácticas y Consideraciones

1.  **Rendimiento (Performance):** La reflexión es **lenta** comparada con el acceso directo. Si llamas a un método vía reflexión millones de veces en un bucle, notarás una caída de rendimiento. 
    *   *Tip:* Si vas a usar reflexión repetidamente, intenta "cachear" (guardar) los objetos `MethodInfo` o `PropertyInfo`.
2.  **Seguridad:** La reflexión puede acceder a miembros privados (`BindingFlags.NonPublic`). Ten cuidado, ya que esto rompe el principio de encapsulamiento y puede hacer que tu código falle si la librería interna cambia.
3.  **Manejo de Errores:** Al trabajar con nombres de strings (como `GetMethod("Saludar")`), el compilador no te avisará si escribes mal el nombre. Prepárate para manejar valores `null` o excepciones `TargetInvocationException`.
4.  **Uso en AOT (Ahead-Of-Time):** En plataformas como iOS (Unity) o aplicaciones .NET compiladas con NativeAOT, la reflexión está limitada porque el compilador elimina el código que cree que "no se usa".

