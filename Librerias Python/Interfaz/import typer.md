### Typer

`typer` es una librería para construir aplicaciones de línea de comandos (CLI) que disfrutas escribir y usar.
Se basa en las **Type Hints** de Python.

**¿Por qué usarlo?**
*   **Intuitivo:** Escribe funciones normales de Python y Typer las convierte en comandos de terminal.
*   **Menos código:** Reduce drásticamente el código repetitivo comparado con `argparse`.
*   **Validación automática:** Valida tipos (si esperas un `int` y te dan "hola", falla automáticamente).
*   **Ayuda automática:** Genera manuales `--help` bonitos y coloridos.

### Conceptos Clave

La filosofía es simple: **Tu función es el comando. Sus parámetros son los argumentos.**

| Concepto                    | Sintaxis Python           | En la Terminal                   |
| :-------------------------- | :------------------------ | :------------------------------- |
| **Argumento** (Obligatorio) | `nombre: str`             | `python app.py Juan`             |
| **Opción** (Opcional)       | `apellido: str = "Perez"` | `python app.py --apellido Gomez` |
| **Flag** (Interruptor)      | `forzar: bool = False`    | `python app.py --forzar`         |

### Ejemplos de Código

##### Ejemplo 1: El "Hola Mundo" más fácil de la historia

Compara esto con las 10 líneas que necesitabas en `argparse`.

```python
import typer

# 1. Creamos la app
app = typer.Typer()

# 2. Convertimos la función en comando
@app.command()
def saludar(nombre: str):
    """
    Este script saluda a quien le digas.
    """
    print(f"Hola, {nombre}!")

if __name__ == "__main__":
    app()
```
*Uso en terminal:*
`python main.py Ana` -> Hola, Ana!
`python main.py --help` -> Muestra la ayuda generada.

##### Ejemplo 2: Opciones y Valores por Defecto

Aquí vemos cómo definir parámetros opcionales (`--algo`) y banderas (`--verbose`).

```python
import typer

app = typer.Typer()

@app.command()
def procesar(
    archivo: str,                  # Argumento (Obligatorio)
    veces: int = 1,                # Opción con valor por defecto (--veces 5)
    verbose: bool = False          # Flag (Boleano) (--verbose / --no-verbose)
):
    if verbose:
        print(f"Iniciando procesamiento del archivo: {archivo}")
    
    for _ in range(veces):
        print(f"Procesando {archivo}...")

if __name__ == "__main__":
    app()
```
*Uso:* `python main.py data.txt --veces 3 --verbose`

##### Ejemplo 3: Múltiples Comandos (Estilo Git)

Typer permite crear herramientas complejas con sub-comandos como `git add`, `git commit`, etc.

```python
import typer

app = typer.Typer()

@app.command()
def crear(usuario: str):
    print(f"Creando usuario: {usuario}")

@app.command()
def borrar(usuario: str):
    print(f"Borrando usuario: {usuario}")

@app.command()
def listar():
    print("Listando todos los usuarios...")

if __name__ == "__main__":
    app()
```
*Uso:*
`python main.py crear Juan`
`python main.py borrar Juan`
`python main.py listar`

##### Ejemplo 4: Interactividad (Prompts y Confirmaciones)

¿Necesitas pedir datos al usuario mientras corre el programa? Typer lo hace fácil y seguro.

```python
import typer

app = typer.Typer()

@app.command()
def registro():
    # Pedir input (como input() pero mejor)
    nombre = typer.prompt("¿Cuál es tu nombre?")
    
    # Pedir password (no se ve al escribir)
    password = typer.prompt("Crea una contraseña", hide_input=True)
    
    # Pregunta Sí/No (Abortar si dice que no)
    if not typer.confirm("¿Estás seguro de guardar estos datos?"):
        print("Operación cancelada.")
        raise typer.Abort()
    
    print(f"Usuario {nombre} registrado con éxito.")

if __name__ == "__main__":
    app()
```

##### Ejemplo 5: Integración con Rich (El Stack Definitivo)

Typer detecta si tienes `rich` instalado y formatea la ayuda con colores automáticamente. Pero también puedes usarlos juntos en el código.

```python
import typer
from rich.console import Console
from rich.table import Table

app = typer.Typer()
console = Console()

@app.command()
def reporte(titulo: str = "Reporte Mensual"):
    console.rule(f"[bold red]{titulo}")
    
    table = Table("ID", "Ventas", "Estado")
    table.add_row("1", "$500", "[green]Ok[/]")
    table.add_row("2", "$1200", "[green]Ok[/]")
    table.add_row("3", "$0", "[red]Error[/]")
    
    console.print(table)
    
    # Mensaje de salida con estilo de Typer (verde si ok, rojo si error)
    typer.secho("Reporte generado correctamente", fg=typer.colors.GREEN, bold=True)

if __name__ == "__main__":
    app()
```

### `argparse` vs `typer`

Para que veas la evolución:

| Característica           | `argparse`                              | `typer`                                 |
| :----------------------- | :-------------------------------------- | :-------------------------------------- |
| **Definición**           | Imperativa (`parser.add_argument(...)`) | Declarativa (Argumentos de función)     |
| **Tipado**               | Manual (`type=int`)                     | Automático (`a: int`)                   |
| **Ayuda**                | Texto plano simple                      | Texto formateado (Markdown/Colores)     |
| **Código**               | Verboso (muchas líneas)                 | Minimalista (pocas líneas)              |
| **Curva de aprendizaje** | Media                                   | Muy Baja (si sabes Python, sabes Typer) |
