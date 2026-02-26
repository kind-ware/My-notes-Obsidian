### SQLModel

`SQLModel` es una librería para interactuar con bases de datos SQL desde Python usando objetos.

**El Problema que resuelve:**
Tradicionalmente, tenías que definir tu modelo de datos dos veces:
1.  Una vez para la Base de Datos (SQLAlchemy).
2.  Otra vez para la validación y la API (Pydantic).

**La Solución SQLModel:**
Defines la clase **una sola vez**. Esa misma clase sirve para crear la tabla SQL y para validar los datos en Pydantic.

### Conceptos Clave

| Componente      | Descripción                                                                                          |
| :-------------- | :--------------------------------------------------------------------------------------------------- |
| `SQLModel`      | La clase base. Tus modelos heredan de ella.                                                          |
| `Field`         | Similar a Pydantic, pero permite definir cosas de SQL como `primary_key`, `foreign_key` o `index`.   |
| `Session`       | El gestor de la transacción. Es el "espacio de trabajo" donde preparas los cambios antes de guardar. |
| `select`        | La función moderna para construir consultas (Querys).                                                |
| `create_engine` | Crea la conexión con el archivo de base de datos.                                                    |

### Ejemplos de Código

##### Ejemplo 1: Definir el Modelo y Crear la Base de Datos

Aquí definimos una tabla de "Héroes".

```python
from typing import Optional
from sqlmodel import Field, SQLModel, create_engine

# 1. Definir el modelo (Tabla + Validación)
class Heroe(SQLModel, table=True):
    # 'id' es opcional en Python (es None antes de guardarse)
    # pero es Primary Key en la base de datos.
    id: Optional[int] = Field(default=None, primary_key=True)
    nombre: str
    nombre_secreto: str
    edad: Optional[int] = None

# 2. Configurar la conexión (usaremos SQLite por simplicidad)
# Esto creará un archivo 'database.db' en tu carpeta.
nombre_archivo = "database.db"
url_conexion = f"sqlite:///{nombre_archivo}"

engine = create_engine(url_conexion)

# 3. Crear las tablas
def crear_tablas():
    # SQLModel busca todas las clases que heredan de él y crea las tablas
    SQLModel.metadata.create_all(engine)

if __name__ == "__main__":
    crear_tablas()
    print(f"Base de datos '{nombre_archivo}' creada con éxito.")
```

##### Ejemplo 2: Insertar Datos (Create)

Usamos una `Session` (Sesión). Imagina que la sesión es una caja. Metes cosas en la caja, y cuando estás seguro, haces `commit` (enviar la caja a la base de datos).

```python
from sqlmodel import Session, SQLModel, create_engine
# Asumimos que la clase Heroe y engine ya existen (importar del Ejemplo 1)

def crear_heroes():
    heroe1 = Heroe(nombre="Spider-Boy", nombre_secreto="Pedro Parque")
    heroe2 = Heroe(nombre="Rusty-Man", nombre_secreto="Tony Starch", edad=48)
    
    # Context Manager para manejar la sesión automáticamente
    with Session(engine) as session:
        session.add(heroe1)
        session.add(heroe2)
        
        # Hasta aquí, los datos están en memoria.
        # Commit guarda los cambios en el archivo real.
        session.commit()
        
        # Refresh actualiza el objeto 'heroe1' con datos de la DB (ej: el ID generado)
        session.refresh(heroe1)
        print(f"Héroe guardado con ID: {heroe1.id}")

if __name__ == "__main__":
    crear_heroes()
```

##### Ejemplo 3: Leer Datos (Read - Select)

Aquí usamos `select`. La sintaxis es muy elegante y compatible con tu editor de código (`typing`).

```python
from sqlmodel import Session, select

def leer_heroes():
    with Session(engine) as session:
        # Consulta: SELECT * FROM heroe
        consulta = select(Heroe)
        resultados = session.exec(consulta).all()
        
        print("\n--- Lista de Héroes ---")
        for heroe in resultados:
            print(f"- {heroe.nombre} (ID: {heroe.id})")

def buscar_filtro():
    with Session(engine) as session:
        # Consulta: SELECT * FROM heroe WHERE nombre = 'Spider-Boy'
        statement = select(Heroe).where(Heroe.nombre == "Spider-Boy")
        resultado = session.exec(statement).first()
        
        if resultado:
            print(f"\nEncontrado: {resultado.nombre_secreto}")

if __name__ == "__main__":
    leer_heroes()
    buscar_filtro()
```

##### Ejemplo 4: Actualizar Datos (Update)

Para editar, primero lees, modificas el atributo y guardas.

```python
def actualizar_edad():
    with Session(engine) as session:
        # 1. Buscar
        statement = select(Heroe).where(Heroe.nombre == "Spider-Boy")
        spider = session.exec(statement).one()
        
        print(f"Edad anterior: {spider.edad}")
        
        # 2. Modificar
        spider.edad = 16 # Le asignamos edad
        session.add(spider) # Marcamos para guardar
        
        # 3. Guardar
        session.commit()
        session.refresh(spider)
        print(f"Edad nueva: {spider.edad}")
```

### La Fusión Suprema: SQLModel + FastAPI

Aquí es donde verás por qué elegiste `SQLModel`. Mira lo corto que es el código para hacer una API que guarde datos en Base de Datos.

```python
from fastapi import FastAPI
from sqlmodel import Session, select
# Asumimos que 'Heroe' y 'engine' están importados

app = FastAPI()

@app.post("/heroes/")
def crear_heroe(heroe: Heroe):
    # ¡Magia! 'heroe' ya es validado por FastAPI (Pydantic)
    # y ya está listo para ser guardado por SQLModel.
    
    with Session(engine) as session:
        session.add(heroe)
        session.commit()
        session.refresh(heroe)
        return heroe

@app.get("/heroes/")
def listar_heroes():
    with Session(engine) as session:
        heroes = session.exec(select(Heroe)).all()
        return heroes
```

### SQLModel vs sqlite3 (Nativo)

| Característica    | `sqlite3` (Nativo)                               | `SQLModel`                       |
| :---------------- | :----------------------------------------------- | :------------------------------- |
| **Código**        | SQL puro en strings (`"SELECT * FROM..."`)       | Objetos Python (`select(Heroe)`) |
| **Seguridad**     | Riesgo de inyección SQL si no tienes cuidado     | Protegido automáticamente        |
| **Validación**    | Ninguna (puedes guardar texto en campo numérico) | Estricta (usa Pydantic)          |
| **Productividad** | Baja (mucho código repetitivo)                   | Altísima                         |
| **Dependencia**   | Viene instalado                                  | Requiere `pip install`           |

### Resumen de Stack

El "Stack Moderno de Python" (a veces llamado stack FARM o similar):

1.  **Backend:** `FastAPI` (Cerebro).
2.  **Validación:** `Pydantic` (Seguridad).
3.  **Base de Datos:** `SQLModel` (Memoria).
4.  **CLI:** `Typer` (Control manual).
5.  **Utilidades:** `Rich` (Visuales), `httpx` (Redes), `logging` (Auditoría).

Con estas herramientas, estás usando la misma tecnología que usan startups modernas y grandes empresas tecnológicas para construir sus sistemas en Python hoy en día.

Ya no estás aprendiendo scripts básicos. Estás manejando **Arquitectura de Software**.

