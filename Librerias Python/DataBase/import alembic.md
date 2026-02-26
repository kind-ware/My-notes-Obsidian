### Alembic

`alembic` es una herramienta ligera de migración de bases de datos para usar con SQLAlchemy (y por ende, con **SQLModel**).

**El Flujo de Trabajo:**
1.  Modificas tu código Python (agregas un campo en tu modelo).
2.  Le pides a Alembic que detecte el cambio (`--autogenerate`).
3.  Alembic crea un script de "migración" (instrucciones de cómo cambiar la DB).
4.  Aplicas la migración y la base de datos se actualiza.

### Inicialización (Solo una vez)

A diferencia de otras librerías, Alembic funciona principalmente desde la **Terminal**, no importándolo en tu script Python.

##### Paso 1: Iniciar Alembic

En la carpeta raíz de tu proyecto:

```bash
alembic init alembic
```

Esto crea:
*   Un archivo `alembic.ini` (Configuración).
*   Una carpeta `alembic/` (Donde se guardarán las versiones).

##### Paso 2: Configurar la URL de la Base de Datos

Abre el archivo `alembic.ini` y busca la línea `sqlalchemy.url`.
Cámbiala por la ruta de tu base de datos (la misma que usas en SQLModel).

```ini
# alembic.ini
sqlalchemy.url = sqlite:///database.db
```

##### Paso 3: Conectar con SQLModel (El paso Crucial)

Por defecto, Alembic no "ve" tus modelos de SQLModel. Tienes que presentarlos.
Edita el archivo **`alembic/env.py`**:

```python
# Dentro de alembic/env.py

# 1. Importa SQLModel y tus modelos
from sqlmodel import SQLModel
from main import Heroe  # <--- Importa tus modelos aquí para que se registren

# 2. Busca la variable target_metadata
# target_metadata = None  <--- COMENTA ESTO O BORRALO

# 3. Asignale los metadatos de SQLModel
target_metadata = SQLModel.metadata
```
*Si no haces esto, Alembic no detectará tus tablas.*

### Comandos Principales (Terminal)

Estos son los comandos que usarás día a día.

| Comando                                        | Descripción                                                                                        |
| :--------------------------------------------- | :------------------------------------------------------------------------------------------------- |
| `alembic revision --autogenerate -m "mensaje"` | **Detecta cambios** en tus modelos Python y crea un archivo de migración. Es como un `git commit`. |
| `alembic upgrade head`                         | **Aplica** los cambios pendientes a la base de datos real. Es como un `git push`.                  |
| `alembic downgrade -1`                         | **Deshace** la última migración (vuelve al pasado). Útil si rompiste algo.                         |
| `alembic history`                              | Muestra la lista de todas las migraciones que has hecho.                                           |

### Ejemplo Práctico: Evolucionando una Tabla

Imagina que ya tienes tu tabla `Heroe` creada y funcionando.

##### Situación: El cliente pide un cambio

Necesitas agregar el campo `nivel_poder` a los héroes.

**1. Modificas tu código Python (`main.py`):**

```python
class Heroe(SQLModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    nombre: str
    nombre_secreto: str
    edad: Optional[int] = None
    
    # NUEVO CAMPO AGREGADO
    nivel_poder: int = Field(default=50) 
```

**2. Creas la migración (Terminal):**

```bash
alembic revision --autogenerate -m "Agregando nivel de poder"
```

*Alembic verá que en Python hay un campo nuevo que no existe en la DB y creará un archivo en `alembic/versions/xxxx_agregando_nivel_de_poder.py`.*

**3. Aplicas el cambio (Terminal):**

```bash
alembic upgrade head
```

*¡Listo! Tu base de datos `database.db` ahora tiene la columna `nivel_poder`, y los datos viejos siguen ahí.*

### Integración con `python-dotenv`

Como aprendimos en otras librerias, **no debes escribir la URL de la DB en `alembic.ini`** si vas a subir el código a Git.

Puedes hacer que Alembic lea tu archivo `.env`.
Edita `alembic/env.py` al principio:

```python
# alembic/env.py
import os
from dotenv import load_dotenv

load_dotenv() # Carga el .env

# ... más abajo en el archivo ...

# Sobrescribimos la configuración de la URL
config = context.config
url_db = os.getenv("DATABASE_URL")
config.set_main_option("sqlalchemy.url", url_db)
```

### Categorización y Resumen

Alembic entra en la categoría de **DevOps / Mantenimiento de Base de Datos**.

**Stack con Alembic:**
1.  **Desarrollo:** Modificas tus modelos `SQLModel` en Python.
2.  **Migración:** Usas `alembic revision` para capturar ese cambio.
3.  **Despliegue:** Cuando subes tu código al servidor, ejecutas `alembic upgrade head` y la base de datos de producción se actualiza sola, sin perder los usuarios registrados.

**Diferencia clave:**
*   **`SQLModel.metadata.create_all()`**: Úsalo solo para prototipos rápidos o tests (`pytest`).
*   **`alembic`**: Úsalo para cualquier proyecto real o profesional.
