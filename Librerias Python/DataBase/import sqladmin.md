### SQLAdmin

`sqladmin` es una interfaz de administración rápida y flexible para aplicaciones FastAPI y Starlette.
Se integra perfectamente con **SQLModel** (y SQLAlchemy).

**¿Qué te da "gratis"?**
*   Un **Dashboard** web.
*   Tablas de datos con **Paginación**.
*   Barras de **Búsqueda** y filtros.
*   Formularios para **Crear/Editar** registros (detecta si es fecha, número o texto).
*   Botones de **Eliminar**.

### Componentes Principales

Para que funcione, necesitas dos piezas:

| Componente  | Descripción                                                                                                            |
| :---------- | :--------------------------------------------------------------------------------------------------------------------- |
| `Admin`     | La aplicación principal. Se "monta" sobre tu FastAPI. Recibe el `app` y el `engine` de base de datos.                  |
| `ModelView` | Una clase que representa **una tabla** en el panel. Aquí configuras qué columnas se ven, cuáles se pueden buscar, etc. |

### Ejemplos de Código

*Nota: Asumiremos que ya tienes el setup de SQLModel del paso anterior (tabla `Heroe`).*

### Ejemplo 1: El Panel Básico

Este código crea un panel accesible en `/admin`.

```python
from fastapi import FastAPI
from sqlmodel import SQLModel, Field, create_engine
from sqladmin import Admin, ModelView

# --- 1. SETUP DE BASE DE DATOS (Lo que ya sabes) ---
class Heroe(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    nombre: str
    nombre_secreto: str
    edad: int | None = None

engine = create_engine("sqlite:///database.db")
SQLModel.metadata.create_all(engine)

app = FastAPI()

# --- 2. CONFIGURACIÓN DE SQLADMIN ---

# Definimos la vista para el modelo Heroe
class HeroeAdmin(ModelView, model=Heroe):
    # Esto define qué columna se muestra como título principal
    column_list = [Heroe.id, Heroe.nombre, Heroe.edad]

# Inicializamos el Admin
# authentication_backend=None (Por ahora sin clave, cuidado en producción)
admin = Admin(app, engine)

# Agregamos la vista al panel
admin.add_view(HeroeAdmin)

# ¡Listo! Corre esto con uvicorn y entra a http://127.0.0.1:8000/admin
```

### Ejemplo 2: Personalización (Iconos, Búsqueda, Orden)

El panel básico es útil, pero el personalizado es poderoso.
Puedes usar iconos de *FontAwesome* (ej: `fa-solid fa-user`).

```python
class HeroeAdmin(ModelView, model=Heroe):
    name = "Superhéroe"
    name_plural = "Superhéroes"
    icon = "fa-solid fa-mask" # Icono de máscara
    
    # ¿Qué columnas mostrar en la lista?
    column_list = [Heroe.id, Heroe.nombre, Heroe.nombre_secreto, Heroe.edad]
    
    # ¿Por qué columnas se puede buscar? (Barra de búsqueda)
    column_searchable_list = [Heroe.nombre, Heroe.nombre_secreto]
    
    # ¿Por qué columnas se puede ordenar (click en cabecera)?
    column_sortable_list = [Heroe.edad, Heroe.id]
    
    # Ocultar columnas sensibles en la lista (pero verlas al editar)
    # column_exclude_list = [Heroe.nombre_secreto]
    
    # Paginación (cuántos ver por hoja)
    page_size = 50
```

### Ejemplo 3: Seguridad (Autenticación)

Un panel admin sin contraseña es un peligro. `sqladmin` facilita ponerle un login.

```python
from sqladmin.authentication import AuthenticationBackend
from starlette.requests import Request
from starlette.responses import RedirectResponse

# 1. Definir la lógica de seguridad
class MiSeguridad(AuthenticationBackend):
    async def login(self, request: Request) -> bool:
        # Aquí capturas los datos del form de login
        form = await request.form()
        username = form.get("username")
        password = form.get("password")

        # VALIDACIÓN SIMPLE (En la vida real, checa la DB y usa hash)
        if username == "admin" and password == "1234":
            # Guardamos un token en la sesión (cookie)
            request.session.update({"token": "token_secreto"})
            return True
        return False

    async def logout(self, request: Request) -> bool:
        request.session.clear()
        return True

    async def authenticate(self, request: Request) -> bool:
        # Verificar si existe el token en la sesión
        token = request.session.get("token")
        return token is not None

# 2. Instanciar la seguridad
authentication_backend = MiSeguridad(secret_key="clave_super_secreta")

# 3. Pasar la seguridad al Admin
admin = Admin(app, engine, authentication_backend=authentication_backend)
```

### Integración con tu Stack

Mira cómo `sqladmin` complementa tu arquitectura:

1.  **FastAPI**: Sirve la API JSON para tu Frontend (React/Mobile) y clientes externos.
2.  **SQLModel**: Define la estructura de datos.
3.  **SQLAdmin**: Te da una **interfaz gráfica** inmediata para ti y tu equipo interno (Soporte, Gerentes).
    *   ¿El equipo de soporte necesita cambiar el email de un usuario? Entran al Admin.
    *   ¿Necesitas borrar un producto obsoleto? Entras al Admin.

### Diferencia: `FastAPI` (Swagger) vs `SQLAdmin`

Es fácil confundirse porque ambos muestran una web.

| Característica    | FastAPI Docs (Swagger UI)                 | SQLAdmin                                       |
| :---------------- | :---------------------------------------- | :--------------------------------------------- |
| **Objetivo**      | Probar los endpoints de la API (Técnico). | Gestionar los datos de la DB (Administrativo). |
| **Usuario**       | Programadores.                            | Dueños del negocio, Soporte, Admins.           |
| **Visualización** | Muestra JSON crudo.                       | Muestra Tablas, Gráficos y Formularios.        |
| **Uso**           | Depuración y Documentación.               | Gestión del día a día (CMS).                   |

### Resumen

Con `sqladmin`, te has ahorrado semanas de trabajo de desarrollo Frontend.

Estructura:
*   Un **Backend** robusto (FastAPI).
*   Una **Base de Datos** sólida (SQLModel/SQLite).
*   Un **Panel de Control** profesional (SQLAdmin).

¿Qué te parece? Tu aplicación ya se ve como un producto completo y vendible.
Si quieres seguir explorando, podemos ver **`pytest`** (para asegurar que tu código no falle con tests automáticos) o quizás **`python-dotenv`** (para manejar contraseñas y claves de forma segura y no hardcodeadas en el código). ¿Cuál prefieres?