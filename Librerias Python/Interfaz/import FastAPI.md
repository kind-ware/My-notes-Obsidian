### FastAPI

`FastAPI` permite construir APIs (Application Programming Interfaces) web de alto rendimiento.
Es famoso por tres cosas:

1.  **Velocidad:** Es tan rápido como NodeJS o Go (gracias a `starlette` y `pydantic`).
2.  **Validación Automática:** Si esperas un número y te mandan texto, FastAPI lanza el error por ti automáticamente.
3.  **Documentación Automática:** Genera una página web interactiva (Swagger UI) para probar tu API sin escribir una sola línea de HTML.
 
**Nota:** No viene instalado con Python. Necesitas instalar el framework y un servidor web (Uvicorn) para correrlo:  pip install fastapi uvicorn

### Conceptos Clave

Para usar FastAPI, definimos una "App" y luego usamos **decoradores** para decirle qué URL activa qué función.

| Decorador              | Método HTTP | Acción típica       |
| :--------------------- | :---------- | :------------------ |
| `@app.get("/ruta")`    | GET         | Leer datos.         |
| `@app.post("/ruta")`   | POST        | Crear datos nuevos. |
| `@app.put("/ruta")`    | PUT         | Actualizar datos.   |
| `@app.delete("/ruta")` | DELETE      | Borrar datos.       |

##### Pydantic (El compañero inseparable)

FastAPI usa una librería llamada `pydantic` para definir la estructura de los datos. Tú creas una clase (Modelo) y FastAPI se asegura de que los datos de entrada cumplan las reglas.

### Ejemplos de Código

**Importante:** Para correr estos ejemplos, guarda el código en un archivo llamado `main.py` y ejecuta en tu terminal:
`uvicorn main:app --reload`

*(Explicación: `main` es el nombre de tu archivo, `app` es la variable dentro del archivo, y `--reload` reinicia el servidor si cambias el código).*

##### Ejemplo 1: Hola Mundo y JSON automático

Fíjate que devolvemos un diccionario de Python, y FastAPI lo convierte a JSON automáticamente.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    # FastAPI convierte esto a JSON: {"mensaje": "Hola Mundo"}
    return {"mensaje": "Hola Mundo desde FastAPI"}

@app.get("/info")
async def info():
    return {
        "curso": "Python Pro",
        "nivel": "Avanzado",
        "estudiante": "Tú"
    }
```

##### Ejemplo 2: Parámetros de Ruta (Path Params)

Aquí usamos los **Type Hints** (`item_id: int`).
*   Si vas a `/items/5`, funciona.
*   Si vas a `/items/juan`, **FastAPI dará error automáticamente** diciendo que `juan` no es un entero. ¡Magia!

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id, "doble": item_id * 2}
```

##### Ejemplo 3: Parámetros de Consulta (Query Params)

Son los que van después del `?` en la URL (ej: `/buscar?q=python&limit=10`).
FastAPI sabe que si el argumento está en la función pero no en la ruta, es un Query Param.

```python
from fastapi import FastAPI

app = FastAPI()

# URL ejemplo: /productos/?categoria=tecnologia&precio_max=100
@app.get("/productos/")
async def listar_productos(categoria: str, precio_max: int = 50):
    return {
        "filtro_categoria": categoria,
        "filtro_precio": precio_max,
        "resultados": ["Mouse", "Teclado"] # Datos simulados
    }
```

##### Ejemplo 4: Recibir Datos (POST) con Pydantic

Aquí es donde brilla. Definimos la "forma" que deben tener los datos para crear un usuario.

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

# 1. Definimos el Modelo de datos
class Usuario(BaseModel):
    username: str
    email: str
    edad: int
    es_vip: Optional[bool] = False # Campo opcional
	
# 2. Creamos la ruta POST
@app.post("/usuarios/")
async def crear_usuario(usuario: Usuario):
    # 'usuario' ya es un objeto validado.
    # Si falta el email o la edad es texto, esto ni siquiera se ejecuta.
    
    print(f"Guardando en base de datos a: {usuario.username}")
    
    return {
        "mensaje": "Usuario creado con éxito",
        "datos_recibidos": usuario,
        "es_adulto": usuario.edad >= 18
    }
```

### La Joya: Documentación Automática

Una vez que tu servidor esté corriendo (con `uvicorn main:app --reload`), ve a tu navegador y entra a:

--> **`http://127.0.0.1:8000/docs`**

Verás una interfaz llamada **Swagger UI**.
*   Muestra todas tus rutas.
*   Te dice qué datos requiere cada una.
*   **¡Te deja probar la API directamente desde el navegador con un botón "Execute"!**

### Integración con otras librerias (`asyncio` + `httpx`)

FastAPI es asíncrono. Esto significa que puedes usar `await` dentro de tus rutas para llamar a otras APIs sin bloquear tu servidor.

```python
import httpx
from fastapi import FastAPI

app = FastAPI()

@app.get("/clima/{ciudad}")
async def obtener_clima(ciudad: str):
    # Usamos httpx (que ya conoces) para llamar a otra API
    async with httpx.AsyncClient() as client:
        # Simulamos una llamada externa
        # resp = await client.get(f"https://api.clima.com/{ciudad}")
        await asyncio.sleep(1) # Simulamos espera de red
        
    return {"ciudad": ciudad, "temperatura": "25°C (Simulado)"}
```
