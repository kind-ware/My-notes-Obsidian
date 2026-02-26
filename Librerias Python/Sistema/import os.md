### Operating System

El módulo `os` (Operating System) provee docenas de funciones para interactuar con el sistema operativo. Permite realizar operaciones como:

*   Navegar por carpetas (directorios).
*   Crear, borrar y renombrar archivos y carpetas.
*   Manejar rutas (paths) de archivos de forma compatible entre Windows y Linux/Mac.
*   Acceder a variables de entorno.

**Nota:** Es una librería estándar, por lo que no necesitas instalarla (`pip`), solo importarla.

### Funciones Principales
##### Navegación y Exploración

| Función            | Descripción                                                                                                       |
| :----------------- | :---------------------------------------------------------------------------------------------------------------- |
| `os.getcwd()`      | **G**et **C**urrent **W**orking **D**irectory. Devuelve la ruta de la carpeta donde estás trabajando actualmente. |
| `os.chdir(ruta)`   | **Ch**ange **Dir**ectory. Cambia la carpeta de trabajo actual a la `ruta` especificada.                           |
| `os.listdir(ruta)` | Devuelve una lista con los nombres de los archivos y carpetas dentro de la `ruta`.                                |

##### Manipulación de Archivos y Carpetas

| Función                   | Descripción                                                                        |
| :------------------------ | :--------------------------------------------------------------------------------- |
| `os.mkdir(ruta)`          | Crea una nueva carpeta. (Falla si la carpeta padre no existe).                     |
| `os.makedirs(ruta)`       | Crea una carpeta y todas las carpetas intermedias necesarias (creación recursiva). |
| `os.remove(archivo)`      | Elimina un archivo (da error si intentas borrar una carpeta).                      |
| `os.rmdir(ruta)`          | Elimina una carpeta **vacía**.                                                     |
| `os.rename(viejo, nuevo)` | Renombra o mueve un archivo/carpeta de un nombre a otro.                           |

##### Manejo de Rutas (`os.path`)
Esta es una sub-librería crucial para que tu código funcione igual en Windows (que usa `\`) y Linux/Mac (que usan `/`).

| Función                 | Descripción                                                                     |
| :---------------------- | :------------------------------------------------------------------------------ |
| `os.path.join(a, b)`    | Une componentes de una ruta usando el separador correcto del sistema operativo. |
| `os.path.exists(ruta)`  | Devuelve `True` si la ruta existe, `False` si no.                               |
| `os.path.isfile(ruta)`  | Devuelve `True` si es un archivo.                                               |
| `os.path.isdir(ruta)`   | Devuelve `True` si es una carpeta.                                              |
| `os.path.abspath(ruta)` | Obtiene la ruta absoluta (completa) de un archivo relativo.                     |
| `os.path.split(ruta)`   | Separa la ruta en (directorio, nombre_archivo).                                 |

### Ejemplos de Código

##### Ejemplo 1: ¿Dónde estoy y qué hay aquí?
Este script muestra tu ubicación actual y lista los archivos.

```python
import os

# 1. Obtener el directorio actual
directorio_actual = os.getcwd()
print(f"Estás trabajando en: {directorio_actual}")

# 2. Listar archivos en este directorio
contenido = os.listdir() # Si no pones argumento, usa el actual
print("Contenido de la carpeta:")
for item in contenido:
    print(f"- {item}")
```

##### Ejemplo 2: Creando y organizando carpetas
Este script crea una estructura de carpetas de forma segura.

```python
import os

nombre_carpeta = "mis_datos/reportes/2024"

# Usamos os.path.join para crear la ruta (Best Practice)
# Esto crea "mis_datos\reportes\2024" en Windows o "mis_datos/reportes/2024" en Linux
ruta_completa = os.path.join("mis_datos", "reportes", "2024")

print(f"Intentando crear: {ruta_completa}")

# Verificamos si existe antes de crear para evitar errores
if not os.path.exists(ruta_completa):
    # makedirs crea las carpetas intermedias (mis_datos y reportes) si no existen
    os.makedirs(ruta_completa)
    print("¡Estructura de carpetas creada con éxito!")
else:
    print("La carpeta ya existía.")
```

##### Ejemplo 3: Renombrar y Borrar (Gestión de Archivos)
*Nota: Para probar esto, asegúrate de tener un archivo de prueba llamado `test.txt` o el código dará error.*

```python
import os

# Supongamos que queremos renombrar 'test.txt' a 'archivo_final.txt'
archivo_original = "test.txt"
archivo_nuevo = "archivo_final.txt"

# 1. Verificar que el original existe
if os.path.exists(archivo_original):
    os.rename(archivo_original, archivo_nuevo)
    print(f"Renombrado: {archivo_original} -> {archivo_nuevo}")
else:
    print(f"El archivo {archivo_original} no existe, no se puede renombrar.")

# 2. Borrar el archivo (CUIDADO: Esto no va a la papelera, se borra permanentemente)
if os.path.exists(archivo_nuevo):
    # os.remove(archivo_nuevo) 
    # print(f"Archivo {archivo_nuevo} eliminado.")
    pass # Comentado por seguridad para que no borres nada por accidente al copiar/pegar
```

##### Ejemplo 4: Variables de Entorno
Muy útil para obtener configuraciones del sistema o claves secretas (API Keys) sin escribirlas en el código.

```python
import os

# Obtener una variable de entorno (Ej: El usuario del sistema)
usuario = os.environ.get('USERNAME') # En Windows
# usuario = os.environ.get('USER')   # En Linux/Mac

# Si la variable no existe, podemos dar un valor por defecto
editor = os.environ.get('EDITOR', 'Notepad')

print(f"Usuario del sistema: {usuario}")
print(f"Editor por defecto: {editor}")
```

### Consejos (Best Practices)

1.  **Usa siempre `os.path.join`**: Nunca concatenes rutas así: `carpeta + "/" + archivo`. Eso fallará en Windows. `os.path.join(carpeta, archivo)` es la forma correcta y profesional.
2.  **Manejo de Errores**: Al trabajar con archivos, siempre usa bloques `try/except` (FileNotFoundError, PermissionError) o verifica con `os.path.exists` antes de actuar.
3.  **La alternativa moderna**: Desde Python 3.4, existe una librería llamada **`pathlib`**. Hace lo mismo que `os.path` pero con una sintaxis orientada a objetos más moderna. Sin embargo, aprender `os` sigue siendo obligatorio porque se usa en millones de proyectos existentes.
