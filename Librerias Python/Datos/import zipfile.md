### Descripción General

El módulo `zipfile` ofrece herramientas para crear, leer, escribir, añadir y listar archivos dentro de un contenedor ZIP.
A diferencia de `shutil` (que es de alto nivel), `zipfile` es de bajo nivel: tú decides archivo por archivo qué entra y cómo se llama dentro del ZIP.

**Concepto Clave: Compresión**
Por defecto, `zipfile` **NO comprime**, solo "empaqueta" (guarda los archivos juntos).
Para que el archivo pese menos, debes activar explícitamente el algoritmo `ZIP_DEFLATED`.

### La Clase Principal: `ZipFile`

Todo gira en torno a crear un objeto `ZipFile`.

```python
zipfile.ZipFile(archivo, modo, compression, allowZip64)
```

##### Modos de Apertura (`mode`)

| Modo  | Descripción                                                                         |
| :---- | :---------------------------------------------------------------------------------- |
| `'r'` | **Leer** un archivo ZIP existente. (Por defecto).                                   |
| `'w'` | **Escribir** (Crear nuevo). **Borra** el ZIP si ya existía.                         |
| `'a'` | **Añadir** (Append). Agrega archivos a un ZIP existente sin borrar lo que ya tiene. |
| `'x'` | **Creación Exclusiva**. Crea el archivo, pero falla si ya existe.                   |

##### Constantes de Compresión

| Constante              | Descripción                                                                       |
| :--------------------- | :-------------------------------------------------------------------------------- |
| `zipfile.ZIP_STORED`   | Solo guarda el archivo (Sin compresión). Es el default.                           |
| `zipfile.ZIP_DEFLATED` | **La que debes usar**. Comprime los archivos usando el algoritmo estándar (zlib). |
| `zipfile.ZIP_LZMA`     | Mayor compresión (más lento). Requiere módulo `lzma`.                             |

### Métodos Principales

| Método                     | Descripción                                                                                                  |
| :------------------------- | :----------------------------------------------------------------------------------------------------------- |
| `write(filename, arcname)` | Mete un archivo al ZIP. `filename` es la ruta en tu disco. `arcname` es el nombre que tendrá dentro del ZIP. |
| `extractall(path)`         | Descomprime **todo** en la carpeta indicada.                                                                 |
| `extract(member, path)`    | Descomprime **un solo archivo**.                                                                             |
| `namelist()`               | Devuelve una lista con los nombres de los archivos dentro del ZIP.                                           |
| `getinfo(name)`            | Devuelve metadatos (tamaño original, tamaño comprimido) de un archivo interno.                               |

### Ejemplos de Código

##### Ejemplo 1: Crear un ZIP comprimido (`'w'`)

Aquí vemos cómo guardar archivos dispersos en un solo paquete.

```python
import zipfile
import os

# Archivos que queremos comprimir (simulados)
archivos = ["reporte.txt", "foto.png", "datos.csv"]

# Creamos dummy files para que el ejemplo funcione
for f in archivos:
    with open(f, "w") as file: file.write("Contenido de prueba " * 100)

nombre_zip = "paquete_seguro.zip"

# Usamos 'with' para asegurar que el zip se cierre y guarde bien
# compression=zipfile.ZIP_DEFLATED es VITAL para ahorrar espacio
with zipfile.ZipFile(nombre_zip, "w", compression=zipfile.ZIP_DEFLATED) as zf:
    for archivo in archivos:
        print(f"Comprimiendo {archivo}...")
        # write(ruta_origen, nombre_en_zip)
        zf.write(archivo)

print(f"¡{nombre_zip} creado con éxito!")
```

### Ejemplo 2: Leer y Listar contenido (`'r'`)

Útil para inspeccionar un archivo que te mandaron antes de descomprimirlo.

