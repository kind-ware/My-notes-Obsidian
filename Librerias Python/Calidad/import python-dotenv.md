### DotEnv

`python-dotenv` lee pares de clave-valor de un archivo `.env` y los añade a las variables de entorno del sistema (`os.environ`).

**¿Por qué es esencial?**
*   **Seguridad:** Evita que subas contraseñas, tokens o claves privadas a GitHub/GitLab (Hardcoding).
*   **Configuración:** Te permite cambiar la configuración (ej: Base de datos de Desarrollo vs. Producción) sin tocar una sola línea de código Python.
*   **Estándar:** Sigue la metodología "The Twelve-Factor App", que es el estándar de la industria para aplicaciones modernas.

**Instalación:**
`pip install python-dotenv`

### El Archivo `.env`

Antes de ver código Python, necesitas crear un archivo llamado exactamente `.env` (sin nombre antes del punto) en la raíz de tu proyecto. Es un archivo de texto simple:

```ini
# Archivo: .env
DB_URL=sqlite:///produccion.db
SECRET_KEY=mi_clave_super_secreta_123
API_KEY_GOOGLE=AIzaSyD...
DEBUG=True
```

### Funciones Principales

| Función           | Descripción                                                                       |
| :---------------- | :-------------------------------------------------------------------------------- |
| `load_dotenv()`   | Busca un archivo `.env` y carga su contenido en memoria.                          |
| `dotenv_values()` | Carga las variables en un diccionario de Python (no toca el entorno del sistema). |
| `find_dotenv()`   | Intenta buscar el archivo `.env` automáticamente si no está en la carpeta actual. |

### Ejemplos de Código

##### Ejemplo 1: Uso Básico (Cargar y Leer)

Este es el patrón estándar que usarás en el 99% de tus scripts.

```python
import os
from dotenv import load_dotenv

# 1. Cargar las variables del archivo .env al sistema
load_dotenv()

# 2. Leerlas como si fueran variables del sistema operativo
# Si no existe, devuelve None (o el valor por defecto que pongas)
usuario_db = os.getenv("DB_USER")
clave_secreta = os.getenv("SECRET_KEY", "clave_por_defecto")

print(f"La clave secreta cargada es: {clave_secreta}")

if not usuario_db:
    print("Advertencia: No se configuró el usuario de base de datos.")
```

##### Ejemplo 2: Configuración de Base de Datos (Integración con SQLModel)

Así es como se protege la conexión a la base de datos en una app real.

```python
# Archivo .env:
# DATABASE_URL=sqlite:///mi_base_real.db

import os
from dotenv import load_dotenv
from sqlmodel import create_engine, SQLModel

# Carga las variables
load_dotenv()

# Obtenemos la URL. Si no hay .env, usará una temporal en memoria.
db_url = os.getenv("DATABASE_URL", "sqlite:///:memory:")

# Creamos el engine con la configuración segura
engine = create_engine(db_url)

print(f"Conectado a: {db_url}")
```

##### Ejemplo 3: No ensuciar el entorno (`dotenv_values`)

Si prefieres tener las configuraciones en un diccionario y no mezclarlas con `os.environ`.

```python
from dotenv import dotenv_values

# Esto devuelve un dict, no toca os.environ
config = dotenv_values(".env")

print(config)
# Salida: {'DB_URL': '...', 'SECRET_KEY': '...'}

print(f"Conectando a {config['DB_URL']}")
```

### 5. Reglas de Oro de Seguridad (Git)

Esta es la parte más importante de la lección:

1.  **NUNCA subas el archivo `.env` a Git.**
    Debes agregarlo a tu archivo `.gitignore`:
    ```text
    # .gitignore
    .env
    __pycache__/
    *.db
    ```

2.  **Crea un archivo `.env.example`**
    Sube este archivo a Git con claves falsas o vacías, para que otros desarrolladores sepan qué variables necesita tu programa.
    ```ini
    # Archivo: .env.example
    DB_URL=sqlite:///prueba.db
    SECRET_KEY=cambia_esto_por_tu_clave
    ```

### Integración con Pydantic

Ya que usamos **Pydantic** y **FastAPI**, existe una forma aún más elegante de hacer esto usando `pydantic-settings` (requiere `pip install pydantic-settings`).

Esto valida tus variables de entorno automáticamente (ej: asegura que el puerto sea un `int`).

```python
from pydantic_settings import BaseSettings

class Configuracion(BaseSettings):
    app_name: str = "Mi API Increíble"
    admin_email: str
    items_per_page: int = 50
    
    # Esto le dice a Pydantic que lea el archivo .env automáticamente
    class Config:
        env_file = ".env"

# Al instanciar, lee el .env, convierte tipos y valida
try:
    settings = Configuracion()
    print(f"Configuración cargada: {settings.app_name}")
    print(f"Items por página: {settings.items_per_page} (Tipo: {type(settings.items_per_page)})")
except Exception as e:
    print("Error: Faltan variables en el .env")
```

### Resumen del Stack Completo

Con **dotenv** la aplicación es segura y profesional.

1.  **`python-dotenv`**: Carga la clave secreta `SUPER_SECRET_KEY` desde un archivo oculto.
2.  **`os`**: Lee esa clave.
3.  **`sqladmin`**: Usa esa clave para autenticar al administrador en el panel.
4.  **`httpx`**: Usa la `API_KEY` del `.env` para conectarse a servicios externos sin exponerla en el código.
5.  **`pytest`**: Cuando corres tests, `pytest` puede cargar un `.env.test` diferente para no borrar tu base de datos real.
