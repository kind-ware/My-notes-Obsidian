### Contex Library

El módulo `contextlib` ofrece utilidades para crear y trabajar con **Gestores de Contexto** (la sentencia `with`).

Normalmente, para crear un objeto compatible con `with`, tendrías que crear una Clase y definir dos métodos mágicos complejos: `__enter__` y `__exit__`.
**`contextlib` simplifica esto drásticamente**, permitiéndote crear un gestor de contexto usando una simple función y un generador (`yield`).

**¿Para qué sirve?**
*   Asegurar que recursos (conexiones, archivos, sockets) se cierren o liberen.
*   Manejar excepciones de forma limpia.
*   Ejecutar código de "inicio" y "fin" automáticamente.

### Lo principal: `@contextmanager`

Es el decorador principal. Convierte una función generadora en un Context Manager.

**La Estructura Mágica:**

```python
from contextlib import contextmanager

@contextmanager
def mi_gestor():
    # 1. CÓDIGO DE SETUP (Se ejecuta al entrar al 'with')
    print("Iniciando...")
    
    try:
        yield "objeto" # Aquí le das el control al bloque 'with'
    finally:
        # 2. CÓDIGO DE TEARDOWN (Se ejecuta al salir del 'with')
        # Esto corre SIEMPRE, incluso si hubo errores.
        print("Cerrando...")
```

### Funciones Principales

| Función / Decorador      | Descripción                                                                                                     |
| :----------------------- | :-------------------------------------------------------------------------------------------------------------- |
| `@contextmanager`        | Crea un gestor de contexto a partir de una función generadora.                                                  |
| `@asynccontextmanager`   | La versión asíncrona (vital para **FastAPI** y `asyncio`).                                                      |
| `suppress(*excepciones)` | Silencia errores específicos. Reemplaza al `try/except pass`.                                                   |
| `closing(objeto)`        | Fuerza el cierre (`.close()`) de un objeto al salir del bloque, aunque el objeto no soporte `with` nativamente. |
| `redirect_stdout`        | Redirige temporalmente los `print()` a otro lugar (ej: un archivo).                                             |

### Ejemplos de Código

##### Ejemplo 1: Creando tu propio Gestor (`@contextmanager`)

Vamos a crear un gestor que mida el tiempo de ejecución de un bloque de código automáticamente.

```python
import time
from contextlib import contextmanager

@contextmanager
def cronometro(nombre):
    inicio = time.time()
    print(f"⏱️  Iniciando tarea: {nombre}")
    
    try:
        yield # Aquí se ejecuta el código dentro del 'with'
    finally:
        fin = time.time()
        print(f"🏁 Tarea {nombre} terminada en {fin - inicio:.4f} segundos.")

# Uso
with cronometro("Cálculo Pesado"):
    # Simulamos trabajo
    time.sleep(1.5)
    print("   Haciendo cálculos...")

# Salida automática:
# ⏱️  Iniciando tarea: Cálculo Pesado
#    Haciendo cálculos...
# 🏁 Tarea Cálculo Pesado terminada en 1.500X segundos.
```

##### Ejemplo 2: El Silenciador (`suppress`)

¿Cansado de escribir `try: ... except FileNotFoundError: pass`?
`suppress` hace el código mucho más limpio.

```python
import os
from contextlib import suppress

archivo = "archivo_que_no_existe.txt"

# Forma vieja y verbosa
try:
    os.remove(archivo)
except FileNotFoundError:
    pass # No me importa si no existe

# Forma Pythonic con contextlib
with suppress(FileNotFoundError):
    os.remove(archivo)

print("Si el archivo estaba, se borró. Si no, no pasó nada.")
```

##### Ejemplo 3: Redirigir Salida (`redirect_stdout`)

Imagina que tienes una función vieja que usa `print()`, pero quieres que esa salida vaya a un archivo de log, no a la consola.

```python
from contextlib import redirect_stdout

def funcion_charlatana():
    print("Hola, soy un mensaje de consola.")
    print("Esto normalmente saldría en pantalla.")

print("--- Inicio ---")

with open("salida.log", "w") as f:
    # Todo lo que haga print aquí dentro, irá al archivo 'f'
    with redirect_stdout(f):
        funcion_charlatana()
        print("Esto también va al archivo.")

print("--- Fin (Revisa salida.log) ---")
```

##### Ejemplo 4: Gestión de Recursos (`closing`)

Útil para objetos que tienen un método `.close()` pero no soportan la sintaxis `with` (como `urllib` antiguo o algunas conexiones de bases de datos viejas).

```python
from contextlib import closing
from urllib.request import urlopen

# urlopen no siempre garantiza el cierre automático en versiones viejas
with closing(urlopen("https://www.google.com")) as pagina:
    for linea in pagina:
        # Procesar línea...
        pass
# Aquí 'pagina.close()' es llamado automáticamente.
```

##### Ejemplo 5: Asincronía (`@asynccontextmanager`)

Esto es fundamental si usas **FastAPI**. Se usa para definir eventos de "Startup" y "Shutdown" de la aplicación (como abrir y cerrar conexión a Base de Datos).

```python
import asyncio
from contextlib import asynccontextmanager

# Simulamos una base de datos
class BaseDeDatos:
    def conectar(self): print("[+] DB Conectada")
    def desconectar(self): print("[x]🔴 DB Desconectada")

db = BaseDeDatos()

@asynccontextmanager
async def ciclo_vida_db():
    db.conectar()
    try:
        yield db # Entregamos la DB para que se use
    finally:
        db.desconectar()

async def main():
    print("Arrancando app...")
    
    async with ciclo_vida_db() as database:
        print("   Ejecutando consultas en la app...")
        await asyncio.sleep(1)
    
    print("App cerrada.")

if __name__ == "__main__":
    asyncio.run(main())
```

