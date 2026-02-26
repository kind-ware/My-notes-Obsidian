### Rich

`rich` es una librería para escribir texto con formato enriquecido (colores, estilos) en la terminal.
Además, ofrece herramientas visuales avanzadas como tablas, barras de progreso y renderizado de código fuente, que funcionan en Windows, Linux y Mac.

**¿Por qué usarla?**
*   **Legibilidad:** Los errores son más fáciles de leer si están resaltados en rojo.
*   **Feedback:** Las barras de carga le dicen al usuario que el programa no se colgó.
*   **Depuración:** Muestra estructuras de datos (listas/diccionarios) de forma ordenada y coloreada.

### Los Componentes Principales

Aunque tiene muchas herramientas, usarás principalmente estas tres:

| Componente           | Descripción                                                                                               |
| :------------------- | :-------------------------------------------------------------------------------------------------------- |
| `rich.print`         | El reemplazo directo del `print()` nativo. Soporta etiquetas de estilo BBCode (ej: `[bold red]Texto[/]`). |
| `Console`            | El objeto principal para controlar la terminal (borrar pantalla, spinners, reglas).                       |
| `Table`              | Para crear tablas de datos ordenadas automáticamente.                                                     |
| `Progress` / `track` | Para crear barras de carga en bucles.                                                                     |
| `Traceback`          | Para que los mensajes de error (crashes) se vean bonitos y fáciles de entender.                           |

### Ejemplos de Código

##### Ejemplo 1: El "Hola Mundo" con Estilo (`print`)

Rich usa una sintaxis de etiquetas similar a HTML o foros antiguos.

```python
from rich import print

# Texto básico con estilos
print("Hola, [bold magenta]Mundo[/]! :earth_americas:")
print("[underline green]Todo listo[/] para [italic red]comenzar[/].")

# Imprimir diccionarios o listas coloreadas automáticamente
datos = {
    "id": 1,
    "lista": [10, 20, 30],
    "activo": True
}
print(datos) # Se imprime con colores sintácticos (Syntax Highlighting)
```

##### Ejemplo 2: La Consola (`Console`)

Para aplicaciones más serias, instanciamos un objeto `Console`.

```python
from rich.console import Console

console = Console()

console.print("Iniciando sistema...", style="bold blue")

# log() es como print() pero agrega la hora automáticamente
console.log("Conectando al servidor...")
console.log("Descargando datos...")

# Mostrar una línea separadora (Rule)
console.rule("[bold red]Reporte Final")
```

##### Ejemplo 3: Tablas Hermosas (`Table`)

Ideal para mostrar datos de bases de datos (`sqlite3`) o archivos CSV.

```python
from rich.console import Console
from rich.table import Table

console = Console()

# 1. Crear la tabla y definir título
table = Table(title="Películas Favoritas")

# 2. Agregar columnas (header_style define el color del encabezado)
table.add_column("Fecha", style="cyan", no_wrap=True)
table.add_column("Título", style="magenta")
table.add_column("Taquilla", justify="right", style="green")

# 3. Agregar filas
table.add_row("Dec 20, 2019", "Star Wars: The Rise of Skywalker", "$952,110,690")
table.add_row("May 25, 2018", "Solo: A Star Wars Story", "$393,151,347")
table.add_row("Dec 15, 2017", "Star Wars Ep. VIII: The Last Jedi", "$1,332,539,889")

# 4. Imprimir
console.print(table)
```

##### Ejemplo 4: Barras de Progreso (`track`)

Olvídate de imprimir "Cargando... 10%, Cargando... 20%".
Rich lo hace visual con `track`.

```python
import time
from rich.progress import track

# track envuelve tu lista o rango
# description es el texto que sale a la izquierda
for i in track(range(10), description="Procesando archivos..."):
    # Simulamos trabajo pesado
    time.sleep(0.5)
    
print("¡Proceso terminado!")
```

##### Ejemplo 5: Manejo de Errores Profesional (`Traceback`)

Cuando un script de Python falla, el muro de texto rojo asusta. Rich lo hace legible.

```python
# Instalar el manejador de tracebacks al inicio de tu script
from rich.traceback import install
install()

# Ahora provocamos un error para ver cómo se ve
def dividir(a, b):
    return a / b

def calcular():
    dividir(10, 0) # Error: División por cero

print("Intentando calcular...")
calcular()
```
*Al ejecutar esto, verás un reporte de error precioso, mostrando exactamente la línea de código y las variables locales.*

##### Ejemplo 6: Paneles y Markdown

Si quieres que tu script parezca una interfaz real o mostrar documentación.

```python
from rich.console import Console
from rich.panel import Panel
from rich.markdown import Markdown

console = Console()

aviso = """
# Atención
Este script va a borrar archivos.
* Asegúrate de tener backup.
* No cierres la terminal.
"""

md = Markdown(aviso)

# Panel crea un recuadro alrededor del contenido
console.print(Panel(md, title="Advertencia de Seguridad", border_style="red"))
```

### Stack Completo

Aquí es donde `rich` une todo lo que has aprendido para crear una CLI (Command Line Interface) de primer nivel.

**El Escenario:**
Un script que descarga datos de una API (`httpx`), valida con `pydantic` y muestra resultados.

```python
import time
import asyncio
import httpx
from pydantic import BaseModel
from rich.console import Console
from rich.table import Table
from rich.progress import track

console = Console()

# 1. Modelo Pydantic
class Usuario(BaseModel):
    id: int
    name: str
    email: str

# 2. Función Async de descarga
async def descargar_usuarios():
    url = "https://jsonplaceholder.typicode.com/users"
    
    with console.status("[bold green]Descargando datos de la API...[/]"):
        async with httpx.AsyncClient() as client:
            resp = await client.get(url)
            datos = resp.json()
            await asyncio.sleep(1.5) # Fake delay para ver la animación
            
    return datos

# 3. Main
async def main():
    console.rule("[bold blue]Gestor de Usuarios v1.0")
    
    raw_data = await descargar_usuarios()
    
    usuarios_validos = []
    
    # Barra de progreso mientras validamos
    for u in track(raw_data, description="Validando con Pydantic..."):
        user = Usuario(**u)
        usuarios_validos.append(user)
        time.sleep(0.1) # Simular proceso
        
    # Mostrar tabla
    table = Table(title="Usuarios del Sistema")
    table.add_column("ID", style="cyan")
    table.add_column("Nombre", style="white")
    table.add_column("Email", style="magenta")
    
    for user in usuarios_validos[:5]: # Solo los primeros 5
        table.add_row(str(user.id), user.name, user.email)
        
    console.print(table)
    console.print(f"\n[bold green] Proceso completado.[/] Total: {len(usuarios_validos)} usuarios.")

if __name__ == "__main__":
    asyncio.run(main())
```
