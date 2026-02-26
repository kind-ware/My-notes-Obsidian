### PyTest

`pytest` permite escribir pruebas (tests) para verificar que tu código hace lo que dice hacer.
Su filosofía es: **Menos código repetitivo (Boilerplate), más aserciones simples.**

**La Regla de Oro (Naming Convention):**
Para que `pytest` encuentre tus pruebas automáticamente:
1.  El archivo debe empezar por `test_` (ej: `test_main.py`).
2.  La función debe empezar por `test_` (ej: `def test_login():`).

### La Magia de `assert`

En otros lenguajes necesitas funciones raras como `assertEqual(a, b)` o `assertTrue(x)`.
En `pytest`, solo usas la palabra clave nativa de Python: **`assert`**.

*   Si la expresión es `True`, el test pasa (verde ✅).
*   Si es `False`, el test falla (rojo ❌) y `pytest` te muestra un reporte detallado de por qué falló.

### Funciones y Decoradores Principales

| Componente | Descripción |
| :--- | :--- |
| `pytest` (CLI) | El comando que ejecutas en la terminal para correr todos los tests. |
| `@pytest.fixture` | Prepara datos o configuraciones antes de un test (ej: conectar a DB) y limpia después. |
| `@pytest.mark.parametrize` | Permite ejecutar el mismo test muchas veces con diferentes datos de entrada. |
| `@pytest.mark.skip` | Salta un test (útil si una funcionalidad está rota pero la estás arreglando). |
| `pytest.raises` | Verifica que tu código lance un error cuando debe lanzarlo. |

### Ejemplos de Código

##### Ejemplo 1: Tu Primer Test (Unitario)

Crea un archivo llamado `test_calculadora.py`.

```python
# Código de la aplicación (normalmente importado, aquí lo pongo junto)
def dividir(a, b):
    if b == 0:
        raise ValueError("No se puede dividir por cero")
    return a / b

# --- LOS TESTS ---

def test_division_correcta():
    # Verificamos el caso feliz
    resultado = dividir(10, 2)
    assert resultado == 5

def test_division_decimal():
    # Verificamos decimales
    assert dividir(5, 2) == 2.5

import pytest

def test_division_por_cero():
    # Verificamos que lance el error correcto
    # "with pytest.raises" dice: "Espero que lo siguiente falle"
    with pytest.raises(ValueError):
        dividir(10, 0)
```
*Para ejecutarlo, abre la terminal y escribe:* `pytest`

##### Ejemplo 2: Fixtures (Preparar el terreno)

Imagina que necesitas un "Usuario Falso" para probar tu lógica. No quieres crearlo en cada función.

```python
import pytest
from pydantic import BaseModel

# Modelo de tu app
class Usuario(BaseModel):
    nombre: str
    es_admin: bool = False

# --- FIXTURE ---
@pytest.fixture
def usuario_normal():
    # Esto se ejecuta ANTES del test
    return Usuario(nombre="Juan", es_admin=False)

@pytest.fixture
def usuario_admin():
    return Usuario(nombre="Jefa", es_admin=True)

# --- TESTS USANDO FIXTURES ---
# Pasamos el nombre del fixture como argumento
def test_permisos_admin(usuario_admin):
    assert usuario_admin.es_admin is True

def test_permisos_usuario(usuario_normal):
    assert usuario_normal.es_admin is False
    assert usuario_normal.nombre == "Juan"
```

##### Ejemplo 3: Parametrización (DRY - Don't Repeat Yourself)

¿Quieres probar tu función de suma con 5 números diferentes? No escribas 5 funciones `def`.

```python
import pytest

def es_par(numero):
    return numero % 2 == 0

# Definimos: (input, esperado)
casos_de_prueba = [
    (2, True),
    (3, False),
    (0, True),
    (-4, True),
    (99, False)
]

@pytest.mark.parametrize("numero, resultado_esperado", casos_de_prueba)
def test_es_par(numero, resultado_esperado):
    # Esto se ejecutará 5 veces, una por cada tupla
    assert es_par(numero) == resultado_esperado
```

##### Ejemplo 4: Testing de Integración (FastAPI)

Esta es la joya. Pytest se integra con **FastAPI** para probar tu API sin levantar el servidor real.

*Nota: Requiere `pip install httpx`.*

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

# 1. Tu App FastAPI
app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id, "nombre": "Mesa"}

# 2. El Cliente de Test (Simula ser un navegador/httpx)
client = TestClient(app)

# 3. El Test
def test_leer_item():
    # Hacemos una petición real a la app (en memoria)
    response = client.get("/items/42")
    
    # 1. Verificar Status Code
    assert response.status_code == 200
    
    # 2. Verificar el JSON de respuesta
    datos = response.json()
    assert datos["item_id"] == 42
    assert datos["nombre"] == "Mesa"
```

### Integración con Stack

`pytest` protege cada ladrillo:

1.  **Validación**: Pruebas que tus modelos de **Pydantic** fallen si les pasas datos inválidos.
2.  **Lógica**: Pruebas que tus funciones de cálculo o procesamiento de texto funcionen.
3.  **API**: Usas `TestClient` para asegurar que tus rutas de **FastAPI** respondan 200 OK.
4.  **DB**: Usas fixtures para crear una base de datos **SQLite** temporal (`:memory:`), creas tablas con **SQLModel