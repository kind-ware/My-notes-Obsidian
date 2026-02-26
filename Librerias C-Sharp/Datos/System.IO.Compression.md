## Descripción General

Este espacio de nombres proporciona clases para comprimir y descomprimir flujos de datos (*streams*) y archivos. Soporta los algoritmos más comunes de la industria: **Zip**, **GZip**, **Deflate** y el moderno **Brotli** (que ofrece una compresión superior para la web).

#### Casos de Uso Comunes

*   **Archivado de logs:** Comprimir archivos de texto viejos para ahorrar espacio en disco.
*   **Creación de instaladores o backups:** Empaquetar carpetas enteras en un solo archivo `.zip`.
*   **Transferencia de datos en red:** Comprimir un JSON gigante antes de enviarlo por un Socket para ahorrar ancho de banda.
*   **Lectura de formatos Office/Java:** Los archivos `.docx`, `.xlsx` o `.jar` son en realidad archivos ZIP; con esta librería puedes "abrirlos" y leer su contenido interno.

## Clases y Métodos Principales

La librería se divide en dos niveles: **operaciones rápidas** (archivos completos) y **operaciones de flujo** (compresión en memoria).

*   **`ZipFile` (Clase Estática):** La forma más sencilla de trabajar con archivos físicos.
    *   `.CreateFromDirectory(source, destination)`: Comprime una carpeta entera.
    *   `.ExtractToDirectory(source, destination)`: Descomprime un ZIP en una carpeta.
    *   `.Open(path, mode)`: Abre un ZIP para añadir o quitar archivos individualmente.
*   **`ZipArchive`:** Para manipular el contenido de un ZIP sin extraerlo todo (leer archivos específicos dentro del paquete).
    *   `.CreateEntry(name)`: Crea un nuevo archivo dentro del ZIP vacío.
    *   `.GetEntry(name)`: Busca un archivo específico dentro del ZIP.
*   **`GZipStream` / `DeflateStream` / `BrotliStream`:** Clases para comprimir datos "al vuelo" mientras se escriben en un flujo (stream).

## Ejemplos Prácticos

> **Nota:** Para usar `ZipFile` en algunos proyectos de .NET antiguos, podrías necesitar agregar la referencia a `System.IO.Compression.FileSystem`.

#### Ejemplo 1: Comprimir y Descomprimir carpetas (Lo más común)

```csharp
using System;
using System.IO.Compression;

class Program
{
    static void Main()
    {
        string carpetaLocal = @"C:\MisDocumentos\Fotos";
        string archivoZip = @"C:\Backups\Fotos_2023.zip";
        string destinoExtraccion = @"C:\Recuperacion\Fotos";

        // Crear un ZIP desde una carpeta
        ZipFile.CreateFromDirectory(carpetaLocal, archivoZip);
        Console.WriteLine("¡Carpeta comprimida con éxito!");

        // Extraer el contenido
        ZipFile.ExtractToDirectory(archivoZip, destinoExtraccion);
        Console.WriteLine("Archivo extraído.");
    }
}
```

#### Ejemplo 2: Leer un archivo específico dentro de un ZIP (Sin extraer todo)

```csharp
using (ZipArchive zip = ZipFile.OpenRead("datos.zip"))
{
    foreach (ZipArchiveEntry entrada in zip.Entries)
    {
        Console.WriteLine($"Archivo: {entrada.FullName}, Tamaño: {entrada.Length} bytes");
        
        if (entrada.Name == "config.txt")
        {
            using (StreamReader lector = new StreamReader(entrada.Open()))
            {
                Console.WriteLine("Contenido de config.txt:");
                Console.WriteLine(lector.ReadToEnd());
            }
        }
    }
}
```

#### Ejemplo 3: Compresión en memoria con GZip (Ideal para Redes)

```csharp
public static byte[] ComprimirTexto(string texto)
{
    byte[] datos = System.Text.Encoding.UTF8.GetBytes(texto);
    using (var ms = new MemoryStream())
    {
        using (var gzs = new GZipStream(ms, CompressionMode.Compress))
        {
            gzs.Write(datos, 0, datos.Length);
        }
        return ms.ToArray(); // Devuelve los bytes comprimidos
    }
}
```

## Buenas Prácticas y Consideraciones

1.  **Niveles de Compresión:** Al crear un archivo, puedes elegir entre `CompressionLevel.Optimal` (máximo ahorro de espacio, más lento) o `CompressionLevel.Fastest` (más rápido, archivo más grande). Úsalo según tu prioridad.
2.  **Brotli para la Web:** Si estás desarrollando una API o un sistema que envía muchos datos por red, usa `BrotliStream`. Es más moderno y eficiente que GZip, aunque consume un poco más de CPU al comprimir.
3.  **No comprimas archivos ya comprimidos:** Intentar comprimir un `.jpg`, un `.mp4` o un `.pdf` con esta librería no reducirá su tamaño (incluso podría aumentarlo ligeramente) y solo gastarás CPU innecesariamente.
4.  **Cierre de Streams:** Siempre usa bloques `using`. Si no cierras correctamente el `ZipArchive` o el `GZipStream`, el archivo resultante podría quedar corrupto o incompleto.
5.  **Codificación de caracteres:** Si los nombres de los archivos dentro del ZIP tienen tildes o caracteres especiales, especifica `EntryNameEncoding` al abrir el archivo para evitar errores.

