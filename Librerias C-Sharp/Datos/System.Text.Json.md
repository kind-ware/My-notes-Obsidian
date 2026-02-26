## Descripción General

`System.Text.Json` es la librería oficial de Microsoft, de alto rendimiento y bajo consumo de memoria, diseñada para serializar y deserializar datos en formato JSON. Introducida en .NET Core 3.0, reemplazó a la famosa (pero más lenta) `Newtonsoft.Json` como el estándar nativo.

Está optimizada para aprovechar las características modernas de C# (como `Span<T>`) y es extremadamente rápida.

#### Casos de Uso Comunes

*   **Deserialización:** Convertir una cadena JSON recibida de una API en un objeto de C# usable.
*   **Serialización:** Convertir un objeto de C# en una cadena JSON para enviarlo a un servidor o guardarlo en un archivo.
*   **Configuración:** Leer archivos de configuración `appsettings.json`.
*   **Manipulación Dinámica:** Acceder a partes de un JSON sin necesidad de crear una clase previa (usando `JsonDocument` o `JsonNode`).

## Clases y Atributos Principales

*  **JsonSerializer (Clase Estática):** El motor principal.
    
	*   `.Serialize<T>(objeto)`: Convierte un objeto a texto JSON.
    *   `.Deserialize<T>(jsonString)`: Convierte texto JSON a un objeto del tipo `<T>`.
	
- **JsonSerializerOptions:** Permite configurar el comportamiento (ej. ignorar mayúsculas/minúsculas, formatear con espacios, etc.).
	
* **Atributos de Propiedad (Data Annotations):**
    
	*   `[JsonPropertyName("nombre_en_json")]`: Mapea una propiedad de C# a un nombre diferente en el JSON.
    *   `[JsonIgnore]`: Evita que una propiedad se incluya en el JSON.
	
* **JsonDocument / JsonElement:** Proporcionan un modelo de solo lectura para navegar por un JSON de forma aleatoria sin deserializar todo.

## Ejemplos Prácticos

#### Ejemplo 1: Serialización y Deserialización Básica

```csharp
using System;
using System.Text.Json;

public class Usuario
{
    public string Nombre { get; set; }
    public int Edad { get; set; }
}

class Program
{
    static void Main()
    {
        var usuario = new Usuario { Nombre = "Alex", Edad = 28 };

        // 1. Convertir objeto a JSON (Serializar)
        string jsonString = JsonSerializer.Serialize(usuario);
        Console.WriteLine($"JSON: {jsonString}"); // Salida: {"Nombre":"Alex","Edad":28}

        // 2. Convertir JSON a objeto (Deserializar)
        Usuario usuarioCopiado = JsonSerializer.Deserialize<Usuario>(jsonString);
        Console.WriteLine($"Nombre: {usuarioCopiado.Nombre}");
    }
}
```

#### Ejemplo 2: Uso de Opciones y Atributos (Caso Real)

A veces el JSON viene en `snake_case` o queremos que el JSON se vea "bonito" (indentado).

```csharp
using System.Text.Json;
using System.Text.Json.Serialization;

public class Producto
{
    [JsonPropertyName("product_id")] // En el JSON se llama "product_id"
    public int Id { get; set; }

    public string Nombre { get; set; }

    [JsonIgnore] // Este campo no se enviará al JSON
    public string NotasInternas { get; set; }
}

// ... dentro del código ...
var opciones = new JsonSerializerOptions 
{ 
    WriteIndented = true, // Hace que el JSON sea legible (con espacios y saltos)
    PropertyNameCaseInsensitive = true // No importa si el JSON viene como "nombre" o "Nombre"
};

string jsonFormateado = JsonSerializer.Serialize(miProducto, opciones);
```

#### Ejemplo 3: Lectura rápida sin clases (`JsonDocument`)

Si solo necesitas un dato específico de un JSON gigante y no quieres crear una clase para todo:

```csharp
string jsonGigante = "{ \"metadatos\": { \"id\": 101 }, \"data\": [1,2,3] }";

using (JsonDocument doc = JsonDocument.Parse(jsonGigante))
{
    int id = doc.RootElement.GetProperty("metadatos").GetProperty("id").GetInt32();
    Console.WriteLine($"El ID extraído es: {id}");
}
```

## Buenas Prácticas y Consideraciones

1.  **Reutiliza `JsonSerializerOptions`:** (Muy importante) No crees un `new JsonSerializerOptions()` cada vez que serialices algo. Guárdalo en una variable estática. Si lo creas cada vez, C# perderá los beneficios del caché interno y tu aplicación será mucho más lenta.
2.  **Case Sensitivity:** A diferencia de la antigua `Newtonsoft.Json`, `System.Text.Json` es **estricto con las mayúsculas y minúsculas** por defecto. Si tu JSON tiene `nombre` y tu clase tiene `Nombre`, la deserialización fallará (dará null) a menos que uses `PropertyNameCaseInsensitive = true`.
3.  **Usa métodos asíncronos para Streams:** Si estás leyendo un JSON directamente de un archivo o de una respuesta HTTP, usa `JsonSerializer.DeserializeAsync(...)`. Es mucho más eficiente para la memoria.
4.  **Propiedades `public` con `{ get; set; }`:** Por defecto, esta librería solo puede ver propiedades públicas que tengan tanto un *getter* como un *setter*. Si olvidas el `set;`, la propiedad no se llenará al deserializar.

