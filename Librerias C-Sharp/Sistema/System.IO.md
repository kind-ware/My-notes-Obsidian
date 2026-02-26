## Descripción General

System.IO contiene los tipos (clases, interfaces, etc.) que permiten leer y escribir en archivos y flujos de datos (streams), así como proporcionar soporte básico para manipular archivos y directorios. En C#, interactuar con el sistema de archivos de forma directa, sincrónica o asincrónica, se hace a través de este namespace.

#### Casos de Uso Comunes

- **Lectura y escritura de archivos de texto o binarios** (ej. guardar logs, leer configuraciones locales, procesar datos CSV).    
- **Manipulación de rutas** (ej. extraer extensiones de archivos, unir carpetas de forma segura sin importar el sistema operativo).
- **Gestión del sistema de archivos** (ej. crear carpetas, listar todos los archivos dentro de un directorio, eliminar, mover o copiar archivos).
- **Manejo de flujos de datos grandes** (ej. descargar un archivo grande en fragmentos usando Streams sin saturar la memoria RAM).

## Clases y Métodos Principales

La librería se divide en clases **estáticas** (para operaciones rápidas de un solo uso) y clases **de instancia** (para realizar múltiples operaciones sobre el mismo archivo/directorio).

- **File (Estática):** Operaciones rápidas sobre archivos.
    
    - File.Exists(path): Verifica si un archivo existe (devuelve un booleano).
    - File.ReadAllText(path) / File.WriteAllText(path, content): Lee o escribe todo el contenido de una vez (ideal para archivos pequeños). 
    - File.Copy(), File.Move(), File.Delete(): Gestión del archivo.
    
- **Directory (Estática):** Operaciones sobre carpetas.
    
    - Directory.Exists(path): Verifica si la carpeta existe.
    - Directory.CreateDirectory(path): Crea una nueva carpeta.
    - Directory.GetFiles(path): Devuelve un arreglo con las rutas de los archivos dentro de la carpeta.
	
- **Path (Estática):** (Equivalente a os.path en Python). Manipulación segura de cadenas de texto que representan rutas.
    
    - Path.Combine(path1, path2): Une rutas usando el separador correcto (\ en Windows, / en Linux/Mac). 
    - Path.GetExtension(path): Extrae la extensión (ej. ".txt").
    - Path.GetFileName(path): Obtiene solo el nombre del archivo de una ruta completa.
    
- **StreamReader y StreamWriter:** Clases para leer o escribir archivos línea por línea, optimizadas para no cargar todo el archivo en memoria a la vez.

## Ejemplos Prácticos

#### Ejemplo 1: Manejo de Rutas y Creación de Carpetas (Path y Directory)

```C#
using System;
using System.IO;

class Program
{
    static void Main()
    {
        // Path.Combine es la forma segura de unir rutas
        string carpetaDestino = Path.Combine("C:", "MiProyecto", "Archivos");
        
        // Comprobar si existe, si no, crearla
        if (!Directory.Exists(carpetaDestino))
        {
            Directory.CreateDirectory(carpetaDestino);
            Console.WriteLine($"Carpeta creada en: {carpetaDestino}");
        }
    }
}
```

#### Ejemplo 2: Operaciones rápidas con archivos (File)

```C#
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string rutaArchivo = "mi_archivo.txt";
        
        // Escribir texto en un archivo (sobrescribe si ya existe)
        File.WriteAllText(rutaArchivo, "Hola, esta es la primera línea.\nSaludos desde C#.");

        // Leer todo el texto
        if (File.Exists(rutaArchivo))
        {
            string contenido = File.ReadAllText(rutaArchivo);
            Console.WriteLine("--- Contenido del archivo ---");
            Console.WriteLine(contenido);
        }
    }
}
```

#### Ejemplo 3: Leer archivos grandes eficientemente (StreamReader)

```C#
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string rutaArchivo = "archivo_grande.txt";
        File.WriteAllText(rutaArchivo, "Linea 1\nLinea 2\nLinea 3"); // Creamos archivo de prueba

        // El bloque 'using' asegura que el archivo se cierre y libere la memoria automáticamente
        using (StreamReader lector = new StreamReader(rutaArchivo))
        {
            string linea;
            // Lee línea por línea hasta que no haya más (null)
            while ((linea = lector.ReadLine()) != null)
            {
                Console.WriteLine($"Procesando: {linea}");
            }
        }
    }
}
```

## Buenas Prácticas y Consideraciones

1. **Usa Path.Combine siempre:** Nunca concatenes rutas con + "\\" +. C# es multiplataforma gracias a .NET Core/.NET 5+, y usar Path.Combine garantiza que tu código funcione tanto en Windows como en Linux/macOS.
2. **El bloque using es obligatorio con Streams:** Cuando uses StreamReader, StreamWriter, o abras un archivo manualmente, siempre envuélvelo en una instrucción using (como en el Ejemplo 3). Esto garantiza que el archivo se "cierre" y se libere (dispose) correctamente, incluso si ocurre un error. (Esto es el equivalente exacto a with open(...) as f: en Python).
3. **Para archivos grandes, no uses ReadAllText:** File.ReadAllText o File.ReadAllLines cargan todo en la memoria RAM. Si el archivo pesa varios Gigabytes, tu aplicación colapsará. Usa StreamReader o File.ReadLines() que cargan perezosamente (lazy evaluation).
4. **Aprovecha la Asincronía (Async/Await):** En aplicaciones modernas web o de escritorio, prefiere los métodos asíncronos para no congelar la interfaz: await File.ReadAllTextAsync().
