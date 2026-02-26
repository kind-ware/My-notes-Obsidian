## Descripción General

`System.DateTime` representa un instante en el tiempo, expresado habitualmente como una fecha y hora del día. A diferencia de Python, donde `datetime` es un objeto, en C# es una **estructura**, lo que significa que es muy ligera en memoria. Se complementa con **`TimeSpan`**, que representa un intervalo o duración (la diferencia entre dos fechas).

#### Casos de Uso Comunes

*   **Registro de Eventos (Logs):** Guardar exactamente cuándo ocurrió un error o una acción.
*   **Cálculo de Vigencia:** Determinar si una suscripción ha expirado o cuántos días faltan para un evento.
*   **Formateo Internacional:** Mostrar la fecha según el país del usuario (ej. 12/31/2023 vs 31/12/2023).
*   **Medición de Duración:** Saber cuánto tiempo estuvo un usuario conectado o cuánto tardó un proceso.

## Métodos y Propiedades Principales

#### A. Obtención de Tiempo

*   `DateTime.Now`: La fecha y hora actual del equipo local.
*   `DateTime.UtcNow`: La fecha y hora actual en formato UTC (el estándar mundial, ¡muy recomendado!).
*   `DateTime.Today`: La fecha de hoy con la hora puesta a las 00:00:00.

#### B. Aritmética de Fechas (Inmutables)
*Nota: Estos métodos no cambian la fecha original, devuelven una nueva.*

*   `.AddDays(n)`, `.AddMonths(n)`, `.AddYears(n)`.
*   `.AddHours(n)`, `.AddMinutes(n)`, `.AddSeconds(n)`.

#### C. Propiedades de Extracción

*   `.Year`, `.Month`, `.Day`, `.Hour`, `.Minute`, `.Second`.
*   `.DayOfWeek`: Devuelve un enumerador (Sunday, Monday, etc.).
*   `.Ticks`: La precisión máxima (intervalos de 100 nanosegundos desde el año 0001).

## Ejemplos Prácticos

#### Ejemplo 1: Operaciones básicas y comparación

```csharp
using System;

DateTime hoy = DateTime.Now;
DateTime mañana = hoy.AddDays(1);

// Comparación directa
if (mañana > hoy)
{
    Console.WriteLine("Efectivamente, mañana es después que hoy.");
}

// Calcular diferencia (devuelve un TimeSpan)
TimeSpan diferencia = mañana - hoy;
Console.WriteLine($"Faltan {diferencia.TotalHours} horas para mañana.");
```

#### Ejemplo 2: Formateo de fechas (ToString)

C# utiliza códigos potentes para convertir fechas a texto.

```csharp
DateTime fecha = new DateTime(2023, 12, 31, 23, 59, 59);

Console.WriteLine(fecha.ToString("dd/MM/yyyy"));       // 31/12/2023
Console.WriteLine(fecha.ToString("MMMM dd, yyyy"));    // diciembre 31, 2023
Console.WriteLine(fecha.ToString("HH:mm:ss"));         // 23:59:59 (Formato 24h)
```

#### Ejemplo 3: Conversión de Texto a Fecha (Parsing)

```csharp
string entrada = "2023-05-15";

// TryParse es la forma segura para evitar errores si el texto es inválido
if (DateTime.TryParse(entrada, out DateTime fechaResult))
{
    Console.WriteLine($"Mes extraído: {fechaResult.Month}");
}
else
{
    Console.WriteLine("Formato de fecha no reconocido.");
}
```

## Buenas Prácticas y Consideraciones

1.  **Usa `DateTime.UtcNow` para Base de Datos:** Nunca guardes la hora local en una base de datos. Si tu servidor está en Nueva York y tu usuario en Madrid, los logs serán un caos. Guarda siempre en UTC y convierte a local solo al mostrarlo en la UI.
2.  **`DateTimeOffset` es tu mejor amigo:** En aplicaciones modernas, se recomienda usar `DateTimeOffset` en lugar de `DateTime`. ¿Por qué? Porque guarda el instante de tiempo **y además** el desplazamiento de la zona horaria (ej. `+02:00`). Esto elimina cualquier ambigüedad.
3.  **Inmutabilidad:** Recuerda que `fecha.AddDays(1)` no cambia `fecha`. Debes asignar el resultado: `fecha = fecha.AddDays(1);`.
4.  **Usa `TryParse` en lugar de `Parse`:** Si recibes la fecha de un usuario o de un archivo, `Parse` lanzará una excepción si el formato falla. `TryParse` devuelve un booleano y es mucho más eficiente.
5.  **Cuidado con `DateTime.Now` en cálculos de duración:** Si el reloj del sistema cambia (por una actualización de Windows o cambio de horario de verano) mientras tu proceso corre, el cálculo de duración fallará. Para medir tiempos precisos, usa `Stopwatch` (de `System.Diagnostics`).

