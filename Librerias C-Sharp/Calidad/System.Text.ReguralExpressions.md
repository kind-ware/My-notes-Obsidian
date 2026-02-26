## Descripción General

Este espacio de nombres proporciona las clases necesarias para utilizar el motor de expresiones regulares de .NET. Permite realizar búsquedas de patrones complejos, validaciones de formato, extracciones de subcadenas y reemplazos masivos dentro de textos de forma mucho más potente que los métodos simples de `string` (como `Contains` o `Replace`).

#### Casos de Uso Comunes

*   **Validación de Formatos:** Comprobar si un texto es un email válido, un número de teléfono, un código postal o una contraseña segura.
*   **Extracción de Datos (Scraping):** Extraer todos los precios, fechas o enlaces de un bloque de texto o HTML.
*   **Limpieza de Texto:** Eliminar caracteres especiales, espacios dobles o etiquetas HTML de un string.
*   **Transformación de Datos:** Cambiar el formato de fechas (de `DD/MM/YYYY` a `YYYY-MM-DD`) de forma masiva.
*   **Parsing de Logs:** Analizar archivos de texto complejos para extraer códigos de error y marcas de tiempo.

## Clases y Métodos Principales

*   **`Regex` (Clase principal):** Contiene el motor de ejecución. Se puede usar mediante métodos estáticos o instanciando un objeto.
    *   `.IsMatch(input, pattern)`: Devuelve `true` si el patrón se encuentra en el texto.
    *   `.Match(input, pattern)`: Busca la **primera** coincidencia y devuelve un objeto `Match`.
    *   `.Matches(input, pattern)`: Busca **todas** las coincidencias y devuelve una `MatchCollection`.
    *   `.Replace(input, pattern, replacement)`: Reemplaza los textos que coincidan con el patrón.
    *   `.Split(input, pattern)`: Divide el string en un array usando el patrón como separador.
*   **`Match`:** Representa los resultados de una sola coincidencia.
    *   `.Success`: Indica si se encontró algo.
    *   `.Value`: El texto capturado.
    *   `.Groups`: Acceso a los grupos de captura (paréntesis en el regex).
*   **`RegexOptions` (Enumeración):** Modificadores de comportamiento.
    *   `IgnoreCase`: No distingue entre mayúsculas y minúsculas.
    *   `Multiline`: Cambia el comportamiento de `^` y `$` para que funcionen por línea.
    *   `Compiled`: Compila el regex a código binario para mayor velocidad (ideal para regex que se usan miles de veces).

## Ejemplos Prácticos

#### Ejemplo 1: Validación de Email (IsMatch)

```csharp
using System;
using System.Text.RegularExpressions;

string email = "contacto@ejemplo.com";
string patron = @"^[^@\s]+@[^@\s]+\.[^@\s]+$"; // Regex básico de email

bool esValido = Regex.IsMatch(email, patron);
Console.WriteLine(esValido ? "Email válido" : "Email inválido");
```

#### Ejemplo 2: Extracción de datos con grupos (Match)

Imagina que quieres extraer el código de área y el número de una cadena.

```csharp
string telefono = "Mi número es (55) 1234-5678";
string patronTel = @"\((\d{2})\)\s(\d{4}-\d{4})";

Match coincidencia = Regex.Match(telefono, patronTel);

if (coincidencia.Success)
{
    Console.WriteLine($"Código de área: {coincidencia.Groups[1].Value}");
    Console.WriteLine($"Número: {coincidencia.Groups[2].Value}");
}
```

#### Ejemplo 3: Reemplazo complejo

Ocultar números de tarjeta de crédito dejando solo los últimos 4.

```csharp
string texto = "Mi tarjeta es 1234-5678-9012-3456";
string patronTC = @"\d{4}-\d{4}-\d{4}-";

string oculto = Regex.Replace(texto, patronTC, "****-****-****-");
Console.WriteLine(oculto); // Salida: Mi tarjeta es ****-****-****-3456
```

## Buenas Prácticas y Consideraciones

1.  **Usa Cadenas Verbatim (`@""`):** En C#, las barras invertidas `\` se usan para escapar caracteres. Al escribir regex, usa el prefijo `@` para que las barras se traten literalmente (ej: `@" \d+"` en lugar de `"\\d+"`).
2.  **Compilación para rendimiento:** Si vas a ejecutar el mismo Regex dentro de un bucle miles de veces, instancia el objeto con la opción `RegexOptions.Compiled`. Esto tarda un poco más al inicio pero es mucho más rápido en cada ejecución.
3.  **Cuidado con el ReDoS:** Expresiones regulares mal diseñadas (con muchos asteriscos anidados) pueden causar un "Ataque de Denegación de Servicio por Regex" que congela la CPU. Siempre que sea posible, especifica un **Timeout**:
    `Regex.Match(input, patron, RegexOptions.None, TimeSpan.FromMilliseconds(200));`
4.  **Grupos con Nombre:** Para que tu código sea más legible, puedes ponerle nombre a los grupos en el regex: `(?<area>\d{2})`. Luego accedes con `coincidencia.Groups["area"]`.
5.  **Estático vs Instancia:** Si el regex lo usas una sola vez, usa el método estático `Regex.Match(...)`. .NET guarda un caché interno de los últimos regex estáticos usados para optimizar.
