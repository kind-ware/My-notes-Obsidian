### JSON Format

El módulo `json` permite convertir datos de Python (diccionarios y listas) a texto en formato JSON (**J**ava**S**cript **O**bject **N**otation) y viceversa.

**¿Por qué es vital?**
*   **APIs:** Casi todos los datos que bajas de internet vienen en JSON.
*   **Configuración:** Es el formato estándar para guardar preferencias de usuario.
*   **Interoperabilidad:** Un archivo JSON generado en Python puede ser leído perfectamente por una página web (JavaScript).

### La Traducción (Mapeo)
Python traduce sus tipos de datos a los estándares de JSON automáticamente:

| Python | JSON |
| :--- | :--- |
| `dict` | Object `{...}` |
| `list` / `tuple` | Array `[...]` |
| `str` | String |
| `int` / `float` | Number |
| `True` / `False` | `true` / `false` (minúscula) |
| `None` | `null` |

### Funciones Principales (La regla de la "S")

Aquí está el secreto para no confundirse nunca. Hay 4 funciones principales divididas en dos grupos:

*   Las que terminan en **"s"** (`dumps`, `loads`) trabajan con **S**trings (texto en memoria).
*   Las que **NO** terminan en "s" (`dump`, `load`) trabajan con **Archivos** (Files).

| Función                | Significado              | Descripción                                                  |
| :--------------------- | :----------------------- | :----------------------------------------------------------- |
| `json.dumps(obj)`      | **Dump** to **S**tring   | Convierte un diccionario Python a texto (String JSON).       |
| `json.loads(str)`      | **Load** from **S**tring | Convierte un texto JSON a un diccionario Python.             |
| `json.dump(obj, file)` | **Dump** to File         | Guarda el diccionario directamente en un archivo `.json`.    |
| `json.load(file)`      | **Load** from File       | Lee un archivo `.json` y lo convierte en diccionario Python. |

### Ejemplos de Código

##### Ejemplo 1: De Python a Texto (`dumps`)
Útil cuando quieres enviar datos por red o imprimirlos en pantalla.

```python
import json

# Datos en Python (Diccionario)
usuario_py = {
    "nombre": "Ana",
    "edad": 28,
    "es_admin": False,
    "intereses": ["fútbol", "programación"],
    "pareja": None
}

# Convertir a String JSON
# ensure_ascii=False permite que se vean bien las tildes y la 'ñ'
datos_json = json.dumps(usuario_py, ensure_ascii=False)

print(f"Tipo de dato original: {type(usuario_py)}") # <class 'dict'>
print(f"Tipo de dato convertido: {type(datos_json)}") # <class 'str'>
print(f"Resultado:\n{datos_json}")

# Nota: En el resultado verás 'false' (minúscula) y 'null'.
```

##### Ejemplo 2: De Texto a Python (`loads`)

Simulamos que recibimos datos de una API (que siempre llegan como texto).

```python
import json

# Imaginemos que esto llegó de internet (es un String)
respuesta_api = '{"id": 101, "status": "ok", "mensajes": 5}'

# Convertimos ese texto a un objeto real de Python
datos = json.loads(respuesta_api)

print(f"Estado: {datos['status']}")
print(f"Mensajes nuevos: {datos['mensajes'] + 1}") # Podemos hacer matemáticas porque ya es int
```

##### Ejemplo 3: Guardar y Leer Archivos (`dump` y `load`)

Aquí es donde `json` brilla para guardar configuraciones. Usaremos lo aprendido en `os`.

```python
import json
import os

archivo_config = "configuracion.json"

# --- ESCRITURA (Guardar) ---
config_inicial = {
    "tema": "oscuro",
    "volumen": 80,
    "ruta_descargas": "C:/Downloads"
}

with open(archivo_config, "w") as f:
    # indent=4 hace que el archivo se vea bonito y legible
    json.dump(config_inicial, f, indent=4)
    print(f"Configuración guardada en {archivo_config}")


# --- LECTURA (Cargar) ---
if os.path.exists(archivo_config):
    with open(archivo_config, "r") as f:
        config_cargada = json.load(f)
        
    print("Configuración cargada:")
    print(f"- Tema: {config_cargada['tema']}")
    print(f"- Volumen: {config_cargada['volumen']}")
```

##### Ejemplo 4: Formato bonito (Pretty Print)

A veces el JSON sale todo en una sola línea y es imposible de leer. Usa `indent`.

```python
import json

data = {"usuarios": [{"id": 1, "nombre": "Luis"}, {"id": 2, "nombre": "Marta"}]}

# Sin indentación (Ahorra espacio, difícil de leer)
print("Compacto:", json.dumps(data))

# Con indentación (Fácil de leer para humanos)
print("\nLegible:")
print(json.dumps(data, indent=4))
```

### Limitaciones y Errores Comunes

##### El problema de `datetime`
El módulo `json` **NO sabe** cómo guardar fechas (`datetime`). Si intentas guardar un objeto `datetime`, dará error.

*Solución:* Convertir la fecha a texto (String) antes de guardar, usando lo que aprendimos en la lección anterior.

```python
import json
from datetime import datetime

ahora = datetime.now()
datos = {
    "evento": "Login",
    # "fecha": ahora  <-- ESTO DARÍA ERROR (TypeError)
    "fecha": ahora.strftime("%Y-%m-%d %H:%M:%S") # <-- ESTO ES CORRECTO
}

print(json.dumps(datos))
```

##### Comillas simples vs dobles
El estándar JSON **exige** comillas dobles `"` para las claves y cadenas.
*   Python acepta `{'clave': 'valor'}`.
*   JSON **solo** acepta `{"clave": "valor"}`.
*   El módulo `json` se encarga de corregir esto automáticamente, pero si escribes JSON a mano, recuérdalo.