```python
import zipfile

nombre_zip = "paquete_seguro.zip"

try:
    with zipfile.ZipFile(nombre_zip, "r") as zf:
        # 1. Ver qué hay dentro
        print("Archivos dentro del ZIP:")
        print(zf.namelist())
        
        # 2. Ver detalles técnicos (Ratio de compresión)
        info = zf.getinfo("reporte.txt")
        print(f"\nDetalle de 'reporte.txt':")
        print(f"- Tamaño original: {info.file_size} bytes")
        print(f"- Tamaño comprimido: {info.compress_size} bytes")
        
        # 3. Descomprimir todo
        zf.extractall("carpeta_descomprimida")
        print("\nArchivos extraídos.")
        
except FileNotFoundError:
    print("El archivo ZIP no existe.")
except zipfile.BadZipFile:
    print("El archivo está corrupto o no es un ZIP válido.")
```

##### Ejemplo 3: El truco de `arcname` (Rutas limpias)

Este es el error más común de los principiantes.
Si comprimes `C:/Usuarios/Juan/Documentos/hola.txt` sin usar `arcname`, al descomprimir **se creará toda esa estructura de carpetas**.

```python
import zipfile
import os

ruta_completa = os.path.join("carpeta_tmp", "subcarpeta", "secreto.txt")

# Simulamos crear la ruta y el archivo
os.makedirs(os.path.dirname(ruta_completa), exist_ok=True)
with open(ruta_completa, "w") as f: f.write("Top Secret")

with zipfile.ZipFile("backup.zip", "w") as zf:
    # MAL: Esto guarda la estructura de carpetas completa dentro del zip
    # zf.write(ruta_completa)
    
    # BIEN: Guardamos el archivo en la raíz del zip, sin carpetas extra
    zf.write(ruta_completa, arcname="secreto.txt")
    
    # PRO: Guardarlo dentro de una carpeta virtual dentro del zip
    zf.write(ruta_completa, arcname="documentos/2024/secreto.txt")

print("Backup creado con rutas limpias.")
```

##### Ejemplo 4: Añadir archivos a un ZIP existente (`'a'`)

Imagina que olvidaste meter un archivo en el backup.

```python
import zipfile

# Abrimos en modo 'a' (Append)
with zipfile.ZipFile("backup.zip", "a", compression=zipfile.ZIP_DEFLATED) as zf:
    # Escribimos un nuevo archivo (virtual, directo desde string)
    # writestr permite crear archivos dentro del zip sin tenerlos en disco
    zf.writestr("leeme.txt", "Este archivo fue agregado después.")

print("Archivo agregado al backup.")
```

### Integración con Stack

Mira cómo `zipfile` encaja perfectamente con lo que ya sabes:

1.  **Con `os.walk`**:
    *   Puedes recorrer una carpeta completa y comprimir archivo por archivo con control total (filtrando extensiones, por ejemplo).

2.  **Con `base64`**:
    *   Puedes leer un ZIP en modo binario (`rb`), convertirlo a Base64 y enviarlo por **`requests`** a una API.

3.  **Con `io.BytesIO` (Memoria)**:
    *   Si estás usando **FastAPI**, puedes crear un ZIP "al vuelo" en la memoria RAM (sin guardarlo en el disco duro) y devolverlo al usuario para que lo descargue inmediatamente.

```python
    import io
    import zipfile
    from fastapi import FastAPI, Response
    
    app = FastAPI()
    
    @app.get("/descargar-zip")
    def descargar():
        # Crear un buffer en memoria
        memoria_zip = io.BytesIO()
        
        with zipfile.ZipFile(memoria_zip, "w", zipfile.ZIP_DEFLATED) as zf:
            zf.writestr("hola.txt", "Hola desde la memoria RAM!")
            zf.writestr("data.csv", "id,nombre\n1,Juan")
            
        # Preparar respuesta
        headers = {"Content-Disposition": "attachment; filename=descarga.zip"}
        return Response(content=memoria_zip.getvalue(), media_type="application/zip", headers=headers)
```

