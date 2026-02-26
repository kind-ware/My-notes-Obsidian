## Descripción General

Este espacio de nombres proporciona las clases necesarias para manipular el **Registro de Windows**. El registro es una base de datos jerárquica (similar a un sistema de archivos, con "carpetas" llamadas **Claves** y "archivos" llamados **Valores**) que almacena la configuración del sistema, del hardware y de las aplicaciones de usuario.

#### Casos de Uso Comunes

*   **Persistencia de Configuración:** Guardar preferencias del usuario que deben mantenerse tras cerrar el programa (ej. "Tema oscuro", "Última carpeta abierta").
*   **Auto-inicio (Startup):** Registrar una aplicación para que se ejecute automáticamente al iniciar Windows.
*   **Asociación de Archivos:** Configurar que ciertos archivos (ej. `.mi-extension`) se abran con tu programa.
*   **Lectura de Información del Sistema:** Obtener datos como la versión de Windows instalada, el nombre del dueño del equipo o configuraciones de red.

## Clases y Métodos Principales

*   **`Registry` (Clase Estática):** Es el punto de entrada. Proporciona acceso a las "Colmenas" (Hives) principales:
    *   `Registry.CurrentUser`: Configuración del usuario actual (no requiere admin).
    *   `Registry.LocalMachine`: Configuración global del equipo (requiere privilegios de **Administrador**).
    *   `Registry.ClassesRoot`: Tipos de archivos y objetos COM.
*   **`RegistryKey`:** Representa una clave específica dentro del registro (como una carpeta).
    *   `.OpenSubKey(name, writable)`: Abre una subclave (el segundo parámetro indica si vas a escribir en ella).
    *   `.CreateSubKey(name)`: Crea una nueva clave o la abre si ya existe.
    *   `.GetValue(name)`: Lee un valor.
    *   `.SetValue(name, value)`: Escribe un valor (soporta strings, ints, binarios, etc.).
    *   `.DeleteValue(name)` / `.DeleteSubKey(name)`: Borra datos.

## Ejemplos Prácticos

#### Ejemplo 1: Escribir y Leer una configuración de usuario

```csharp
using System;
using Microsoft.Win32;

class Program
{
    static void Main()
    {
        string subKeyPath = @"Software\MiGranProyecto";

        // 1. ESCRIBIR: Crear o abrir la clave en HKEY_CURRENT_USER
        using (RegistryKey key = Registry.CurrentUser.CreateSubKey(subKeyPath))
        {
            key.SetValue("UltimoUsuario", "Alex");
            key.SetValue("Version", 1.5);
            Console.WriteLine("Configuración guardada.");
        }

        // 2. LEER: Abrir la clave en modo lectura
        using (RegistryKey key = Registry.CurrentUser.OpenSubKey(subKeyPath))
        {
            if (key != null)
            {
                string usuario = (string)key.GetValue("UltimoUsuario");
                Console.WriteLine($"Bienvenido de nuevo, {usuario}");
            }
        }
    }
}
```

#### Ejemplo 2: Hacer que tu programa inicie con Windows

```csharp
string rutaApp = System.Reflection.Assembly.GetExecutingAssembly().Location;
string startupKey = @"Software\Microsoft\Windows\CurrentVersion\Run";

using (RegistryKey key = Registry.CurrentUser.OpenSubKey(startupKey, true))
{
    key.SetValue("MiAplicacionIncreible", $"\"{rutaApp}\"");
    Console.WriteLine("App configurada para iniciar con el sistema.");
}
```

#### Ejemplo 3: Leer información del Procesador (HKEY_LOCAL_MACHINE)

```csharp
string pathProcesador = @"HARDWARE\DESCRIPTION\System\CentralProcessor\0";
using (RegistryKey key = Registry.LocalMachine.OpenSubKey(pathProcesador))
{
    if (key != null)
    {
        string nombreProc = (string)key.GetValue("ProcessorNameString");
        Console.WriteLine($"Tu procesador es: {nombreProc}");
    }
}
```

## Buenas Prácticas y Consideraciones

1.  **Exclusivo de Windows:** Esta librería lanzará una excepción en Linux o macOS. Si usas .NET Core/5+, asegúrate de verificar `RuntimeInformation.IsOSPlatform(OSPlatform.Windows)`.
2.  **Usa `using`:** `RegistryKey` implementa `IDisposable`. Es vital cerrar las claves para liberar los manejadores del sistema operativo.
3.  **Privilegios:** Escribir en `LocalMachine` requiere que tu aplicación se ejecute como **Administrador**. Para configuraciones de tu propia app, usa siempre `CurrentUser`.
4.  **Manejo de Nulos:** `OpenSubKey` devolverá `null` si la clave no existe. Valídalo siempre antes de llamar a `GetValue` para evitar el temido `NullReferenceException`.
5.  **Tipos de datos:** El registro soporta `int` (DWORD) y `long` (QWORD). Al leer, recuerda hacer el *cast* correcto: `(int)key.GetValue("MiValor")`.
6.  **No abuses del Registro:** No guardes archivos grandes o imágenes aquí. El registro está diseñado para pequeñas piezas de configuración. Para datos grandes, usa archivos locales o bases de datos.

