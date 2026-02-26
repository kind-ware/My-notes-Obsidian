### System-specific parameters and functions

El módulo sys (System-specific parameters and functions) provee acceso a variables y funciones que interactúan fuertemente con el **intérprete de Python**.

Se utiliza principalmente para:

- Leer **argumentos** enviados desde la terminal (línea de comandos).
- Finalizar la ejecución del script (**salir**).
- Identificar en qué **plataforma** (Windows/Linux) se está ejecutando Python.
- Manipular dónde busca Python las librerías para importar (path).

**Notas:** Si quieres "tocar" el disco duro, usa os o shutil.  
Si quieres "controlar" el flujo del script o su configuración, usa sys.

### Variables y Funciones Principales

A diferencia de os o shutil, sys tiene más **variables** (datos que puedes leer) que funciones.

|                 |             |                                                                                                                                         |
| --------------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| Nombre          | Tipo        | Descripción                                                                                                                             |
| sys.argv        | **Lista**   | La "joya de la corona" de sys. Contiene los argumentos de línea de comandos pasados al script. argv[0] es siempre el nombre del script. |
| sys.exit([arg]) | **Función** | Detiene el programa inmediatamente. Puedes pasar un número (0 = éxito, 1 = error) o un mensaje de texto.                                |
| sys.path        | **Lista**   | Lista de directorios donde Python busca módulos para importar. Puedes agregar carpetas aquí para importar tus propios scripts.          |
| sys.platform    | **String**  | Indica el sistema operativo (win32, linux, darwin para Mac).                                                                            |
| sys.executable  | **String**  | La ruta absoluta del archivo ejecutable de Python que está corriendo el script (útil para entornos virtuales).                          |
| sys.version     | **String**  | Información sobre la versión de Python instalada.                                                                                       |

### Ejemplos de Código

##### Ejemplo 1: Recibir argumentos desde la terminal (sys.argv)

Este es el uso más común. Permite que tu script sea dinámico sin cambiar el código.  
Imagina que ejecutas este script así desde la terminal:  
python mi_script.py Ziven 25

```python
import sys

# sys.argv es una lista.
# El índice 0 siempre es el nombre del archivo (mi_script.py)
print(f"Nombre del script: {sys.argv[0]}")

# Verificamos si nos pasaron argumentos extra
if len(sys.argv) > 1:
    nombre = sys.argv[1] # El primer argumento real
    print(f"Hola, {nombre}!")
    
    if len(sys.argv) > 2:
        edad = sys.argv[2]
        print(f"Tu edad es: {edad}")
else:
    print("No has pasado argumentos. Intenta ejecutar: python script.py [nombre]")
```

##### Ejemplo 2: Detener el programa (sys.exit)

Muy útil para manejo de errores. Si algo sale mal, "matas" el programa antes de que cause más problemas.

```python
import sys

variable_critica = None

print("Iniciando programa...")

# Simulamos un error
if variable_critica is None:
    print("Error fatal: La variable crítica está vacía.")
    # sys.exit detiene todo aquí. Lo de abajo nunca se imprimirá.
    sys.exit("El programa se cerró por seguridad.") 

print("Esto nunca se va a imprimir.")
```

##### Ejemplo 3: Identificar el Sistema Operativo (sys.platform)

Si tu script debe comportarse diferente en Windows o Linux (por ejemplo, limpiar la pantalla).

```python
import sys
import os

print(f"Detectando sistema: {sys.platform}")

if sys.platform == "win32":
    # Comando para limpiar consola en Windows
    print("Estás en Windows.")
    # os.system('cls') 
elif sys.platform.startswith("linux"):
    # Comando para limpiar consola en Linux
    print("Estás en Linux.")
    # os.system('clear')
else:
    print("Estás en Mac u otro sistema.")
```

##### Ejemplo 4: Manipular importaciones (sys.path)

Imagina que tienes una librería propia en una carpeta que no está instalada.

```python
import sys

# Vemos dónde busca Python las librerías
print("Rutas de búsqueda actuales:")
# for ruta in sys.path: print(ruta)

# Agregamos una ruta nueva temporalmente
nueva_ruta = "/home/usuario/mis_librerias_secretas"
sys.path.append(nueva_ruta)

print(f"Ahora Python también buscará librerías en: {nueva_ruta}")

# Ahora podrías hacer: import mi_libreria_secreta
```

### Diferencia Clave: os vs sys

Pregunta de entrevista técnica muy común:
1. **os (Operating System):**
    - Se encarga de lo que está **afuera** de Python.
    - Archivos, carpetas, procesos, variables de entorno del PC.
    - Ejemplo: Crear una carpeta, leer un archivo.
2. **sys (System/Interpreter):**
    - Se encarga de lo que está **adentro** del entorno de ejecución de Python.
    - Listas de importación, argumentos del script, memoria, versión del lenguaje.
    - Ejemplo: Saber qué versión de Python usas, salir del script, recibir parámetros.