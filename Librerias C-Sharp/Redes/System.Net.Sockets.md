## Descripción General

Este espacio de nombres proporciona una implementación administrada de la interfaz de sockets de Windows (Winsock). Permite controlar la comunicación a nivel de transporte (Capa 4 del modelo OSI), permitiéndote trabajar directamente con **TCP** (orientado a conexión) y **UDP** (sin conexión/datagramas).

#### Casos de Uso Comunes

*   **Protocolos Propios:** Crear tu propio protocolo de comunicación ligero y ultra veloz.
*   **Servidores de Chat o Juegos:** Donde necesitas una conexión persistente y bidireccional en tiempo real.
*   **Comunicación IoT:** Hablar con dispositivos embebidos que no soportan HTTP pero sí sockets TCP/UDP.
*   **Proxy y VPNs:** Desarrollar herramientas que intercepten o redirijan tráfico de red.

## Clases y Métodos Principales

Existen dos niveles de abstracción dentro de esta librería:

#### A. Clases de "Alto Nivel" (Recomendadas para la mayoria de casos):

*   **`TcpListener`:** Escucha conexiones entrantes de clientes TCP (es el "servidor").
*   **`TcpClient`:** Se conecta a un servidor y permite enviar/recibir datos mediante un flujo.
*   **`UdpClient`:** Envía y recibe paquetes (datagramas) individuales de forma rápida pero sin garantía de llegada.

### B. Clase de "Bajo Nivel":

*   **`Socket`:** La clase base que usan todas las anteriores. Te da control total (timeout, tamaño de búfer, KeepAlive, etc.), pero es mucho más compleja de usar.

#### Métodos Clave (Asíncronos):

*   `ConnectAsync()`: Establece una conexión con un host.
*   `AcceptTcpClientAsync()`: Espera y acepta una nueva conexión entrante.
*   `GetStream()`: (En `TcpClient`) Devuelve un `NetworkStream` para leer y escribir.
*   `SendAsync()` / `ReceiveAsync()`: Envío y recepción de bytes en crudo.

## Ejemplos Prácticos

#### Ejemplo 1: Servidor TCP Simple (Eco)

Este servidor espera a un cliente, recibe un mensaje y lo imprime.

```csharp
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading.Tasks;

class ServidorTCP
{
    static async Task Main()
    {
        TcpListener listener = new TcpListener(IPAddress.Any, 8080);
        listener.Start();
        Console.WriteLine("Servidor iniciado en el puerto 8080...");

        // Aceptamos al cliente
        using TcpClient cliente = await listener.AcceptTcpClientAsync();
        Console.WriteLine("¡Cliente conectado!");

        using NetworkStream stream = cliente.GetStream();
        byte[] buffer = new byte[1024];

        // Leemos los bytes enviados por el cliente
        int bytesLeidos = await stream.ReadAsync(buffer, 0, buffer.Length);
        string mensaje = Encoding.UTF8.GetString(buffer, 0, bytesLeidos);

        Console.WriteLine($"El cliente dice: {mensaje}");
        listener.Stop();
    }
}
```

#### Ejemplo 2: Cliente TCP Simple

```csharp
using System.Net.Sockets;
using System.Text;

using TcpClient cliente = new TcpClient();
await cliente.ConnectAsync("127.0.0.1", 8080);

using NetworkStream stream = cliente.GetStream();
byte[] datos = Encoding.UTF8.GetBytes("Hola desde el cliente!");

await stream.WriteAsync(datos, 0, datos.Length);
Console.WriteLine("Mensaje enviado.");
```

## Buenas Prácticas y Consideraciones

1.  **Endianness (Orden de bytes):** Al enviar números (integers) entre diferentes sistemas (ej. C# a un Arduino), recuerda que el orden de los bytes puede cambiar (*Little Endian* vs *Big Endian*). Usa `IPAddress.HostToNetworkOrder` para estandarizar.
2.  **No asumas mensajes completos:** TCP es un "stream", no un sistema de mensajes. Si envías "Hola" y "Mundo", el receptor podría recibir "HolaMundo" de un solo golpe o recibir "H", luego "ola", etc. Debes implementar un separador (como `\n`) o enviar primero el tamaño del mensaje.
3.  **Uso de `NetworkStream`:** Siempre envuelve el stream en un bloque `using` para que los sockets se cierren correctamente y no queden puertos "zombies" en el sistema.
4.  **Async SIEMPRE:** Las operaciones de sockets son "bloqueantes". Si usas `Connect()` en lugar de `ConnectAsync()`, tu aplicación se detendrá hasta que el servidor responda o el tiempo agote.
5.  **Timeout y KeepAlive:** En conexiones largas (como un chat), la red puede cortarse sin que el socket se entere inmediatamente. Configura `TcpClient.Client.KeepAlive = true` para detectar desconexiones fantasmas.

