### PyDantic

`pydantic` utiliza las **Type Hints** (pistas de tipo) de Python para validar datos.

**La Diferencia Clave (Parsing vs Validación):**
Pydantic no solo valida, **intenta arreglar** los datos (Parsing).

*   Si defines un campo como `int` y le envías el texto `"50"`, Pydantic lo convierte al número `50` automáticamente.
*   Si le envías `"cinco"`, Pydantic lanza un error porque no puede arreglarlo.

Mientras que `json` convierte texto a diccionarios "tontos" (donde todo vale), **`pydantic`** convierte datos en **objetos estrictos y validados**. Es el "Guardia de Seguridad" de tus datos: si algo no cumple las reglas, no entra.

**Nota:** Es una librería externa (y el corazón de FastAPI).
Instalación: `pip install pydantic`

##### El Bloque Base: `BaseModel`

Todo en Pydantic empieza heredando de la clase `BaseModel`.
Al hacerlo, tu clase gana superpoderes: validación en el constructor, exportación a JSON, etc.

### Ejemplos de Código

*(Nota: Usaremos la sintaxis de **Pydantic V2**, que es la versión moderna y actual).*

##### Ejemplo 1: Tu primer Modelo

Definimos qué forma deben tener los datos.

```python
from pydantic import BaseModel

# 1. Definimos la estructura
class Usuario(BaseModel):
    id: int
    nombre: str
    es_activo: bool = True  # Valor por defecto

# 2. Creamos una instancia (Happy Path)
# Nota: No hace falta escribir un __init__, Pydantic lo hace solo.
user1 = Usuario(id=101, nombre="Ana")

print(f"Usuario creado: {user1}")
print(f"ID: {user1.id} (Tipo: {type(user1.id)})")
```

##### Ejemplo 2: La Magia de la Conversión (Parsing)

Mira cómo Pydantic "limpia" los datos sucios que suelen llegar de formularios web o APIs.

```python
from pydantic import BaseModel

class Producto(BaseModel):
    nombre: str
    precio: float
    cantidad: int

# Datos "sucios" (strings en lugar de números)
datos_externos = {
    "nombre": "Laptop",
    "precio": "1500.50",  # Es un string
    "cantidad": "3"       # Es un string
}

# Pydantic los arregla automáticamente
item = Producto(**datos_externos)

print(f"Precio: {item.precio} (Tipo: {type(item.precio)})") 
# Salida: Precio: 1500.5 (Tipo: <class 'float'>)
```

##### Ejemplo 3: Manejo de Errores (`ValidationError`)

¿Qué pasa si los datos están muy mal y no se pueden arreglar?

```python
from pydantic import BaseModel, ValidationError

class Coordenada(BaseModel):
    x: int
    y: int

try:
    # Intentamos pasar "hola" donde debería ir un número
    punto = Coordenada(x=10, y="hola")
except ValidationError as e:
    print("¡Error de Validación!")
    # El error es muy detallado (dice qué campo falló y por qué)
    print(e.json()) 
```

##### Ejemplo 4: Tipos Avanzados y Opcionales

Pydantic brilla cuando usas tipos más complejos que `str` o `int`.

```python
from pydantic import BaseModel, EmailStr, HttpUrl
from typing import Optional, List

# Nota: Para usar EmailStr necesitas instalar: pip install pydantic[email]
# Si no quieres instalar eso, usa 'str' normal para el ejemplo.

class Empresa(BaseModel):
    nombre: str
    # Optional[str] significa que puede ser str o None
    sitio_web: Optional[str] = None 
    etiquetas: List[str]
    
data = {
    "nombre": "Tech Solutions",
    # sitio_web no lo enviamos, así que será None
    "etiquetas": ["SaaS", "AI", 123] # El 123 se convertirá a "123"
}

empresa = Empresa(**data)
print(empresa)
```

##### Ejemplo 5: Validaciones Personalizadas (`@field_validator`)

A veces los tipos no son suficientes. ¿Qué tal si el precio debe ser positivo? ¿O si la contraseña debe tener 8 caracteres?

```python
from pydantic import BaseModel, field_validator

class CuentaBancaria(BaseModel):
    titular: str
    saldo: float

    # Validamos que el saldo nunca sea negativo
    @field_validator('saldo')
    def saldo_positivo(cls, v):
        if v < 0:
            raise ValueError('El saldo no puede ser negativo')
        return v
    
    # Validamos que el nombre esté en mayúsculas (Normalización)
    @field_validator('titular')
    def nombre_mayusculas(cls, v):
        return v.upper()

try:
    cuenta = CuentaBancaria(titular="juan perez", saldo=-50)
except ValueError as e:
    print(f"Error detectado: {e}")

# Probamos uno válido para ver la normalización del nombre
cuenta_ok = CuentaBancaria(titular="maria lopez", saldo=100)
print(f"Titular guardado: {cuenta_ok.titular}") # MARIA LOPEZ
```

##### Ejemplo 6: Exportar datos (`model_dump` y `model_dump_json`)

Una vez que validaste y procesaste los datos, a menudo necesitas volver a convertirlos para guardarlos o enviarlos.

```python
from pydantic import BaseModel

class Coche(BaseModel):
    marca: str
    modelo: str

auto = Coche(marca="Toyota", modelo="Corolla")

# 1. Convertir a Diccionario de Python (para usar con otras librerías)
diccionario = auto.model_dump() 
print(f"Diccionario: {diccionario['marca']}")

# 2. Convertir a JSON String (para enviar por red o guardar archivo)
json_str = auto.model_dump_json()
print(f"JSON: {json_str}")
```

### Integración completa

Mira cómo `pydantic` limpia tu código anterior:

**Sin Pydantic (Usando `json` puro):**
```python
import json

datos = json.loads(respuesta_api)
# Tienes que validar a mano, lo cual es propenso a errores
if "id" in datos and isinstance(datos["id"], int):
    usuario_id = datos["id"]
else:
    raise Exception("ID inválido")
```

**Con Pydantic:**
```python
class Usuario(BaseModel):
    id: int

# Una sola línea valida, convierte tipos y lanza errores si falla.
usuario = Usuario(**json.loads(respuesta_api))
```
