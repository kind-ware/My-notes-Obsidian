## Descripción General

Este espacio de nombres proporciona acceso a datos estadísticos sobre el tráfico de red, información sobre las interfaces de red locales (tarjetas Wi-Fi, Ethernet, etc.) y la capacidad de realizar diagnósticos básicos como el envío de paquetes de eco (Ping). Es la herramienta ideal para saber si una computadora está conectada, cuántos datos ha enviado o qué dirección física (MAC) tiene.

#### Casos de Uso Comunes

*   **Detección de conectividad:** Verificar si el equipo tiene acceso a internet o a una red local antes de intentar una descarga.
*   **Inventario de Hardware:** Obtener las direcciones IP, máscaras de subred y direcciones MAC de todas las interfaces del sistema.
*   **Diagnóstico de Latencia:** Implementar un sistema de "latencia" o "ping" dentro de un juego o aplicación para medir la respuesta del servidor.
*   **Monitoreo de tráfico:** Medir cuántos bytes se han enviado o recibido por una interfaz específica (útil para widgets de rendimiento).
*   **Eventos de red:** Reaccionar automáticamente cuando el usuario se desconecta del Wi-Fi o cambia de red.

## Clases y Métodos Principales

*   **`NetworkInterface`:** La clase más importante para obtener datos de hardware.
    *   `.GetAllNetworkInterfaces()`: Devuelve un arreglo con todas las tarjetas de red detectadas.
    *   `.GetIsNetworkAvailable()`: Método estático rápido para saber si hay alguna conexión activa.
*   **`Ping`:** Permite enviar una solicitud de eco ICMP.
    *   `.Send(address)` / `.SendPingAsync(address)`: Envía el ping y devuelve un objeto `PingReply`.
*   **`IPGlobalProperties`:** Proporciona información sobre la configuración de red del equipo local.
    *   `.GetIPGlobalProperties()`: Permite ver conexiones TCP/UDP activas y nombres de dominio.
*   **`NetworkChange`:** Proporciona eventos para detectar cambios.
    *   `.NetworkAddressChanged`: Evento que se dispara si la IP cambia o la red se cae.

## Ejemplos Prácticos

#### Ejemplo 1: Listar interfaces de red y sus direcciones MAC

```csharp
using System;
using System.Net.NetworkInformation;

class Program
{
    static void Main()
    {
        Console.WriteLine("Interfaces de red detectadas:");
        
        foreach (NetworkInterface nic in NetworkInterface.GetAllNetworkInterfaces())
        {
            // Filtrar solo las que están activas
            if (nic.OperationalStatus == OperationalStatus.Up)
            {
                Console.WriteLine($"Nombre: {nic.Name}");
                Console.WriteLine($"Tipo: {nic.NetworkInterfaceType}");
                Console.WriteLine($"Velocidad: {nic.Speed / 1_000_000} Mbps");
                Console.WriteLine($"Dirección MAC: {nic.GetPhysicalAddress()}");
                Console.WriteLine("--------------------------------------");
            }
        }
    }
}
```

#### Ejemplo 2: Realizar un Ping asíncrono

```csharp
using System;
using System.Net.NetworkInformation;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Ping pingSender = new Ping();
        string host = "8.8.8.8"; // Google DNS
        
        try
        {
            PingReply respuesta = await pingSender.SendPingAsync(host);

            if (respuesta.Status == IPStatus.Success)
            {
                Console.WriteLine($"Respuesta desde {host}:");
                Console.WriteLine($"Tiempo de respuesta: {respuesta.RoundtripTime} ms");
                Console.WriteLine($"Estado: {respuesta.Status}");
            }
            else
            {
                Console.WriteLine($"Fallo al contactar: {respuesta.Status}");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error al realizar ping: {ex.Message}");
        }
    }
}
```

#### Ejemplo 3: Detectar si hay conexión a Internet (Rápido)

```csharp
if (NetworkInterface.GetIsNetworkAvailable())
{
    Console.WriteLine("El equipo está conectado a una red.");
}
else
{
    Console.WriteLine("No se detectó ninguna conexión activa.");
}
```

## Buenas Prácticas y Consideraciones

1.  **`Ping` implementa `IDisposable`:** Si creas muchas instancias de la clase `Ping`, asegúrate de usar un bloque `using` o reutilizar la misma instancia para evitar fugas de recursos.
2.  **No confíes solo en `GetIsNetworkAvailable`:** Este método devuelve `true` si el equipo está conectado a una red (como un router), pero **no garantiza** que haya internet real hacia el exterior. Para verificar internet real, es mejor intentar un `Ping` a un servidor DNS confiable o una petición HTTP ligera.
3.  **Manejo de Excepciones en Ping:** El método `Send` o `SendAsync` puede lanzar excepciones si no hay conexión física, si el nombre del host no se resuelve o si el firewall bloquea el tráfico ICMP. Siempre envuélvelo en un `try-catch`.
4.  **Cuidado con las interfaces virtuales:** Si tienes instalado Docker, VMware o VirtualBox, `GetAllNetworkInterfaces()` devolverá muchas interfaces "ficticias". Puedes filtrarlas revisando si `nic.Description` o `nic.Name` contienen palabras como "Virtual", "Loopback" o "vEthernet".
5.  **Permisos:** En algunos entornos muy restrictivos (como aplicaciones móviles o sandboxed), obtener la dirección MAC o realizar pings puede requerir permisos especiales en el manifiesto de la aplicación.

