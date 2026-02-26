## Descripción General

`System.Net.Http` proporciona una interfaz de programación moderna y flexible para aplicaciones que necesitan conectarse a servicios a través de HTTP. Su clase estrella es **`HttpClient`**, que permite enviar solicitudes y recibir respuestas de recursos web (APIs, sitios web, archivos) de forma totalmente asíncrona.

Es el sucesor de las antiguas clases `WebClient` y `HttpWebRequest` (las cuales ya no se recomiendan en .NET moderno).

## Casos de Uso Comunes

*   **Consumo de APIs REST:** Enviar y recibir datos (usualmente en formato JSON) desde un servidor.
*   **Descarga de contenido:** Bajar imágenes, archivos o el código HTML de una página.
*   **Envío de formularios:** Enviar datos de registro o inicio de sesión a un servidor.
*   **Automatización Web:** Interactuar con servicios de terceros (como la API de Telegram, Google Maps, etc.).

## Clases y Métodos Principales

*   **`HttpClient`:** El objeto principal que actúa como un "navegador" interno para tu código.
    *   `GetAsync(url)`: Envía una petición GET para obtener datos.
    *   `PostAsync(url, content)`: Envía una petición POST para crear/enviar datos.
    *   `PutAsync(url, content)`: Actualiza un recurso existente.
    *   `DeleteAsync(url)`: Elimina un recurso.
    *   `SendAsync(request)`: Permite un control total sobre la configuración de la petición.
*   **`HttpResponseMessage`:** Representa la respuesta que el servidor nos devuelve.
    *   `.IsSuccessStatusCode`: Propiedad booleana (true si el código es 200-299).
    *   `.StatusCode`: El código de estado (404 Not Found, 500 Error, etc.).
    *   `.Content`: El cuerpo de la respuesta.
*   **`HttpContent`:** Clase base para el contenido que envías (cuerpo del mensaje).
    *   `StringContent`: Para enviar texto plano o JSON.
    *   `MultipartFormDataContent`: Para subir archivos.

## Ejemplos Prácticos

#### Ejemplo 1: Petición GET básica (Obtener datos de una API)

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    // HttpClient debe ser estático o de larga duración (ver Buenas Prácticas)
    private static readonly HttpClient client = new HttpClient();

    static async Task Main()
    {
        try
        {
            // Realizamos la petición de forma asíncrona
            string url = "https://jsonplaceholder.typicode.com/posts/1";
            HttpResponseMessage respuesta = await client.GetAsync(url);

            // Verificamos si la petición fue exitosa
            respuesta.EnsureSuccessStatusCode();

            // Leemos el contenido como string
            string contenido = await respuesta.Content.ReadAsStringAsync();
            
            Console.WriteLine("Respuesta recibida:");
            Console.WriteLine(contenido);
        }
        catch (HttpRequestException e)
        {
            Console.WriteLine($"Error de red: {e.Message}");
        }
    }
}
```

#### Ejemplo 2: Petición POST (Enviar JSON a un servidor)

```csharp
using System.Text;
using System.Net.Http;

// ... dentro de un método async ...
string jsonParaEnviar = "{\"titulo\": \"Post de C#\", \"autor\": \"Yo\"}";
var contenido = new StringContent(jsonParaEnviar, Encoding.UTF8, "application/json");

HttpResponseMessage respuesta = await client.PostAsync("https://api.ejemplo.com/posts", contenido);

if (respuesta.IsSuccessStatusCode)
{
    Console.WriteLine("¡Post creado con éxito!");
}
```

## Buenas Prácticas y Consideraciones

1.  **NO uses `using` con `HttpClient`:** (Este es el consejo más importante). Aunque `HttpClient` implementa `IDisposable`, cerrarlo y abrirlo constantemente para cada petición provoca un error llamado **"Socket Exhaustion"** (agotamiento de puertos). 
    
	*   **Lo correcto:** Mantén una única instancia de `HttpClient` estática para toda tu aplicación o usa `IHttpClientFactory` en aplicaciones ASP.NET Core.
	
2.  **Usa siempre métodos `Async`:** Las peticiones web pueden tardar milisegundos o segundos. Si no usas `await`, tu programa se "congelará" mientras espera la respuesta.
3.  **Configura el Timeout:** Por defecto, `HttpClient` espera 100 segundos antes de rendirse. Es buena práctica bajarlo (ej. `client.Timeout = TimeSpan.FromSeconds(10);`) para que tu aplicación no se quede colgada infinitamente si el servidor no responde.
4.  **Validación de errores:** Siempre usa `EnsureSuccessStatusCode()` o revisa `IsSuccessStatusCode`. Un servidor puede responderte un error 500 y `HttpClient` no lanzará una excepción automáticamente, simplemente te dará la respuesta con el código de error.
5.  **User-Agent:** Algunos servidores bloquean peticiones que no tienen un encabezado `User-Agent`. Puedes añadir uno globalmente: 
    `client.DefaultRequestHeaders.Add("User-Agent", "MiAppCsharp");`