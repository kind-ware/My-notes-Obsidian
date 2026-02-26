## Descripción General

Este espacio de nombres proporciona acceso a la funcionalidad básica de gráficos de GDI+. Permite crear imágenes desde cero, cargar archivos existentes (JPG, PNG, BMP, etc.), dibujar formas geométricas, manipular texto con fuentes específicas y aplicar transformaciones de color. Es la base de todo lo que se renderiza visualmente en las aplicaciones de escritorio tradicionales.

#### Casos de Uso Comunes

*   **Manipulación de Imágenes:** Cambiar el tamaño de una foto (Resizing), recortarla o rotarla.
*   **Marcas de Agua:** Superponer texto o logotipos sobre imágenes de forma automática.
*   **Gráficos Personalizados:** Dibujar diagramas, barras o pasteles estadísticos manualmente.
*   **Captura de Pantalla:** Tomar "screenshots" del escritorio o de una ventana específica.
*   **Filtros Básicos:** Convertir una imagen a escala de grises o ajustar su brillo/contraste.

## Clases y Estructuras Principales

*   **`Graphics`:** La clase más importante. Representa el "lienzo" donde se dibuja. No se instancia directamente, se obtiene de una ventana o de una imagen.
    *   `.DrawLine()`, `.DrawRectangle()`, `.DrawEllipse()`: Dibujan el contorno.
    *   `.FillRectangle()`, `.FillPie()`: Dibujan el interior relleno.
    *   `.DrawString()`: Escribe texto en el lienzo.
    *   `.DrawImage()`: Dibuja una imagen dentro de otra.
*   **`Bitmap` e `Image`:** Representan los datos de los píxeles. `Bitmap` es la implementación más común para trabajar con archivos de imagen.
*   **`Color`:** Estructura para definir colores (ej. `Color.Red`, `Color.FromArgb(255, 0, 0)`).
*   **`Pen`:** Se usa para dibujar **líneas y contornos**. Tiene grosor y color.
*   **`Brush` (y `SolidBrush`):** Se usa para **rellenar formas**.
*   **`Font`:** Define la familia tipográfica, el tamaño y el estilo del texto.
*   **`Rectangle`, `Point`, `Size`:** Estructuras matemáticas para definir posiciones y dimensiones.

## Ejemplos Prácticos

#### Ejemplo 1: Crear una imagen desde cero y dibujar en ella

```csharp
using System;
using System.Drawing; // Requiere referencia a System.Drawing.Common en .NET moderno
using System.Drawing.Imaging;

class Program
{
    static void Main()
    {
        // 1. Crear un lienzo de 400x200 píxeles
        using (Bitmap lienzo = new Bitmap(400, 200))
        {
            // 2. Crear el objeto Graphics para dibujar sobre el bitmap
            using (Graphics g = Graphics.FromImage(lienzo))
            {
                g.Clear(Color.White); // Fondo blanco

                // 3. Dibujar un rectángulo azul
                Pen pincelAzul = new Pen(Color.Blue, 5);
                g.DrawRectangle(pincelAzul, 50, 50, 300, 100);

                // 4. Escribir un texto rojo
                Font fuente = new Font("Arial", 16, FontStyle.Bold);
                g.DrawString("¡Hola desde GDI+!", fuente, Brushes.Red, 70, 85);
            }

            // 5. Guardar el resultado como archivo PNG
            lienzo.Save("resultado.png", ImageFormat.Png);
            Console.WriteLine("Imagen generada con éxito.");
        }
    }
}
```

#### Ejemplo 2: Cambiar el tamaño de una imagen (Resize)

```csharp
public Image Redimensionar(Image imagenOriginal, int ancho, int alto)
{
    Bitmap nuevaImagen = new Bitmap(ancho, alto);
    using (Graphics g = Graphics.FromImage(nuevaImagen))
    {
        // Configuramos la máxima calidad de interpolación
        g.InterpolationMode = System.Drawing.Drawing2D.InterpolationMode.HighQualityBicubic;
        g.DrawImage(imagenOriginal, 0, 0, ancho, alto);
    }
    return nuevaImagen;
}
```

## Buenas Prácticas y Consideraciones

1.  **¡LIBERA LA MEMORIA! (`using` obligatorio):** Las clases de `System.Drawing` (como `Graphics`, `Bitmap`, `Brush`, `Pen`) consumen recursos de GDI+ que no son manejados automáticamente por el recolector de basura de C# de forma inmediata. **Siempre** envuélvelas en un bloque `using` o llama a `.Dispose()`. No hacerlo causará fugas de memoria (*memory leaks*) rápidamente.
2.  **Referencia en .NET moderno:** En .NET 6 o superior, esta librería no viene por defecto. Debes instalar el paquete NuGet `System.Drawing.Common`.
3.  **Limitación Multiplataforma:** `System.Drawing.Common` está diseñado principalmente para **Windows**. Microsoft ha marcado esta librería como obsoleta para Linux/macOS en versiones recientes de .NET. Si necesitas procesar imágenes en la nube (Linux), se recomienda usar alternativas como **SkiaSharp** o **ImageSharp**.
4.  **Coordenadas:** El origen `(0,0)` es siempre la esquina **superior izquierda**. Las "Y" positivas van hacia abajo.
5.  **Rendimiento:** Evita crear objetos `Pen` o `Brush` dentro de un evento `Paint` de una ventana (que se ejecuta muchas veces por segundo). Créalos una vez y reutilízalos, o usa los estáticos como `Brushes.Black`.

