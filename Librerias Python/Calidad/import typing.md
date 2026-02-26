### Typing

El módulo `typing` provee soporte para **Type Hints** (anotaciones de tipo).
Python no fuerza estos tipos en tiempo de ejecución (el programa no falla si le pasas un string a una función que espera un entero), pero herramientas externas (como `mypy`, editores de código, y librerías como Pydantic) usan esta información para validar tu código.

**Evolución Importante (Python Moderno):**
*   **Python < 3.9:** Debías importar todo: `from typing import List, Dict`.
*   **Python 3.9+:** Puedes usar los tipos nativos: `list[str]`, `dict[str, int]`.
*   **Python 3.10+:** Puedes usar `|` en lugar de `Union`.

Sin embargo, `typing` sigue siendo esencial para tipos especiales como `Optional`, `Any`, `Callable`, etc.

### Tipos Principales

| Tipo          | Descripción                                                | Ejemplo                                    |
| :------------ | :--------------------------------------------------------- | :----------------------------------------- |
| `List[T]`     | Una lista donde todos los elementos son del tipo T.        | `List[int]` (Lista de enteros)             |
| `Dict[K, V]`  | Un diccionario con claves tipo K y valores tipo V.         | `Dict[str, float]` (Nombre -> Precio)      |
| `Tuple[A, B]` | Una tupla con estructura fija (primero A, luego B).        | `Tuple[str, int]` ("Juan", 25)             |
| `Set[T]`      | Un conjunto de elementos únicos tipo T.                    | `Set[str]`                                 |
| `Optional[T]` | Puede ser del tipo T **o** `None`.                         | `Optional[int]` (Un número o nada)         |
| `Union[A, B]` | Puede ser del tipo A **o** del tipo B.                     | `Union[int, str]` (ID numérico o ID texto) |
| `Any`         | **Comodín.** Acepta cualquier cosa (desactiva el chequeo). | `Any`                                      |
| `Literal`     | Solo acepta valores específicos exactos.                   | `Literal["rojo", "verde"]`                 |
| `Callable`    | Para indicar que una variable es una **función**.          | `Callable[[int], str]`                     |

### Ejemplos de Código

##### Ejemplo 1: Listas y Diccionarios (`List`, `Dict`)

Definimos estructuras de datos estrictas.

```python
from typing import List, Dict

# Una lista que SOLO puede contener strings
nombres: List[str] = ["Ana", "Beto", "Carla"]

# Un diccionario: Clave String, Valor Entero
edades: Dict[str, int] = {
    "Ana": 25,
    "Beto": 30
}

# Esto no da error al correr (Python es dinámico), 
# pero tu editor lo subrayará en rojo.
# nombres.append(100) 
```

##### Ejemplo 2: El caso del valor nulo (`Optional`)

Este es el error más común en programación: intentar usar una variable que está vacía (`None`). `Optional` te obliga a pensar en ello.

```python
from typing import Optional

def buscar_usuario(id_usuario: int) -> Optional[str]:
    if id_usuario == 1:
        return "Juan Perez"
    else:
        return None  # No se encontró

usuario = buscar_usuario(99)

# Si tienes un linter configurado, te advertirá aquí:
# "Oye, 'usuario' podría ser None, no puedes usar .upper() directamente"
if usuario:
    print(usuario.upper())
else:
    print("Usuario no encontrado")
```

##### Ejemplo 3: Múltiples opciones (`Union`)

A veces un ID puede venir como número (`123`) o como texto (`"123"`).

```python
from typing import Union

# Python 3.10+ permite escribir: int | str
def procesar_id(item_id: Union[int, str]) -> str:
    # Como puede ser int o str, lo convertimos a str para estar seguros
    return f"Procesando ID: {str(item_id)}"

print(procesar_id(101))      # Válido
print(procesar_id("A-55"))   # Válido
# print(procesar_id(20.5))   # El editor marcará error (float no permitido)
```

##### Ejemplo 4: Restricciones estrictas (`Literal`)

Muy útil para configuraciones o estados, similar a un `Enum` pero más simple. Vital en **FastAPI**.

```python
from typing import Literal

def abrir_archivo(archivo: str, modo: Literal["r", "w", "a"]):
    print(f"Abriendo {archivo} en modo {modo}")

abrir_archivo("datos.txt", "r")  # Correcto
abrir_archivo("datos.txt", "w")  # Correcto

# El editor marcará error aquí antes de ejecutar, 
# porque "x" no está en la lista de permitidos.
# abrir_archivo("datos.txt", "x") 
```

##### Ejemplo 5: Funciones como argumentos (`Callable`)

Cuando pasas una función dentro de otra (Callbacks), `typing` ayuda a saber qué argumentos necesita esa función.

```python
from typing import Callable

# Definimos que 'funcion_log' debe ser:
# - Una función (Callable)
# - Que recibe un str ([str])
# - Y no devuelve nada (None)
def procesar_datos(mensaje: str, logger: Callable[[str], None]):
    print("Procesando...")
    logger(mensaje)

def mi_logger(texto: str):
    print(f"LOG: {texto}")

procesar_datos("Error crítico", mi_logger)
```

##### Ejemplo 6: Alias de Tipos (Organización)

Si tienes una estructura compleja que usas mucho, ponle nombre.

```python
from typing import Dict, List, Union

# Definimos un alias
Coordenada = Dict[str, Union[int, float]]
Camino = List[Coordenada]

# Ahora el código es más limpio
mapa: Camino = [
    {"x": 10, "y": 20.5},
    {"x": 5, "y": 0}
]
```

### `typing` vs `classes` (Pydantic)

Es fácil confundirse. ¿Cuándo uso `Dict` y cuándo un modelo de `Pydantic`?

**Usa `typing` (Dict/List) cuando:**
*   Los datos son simples o variables.
*   No necesitas validación de datos en tiempo de ejecución (solo ayuda visual).

**Usa `Pydantic` (BaseModel) cuando:**
*   Necesitas asegurar que los datos sean correctos (validación real).
*   La estructura es compleja (Usuarios, Productos).
*   Es una API (FastAPI).

```python
from typing import Dict, Any
from pydantic import BaseModel

# Typing: Solo sugiere, no valida al correr
datos_typing: Dict[str, Any] = {"edad": "veinte"} 

# Pydantic: Valida y lanza error si está mal
class Datos(BaseModel):
    edad: int

# d = Datos(edad="veinte") # <-- Esto explotaría (Error de validación)
```


## Integración con Stack

1.  **FastAPI**: Usa `typing` intensivamente. `def get_items(q: Optional[str] = None)`.
2.  **Typer**: Usa `typing` para definir argumentos CLI. `def main(name: str, force: bool = False)`.
3.  **Pydantic**: Usa `typing` para definir campos. `tags: List[str]`.
4.  **Rich**: Puedes usar `typing` para que tu editor te ayude a saber qué métodos tiene el objeto `Console`.
