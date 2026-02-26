### Time

El módulo `time` proporciona funciones para interactuar con el reloj del sistema.
Su concepto central es el **Unix Timestamp** (Marca de tiempo Unix): Un número flotante que representa los segundos transcurridos desde el **1 de enero de 1970 (The Epoch)**.

**Usos principales:**

*   Pausar la ejecución de un programa (`sleep`).
*   Medir cuánto tarda en ejecutarse un bloque de código (Benchmarking).
*   Obtener la fecha/hora actual en formato crudo (para sistemas, no para humanos).

### Funciones Principales

| Función                | Descripción                                                                                                           | Ejemplo de retorno                  |
| :--------------------- | :-------------------------------------------------------------------------------------------------------------------- | :---------------------------------- |
| `time.time()`          | Devuelve el **Timestamp** actual (segundos desde 1970). Es la base de todo.                                           | `1706645.1234`                      |
| `time.sleep(segundos)` | Detiene (congela) la ejecución del hilo actual por X segundos.                                                        | `None`                              |
| `time.ctime(segundos)` | Convierte un timestamp en una cadena de texto legible para humanos. Si no pasas argumentos, usa el actual.            | `'Tue Jan 30 14:00:00 2024'`        |
| `time.perf_counter()`  | Devuelve un tiempo con **altísima precisión**. Se usa solo para medir intervalos (cronómetro), no para saber la hora. | `3456.12839`                        |
| `time.localtime()`     | Convierte el timestamp a una estructura (`struct_time`) con año, mes, día, hora separados.                            | `time.struct_time(tm_year=2024...)` |

### Ejemplos de Código

##### Ejemplo 1: El Timestamp (`time.time`)

Entendiendo el lenguaje de las máquinas.

```python
import time

# 1. Obtener el tiempo crudo
segundos = time.time()
print(f"Timestamp actual: {segundos}")

# ¿Por qué es útil? Para matemáticas simples.
# ¿Cuál será el timestamp dentro de una hora (3600 segundos)?
futuro = segundos + 3600
print(f"Timestamp en una hora: {futuro}")

# 2. Convertirlo a algo legible rápido
print(f"Fecha legible: {time.ctime(segundos)}")
```

##### Ejemplo 2: La Pausa (`time.sleep`)

Esencial para no saturar servidores al hacer peticiones o para esperar procesos.

```python
import time

print("Iniciando cuenta regresiva...")

for i in range(3, 0, -1):
    print(f"{i}...")
    time.sleep(1) # Pausa el programa 1 segundo exacto

print("¡Despegue! 🚀")
```

##### Ejemplo 3: Cronómetro de Rendimiento (`perf_counter`)

Esta es la forma **profesional** de medir cuánto tarda tu código.
*Nota: No uses `time.time()` para medir rendimiento, porque si el sistema actualiza su hora por internet (NTP) mientras mides, tu resultado será falso. `perf_counter` es inmune a eso.*

```python
import time

print("Calculando operación pesada...")

# Inicio del cronómetro
inicio = time.perf_counter()

# Simulamos tarea pesada (contar hasta 10 millones)
x = 0
for i in range(10000000):
    x += 1

# Fin del cronómetro
fin = time.perf_counter()

duracion = fin - inicio
print(f"La tarea tardó: {duracion:.6f} segundos.")
```

##### Ejemplo 4: Formateo manual (`strftime`)

Aunque `datetime` es mejor para esto, `time` también puede formatear fechas si ya tienes un `struct_time`.

```python
import time

# Obtener estructura de tiempo local
ahora = time.localtime()

# Formatear: Año-Mes-Día Hora:Minuto
# %Y=Año, %m=Mes, %d=Día, %H=Hora, %M=Minuto, %S=Segundo
texto = time.strftime("%Y-%m-%d %H:%M:%S", ahora)

print(f"Hora del sistema formateada: {texto}")
```

### Diferencias Clave y "Trampas"

##### `time` vs `datetime`

*   **Usa `datetime`** cuando necesites mostrar fechas a usuarios, sumar días, trabajar con zonas horarias o guardar en bases de datos.
*   **Usa `time`** cuando necesites pausar el programa (`sleep`) o medir cuánto tarda en ejecutarse una función (`perf_counter`).

##### `time.sleep()` vs `asyncio.sleep()`
Recuerda la lección de `asyncio`:

*   `time.sleep(5)`: Detiene **TODO**. Si tienes una interfaz gráfica, se congela. Si tienes un servidor, deja de responder a todos.
*   `await asyncio.sleep(5)`: Detiene solo esa tarea. El programa sigue atendiendo otras cosas.

##### El problema del Año 2038

El timestamp se guarda como un entero de 32 bits en muchos sistemas antiguos. El **19 de Enero de 2038**, los segundos superarán la capacidad de almacenamiento y las computadoras "volverán" al año 1901. Python moderno maneja esto bien (usa 64 bits), pero es un dato de cultura general importante sobre esta librería.

