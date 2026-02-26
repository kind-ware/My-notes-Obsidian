### Shell Utilities

El módulo `shutil` ofrece operaciones de alto nivel sobre archivos y colecciones de archivos. Es la librería estándar que debes usar cuando necesites:

*   **Copiar** archivos (preservando datos y fechas).
*   **Mover** archivos o carpetas de un lugar a otro.
*   **Copiar carpetas enteras** con todo su contenido (recursivamente).
*   **Borrar carpetas llenas** (algo que `os` no puede hacer fácilmente).
*   Crear archivos comprimidos (ZIP, TAR).

### Funciones Principales

##### Copiado de Archivos

| Función                         | Descripción                                                                                                                             |
| :------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------- |
| `shutil.copy(origen, destino)`  | Copia el contenido del archivo y los permisos.                                                                                          |
| `shutil.copy2(origen, destino)` | **Recomendada**. Igual que la anterior, pero también intenta copiar los **metadatos** (fecha de creación, fecha de modificación, etc.). |

##### Operaciones con Carpetas (Directorios)

| Función                            | Descripción                                                                                           |
| :--------------------------------- | :---------------------------------------------------------------------------------------------------- |
| `shutil.copytree(origen, destino)` | Copia una carpeta **entera** y todo lo que tiene adentro a una nueva ubicación.                       |
| `shutil.rmtree(ruta)`              | **Peligrosa y Potente**. Borra una carpeta y **todo** su contenido (aunque tenga archivos dentro).    |
| `shutil.move(origen, destino)`     | Mueve un archivo o carpeta a otra ubicación (es como "Cortar y Pegar"). También sirve para renombrar. |

##### Utilidades del Sistema y Archivos

| Función                                    | Descripción                                                          |
| :----------------------------------------- | :------------------------------------------------------------------- |
| `shutil.disk_usage(ruta)`                  | Devuelve el espacio total, usado y libre del disco duro en esa ruta. |
| `shutil.make_archive(base, formato, ruta)` | Crea un archivo comprimido (zip, tar) de una carpeta.                |

### Ejemplos de Código

##### Ejemplo 1: Hacer un Backup de un archivo (Copiado)
Este es el uso más común: respaldar un archivo antes de modificarlo.

```python
import shutil
import os

archivo_original = "documento_importante.txt"
archivo_backup = "documento_importante_backup.txt"

# Creamos un archivo falso para el ejemplo
with open(archivo_original, 'w') as f:
    f.write("Datos cruciales de la empresa.")

# Usamos copy2 para preservar las fechas de modificación
if os.path.exists(archivo_original):
    shutil.copy2(archivo_original, archivo_backup)
    print(f"Respaldo creado: {archivo_backup}")
else:
    print("El archivo original no existe.")
```

##### Ejemplo 2: Copiar una carpeta entera (`copytree`)
Imagina que quieres duplicar toda la carpeta de un proyecto. `os` no puede hacer esto solo, pero `shutil` sí.

```python
import shutil
import os

origen = "proyecto_v1"
destino = "proyecto_v2"

# Generamos carpeta origen para el ejemplo
if not os.path.exists(origen):
    os.makedirs(os.path.join(origen, "subcarpeta"))

# Nota: copytree exige que la carpeta 'destino' NO exista previamente
# (la crea él mismo).
try:
    shutil.copytree(origen, destino)
    print(f"Se ha clonado la carpeta '{origen}' en '{destino}'")
except FileExistsError:
    print(f"Error: La carpeta '{destino}' ya existe.")
except Exception as e:
    print(f"Ocurrió un error: {e}")
```

##### Ejemplo 3: Mover y Organizar (`move`)
Sirve tanto para mover de una carpeta a otra como para renombrar archivos.

```python
import shutil
import os

# Supongamos que descargamos un archivo en 'Descargas' y queremos moverlo a 'Documentos'
archivo = "foto_vacaciones.jpg"
carpeta_destino = "mis_imagenes"

# Crear entorno de prueba
with open(archivo, 'w') as f: f.write("imagen")
if not os.path.exists(carpeta_destino): os.mkdir(carpeta_destino)

# Movemos el archivo
# shutil.move(qué_mover, a_dónde_mover)
ruta_final = shutil.move(archivo, carpeta_destino)

print(f"Archivo movido exitosamente a: {ruta_final}")
# Ahora 'foto_vacaciones.jpg' ya no está en la raíz, está dentro de 'mis_imagenes'
```

##### Ejemplo 4: El "Borrado Total" (`rmtree`)
**Advertencia:** Ten mucho cuidado con este comando.

```python
import shutil
import os

carpeta_basura = "temp_folder"

# Creamos una carpeta con cosas adentro para probar
os.makedirs(f"{carpeta_basura}/subcarpeta", exist_ok=True)

print(f"Borrando {carpeta_basura} y todo su contenido...")

# os.rmdir(carpeta_basura) # ESTO DARÍA ERROR porque la carpeta no está vacía
shutil.rmtree(carpeta_basura) # ESTO FUNCIONA

print("Borrado completado.")
```

##### Ejemplo 5: Comprimir archivos (Hacer un ZIP)
Python trae esto "gratis" con `shutil`.

```python
import shutil

carpeta_a_comprimir = "mis_imagenes" # La carpeta del Ejemplo 3
nombre_archivo_zip = "backup_imagenes" # Sin la extensión .zip

# make_archive(nombre_final, formato, carpeta_origen)
shutil.make_archive(nombre_archivo_zip, 'zip', carpeta_a_comprimir)

print(f"Se ha creado {nombre_archivo_zip}.zip")
```

### Diferencias enter OS y SHUTIL

1. **Usa os para:**
    - Ver archivos (listdir).
    - Crear carpetas (mkdir).
    - Borrar archivos individuales (remove).
    - Manejar rutas (path).
2. **Usa shutil para:**
    - **Copiar** cosas (archivos o carpetas).
    - **Mover** cosas.
    - **Borrar carpetas que no están vacías**.
    - Comprimir archivos.