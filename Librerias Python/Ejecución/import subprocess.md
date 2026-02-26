### Subprocess

El módulo `subprocess` permite generar nuevos procesos (ejecutar programas externos), conectarse a sus tuberías de entrada/salida/error y obtener sus códigos de retorno.

**¿Para qué sirve?**

*   Ejecutar comandos del sistema (reemplaza al antiguo `os.system`).
*   Automatizar tareas que harías manualmente en la terminal.
*   Capturar la respuesta de un comando externo para usarla en tu código Python.

### Funciones Principales

Desde Python 3.5, se recomienda usar **una única función** para la mayoría de los casos: `run()`.

| Función / Clase | Descripción |
| :--- | :--- |
| `subprocess.run(args, ...)` | **La recomendada.** Ejecuta el comando, espera a que termine y devuelve un objeto con el resultado. |
| `subprocess.Popen(...)` | **Avanzada.** Ofrece un control más granular (ej: interactuar con el proceso mientras corre o ejecutarlo en segundo plano sin esperar). |
| `subprocess.PIPE` | Constante especial para indicar que queremos capturar la salida (output) o enviar datos (input) al proceso. |
| `subprocess.DEVNULL` | Constante para ignorar (silenciar) la salida del comando. |

### Argumentos clave de `run()`

*   `args`: El comando a ejecutar (lista o string).
*   `capture_output=True`: Captura lo que el programa imprime (`stdout`) y sus errores (`stderr`).
*   `text=True` (o `universal_newlines=True`): Devuelve la salida como **texto** (string) en lugar de **bytes**.
*   `shell=True`: Ejecuta el comando a través del shell del sistema (`cmd.exe` o `bash`). *Cuidado: Riesgo de seguridad si usas inputs de usuario.*
*   `check=True`: Si el comando falla (error), Python lanza una excepción automáticamente.

### Ejemplos de Código

##### Ejemplo 1: Ejecución Básica (Ping)

Vamos a hacer un `ping` a Google. Nota cómo detectamos el sistema operativo (gracias a lo que aprendimos de `platform`) para elegir el comando correcto.

```python
import subprocess
import platform

# Definimos el comando según el sistema
parametro = "-n" if platform.system() == "Windows" else "-c"
comando = ["ping", parametro, "1", "google.com"]

print(f"Ejecutando: {' '.join(comando)}...")

# Ejecutamos el comando
# run espera a que el comando termine antes de seguir
resultado = subprocess.run(comando)

print("¡Terminado!")
# returncode 0 significa éxito, cualquier otro número es error
print(f"Código de retorno: {resultado.returncode}")
```

##### Ejemplo 2: Capturar la Salida (Lo más útil)

Imagina que quieres saber la versión de `git` instalada y guardar ese texto en una variable de Python.

```python
import subprocess

try:
    # capture_output=True: Guarda la respuesta
    # text=True: Nos da un string, no bytes
    resultado = subprocess.run(["git", "--version"], capture_output=True, text=True, check=True)
    
    # Accedemos a la salida estándar (stdout)
    version = resultado.stdout.strip() # .strip() quita espacios extra
    
    print(f"La versión detectada es: {version}")
    
except FileNotFoundError:
    print("Error: Parece que 'git' no está instalado o no está en el PATH.")
except subprocess.CalledProcessError as e:
    print(f"El comando falló con error: {e}")
```

##### Ejemplo 3: Manejo de Errores (`check=True`)

A veces los comandos fallan. Si usas `check=True`, Python te avisará.

```python
import subprocess

comando_invalido = ["cat", "archivo_que_no_existe.txt"]

try:
    # Intentamos leer un archivo inexistente (en Linux/Mac) o type (en Windows)
    # check=True lanzará una excepción si el returncode no es 0
    subprocess.run(comando_invalido, capture_output=True, text=True, check=True)

except subprocess.CalledProcessError as e:
    print("¡Ups! Hubo un problema.")
    print(f"Comando: {e.cmd}")
    print(f"Código de error: {e.returncode}") # Generalmente 1
    print(f"Mensaje de error del sistema: {e.stderr}") # Aquí dice "No such file..."
```

##### Ejemplo 4: Usar `shell=True` (Cuidado)

A veces quieres ejecutar comandos complejos con tuberías (`|`) o redirecciones (`>`) propias de la terminal.

```python
import subprocess

# Queremos listar archivos y guardar en un txt.
# Esto requiere shell=True porque ">" es una función de la terminal, no del programa.
comando = "echo 'Hola desde Python' > mensaje.txt"

# shell=True permite pasar el comando como un string completo
subprocess.run(comando, shell=True)

print("Archivo mensaje.txt creado.")

# Leemos para verificar (usando Python, no subprocess)
with open("mensaje.txt", "r") as f:
    print(f"Contenido: {f.read()}")
```

### Conceptos avanzados

##### Comparación entre `run` vs `Popen`

Para la mayoría de los scripts, `subprocess.run` es suficiente. Pero aquí está la diferencia:

1.  **`subprocess.run()`**: Es bloqueante. Tu script de Python **se detiene** y espera hasta que el comando externo termine. Es como llamar a una función.
2.  **`subprocess.Popen()`**: No es bloqueante (por defecto). Lanzas el comando y tu script de Python **sigue corriendo** en paralelo. Útil si quieres lanzar un servidor en segundo plano y seguir haciendo cosas.

##### Seguridad: La advertencia de `shell=True`

En la documentación siempre verás esta advertencia.

*   **Seguro:** `subprocess.run(["ls", "-l", nombre_archivo])`
    *   Python pasa los argumentos directamente al programa.
*   **Inseguro:** `subprocess.run(f"ls -l {nombre_archivo}", shell=True)`
    *   Si `nombre_archivo` contiene `"; rm -rf /"`, la terminal ejecutará el borrado. Esto se llama **Shell Injection**.

**Nota:** Siempre que sea posible, pasa el comando como una **lista** `['cmd', 'arg1', 'arg2']` y evita `shell=True`