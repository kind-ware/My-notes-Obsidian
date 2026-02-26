### Asyncio

`asyncio` es una librería para escribir código **concurrente** usando la sintaxis `async` / `await`.

Se utiliza principalmente para programas que pasan mucho tiempo "esperando" (I/O Bound), como:
*   Hacer peticiones a redes (Web scraping, APIs).
*   Consultas a bases de datos.
*   Servidores web.

**La Analogía del Camarero:**
*   **Síncrono (Normal):** El camarero toma el pedido de la Mesa 1, va a la cocina, *espera parado* a que cocinen, lleva la comida, y recién ahí va a la Mesa 2.
*   **Asíncrono (`asyncio`):** El camarero toma el pedido de la Mesa 1, entrega la nota a la cocina, y *mientras cocinan*, va a atender a la Mesa 2.

### Palabras Clave (La Sintaxis)

Para usar `asyncio`, el código cambia un poco de forma. Necesitas dominar dos palabras nuevas:

| Palabra     | Significado                                                                                                                                         |
| :---------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| `async def` | Define una función como una **Corutina**. Significa que esta función puede ser pausada y reanudada.                                                 |
| `await`     | Significa "Espera aquí a que esto termine, pero mientras tanto, deja que otras tareas se ejecuten". Solo se puede usar dentro de funciones `async`. |

### Funciones Principales

| Función                            | Descripción                                                                                                       |
| :--------------------------------- | :---------------------------------------------------------------------------------------------------------------- |
| `asyncio.run(corutina)`            | El botón de encendido. Inicia el bucle de eventos (Event Loop) y ejecuta la función principal.                    |
| `asyncio.sleep(segundos)`          | Pausa la tarea actual X segundos, pero **no bloquea** el programa (permite que otras tareas corran).              |
| `asyncio.gather(*tareas)`          | Ejecuta múltiples corutinas **al mismo tiempo** (concurrentemente) y espera a que todas terminen.                 |
| `asyncio.create_task(corutina)`    | Lanza una corutina en segundo plano ("fuego y olvido") para que se ejecute mientras tú sigues haciendo otra cosa. |
| `asyncio.wait_for(tarea, timeout)` | Espera a que una tarea termine, pero si tarda más del tiempo límite, la cancela (lanza error).                    |

### Ejemplos de Código

##### Ejemplo 1: Hola Mundo Asíncrono

Nota cómo definimos la función con `async def` y la ejecutamos con `asyncio.run()`.

```python
import asyncio

# Definimos una corutina
async def saludar():
    print("Hola...")
    # Simulamos una espera de 1 segundo (ej: base de datos)
    # Usamos await para "ceder el control"
    await asyncio.sleep(1)
    print("...Mundo!")

# Punto de entrada
if __name__ == "__main__":
    asyncio.run(saludar())
```

##### Ejemplo 2: El Poder de la Concurrencia (`gather`)

Aquí verás la magia. Vamos a ejecutar 3 tareas que tardan 2 segundos cada una.
*   Síncrono: Tardaría 6 segundos (2+2+2).
*   Asíncrono: Tardará **solo 2 segundos** (porque corren a la vez).

```python
import asyncio
import time

async def cocinar_plato(nombre, tiempo):
    print(f" Empezando a cocinar {nombre}...")
    await asyncio.sleep(tiempo) # Simula tiempo de cocción
    print(f"{nombre} está listo.")
    return f"{nombre} caliente"

async def servicio_cena():
    inicio = time.time()
    
    print("--- Cocina Abierta ---")
    
    # gather lanza las tres funciones a la vez y espera los resultados
    resultados = await asyncio.gather(
        cocinar_plato("Pasta", 2),
        cocinar_plato("Ensalada", 1),
        cocinar_plato("Bife", 3)
    )
    
    fin = time.time()
    
    print("--- Todos los platos listos ---")
    print(f"Resultados: {resultados}")
    print(f"Tiempo total transcurrido: {fin - inicio:.2f} segundos")

if __name__ == "__main__":
    asyncio.run(servicio_cena())

# RESULTADO: Verás que tarda ~3 segundos (el tiempo del Bife), no 6.
```

##### Ejemplo 3: Timeouts (No esperar para siempre)

Si una tarea tarda demasiado, la cancelamos.

```python
import asyncio

async def tarea_lenta():
    print("Iniciando tarea pesada...")
    await asyncio.sleep(5) # Tarda 5 segundos
    return "Terminado"

async def main():
    try:
        # Intentamos correrla, pero solo le damos 2 segundos de paciencia
        resultado = await asyncio.wait_for(tarea_lenta(), timeout=2)
        print(resultado)
    except asyncio.TimeoutError:
        print("¡Error! La tarea tardó demasiado y fue cancelada.")

if __name__ == "__main__":
    asyncio.run(main())
```

##### Ejemplo 4: Tareas en Segundo Plano (`create_task`)

A veces quieres que algo corra "en el fondo" mientras tu programa principal sigue.

```python
import asyncio

async def grabar_log():
    print("(Background) Guardando log en disco...")
    await asyncio.sleep(3)
    print("(Background) Log guardado.")

async def proceso_principal():
    # Creamos la tarea, esto la agenda para ejecutarse "ya mismo"
    tarea = asyncio.create_task(grabar_log())
    
    print("El usuario está usando la aplicación...")
    await asyncio.sleep(1)
    print("El usuario cerró la sesión.")
    
    # Esperamos a que la tarea de fondo termine antes de cerrar el script
    await tarea 

if __name__ == "__main__":
    asyncio.run(proceso_principal())
```

### 5. Diferencia Clave: `time.sleep` vs `asyncio.sleep`

Este es el error número 1 de los principiantes.

1.  **`time.sleep(5)` (Bloqueante):**
    *   Detiene **todo** el programa.
    *   Nadie hace nada por 5 segundos. La CPU se queda quieta.
    *   *Si lo usas dentro de una función `async`, rompes la asincronía.*

2.  **`await asyncio.sleep(5)` (No Bloqueante):**
    *   Pausa **solo esa tarea**.
    *   Le dice al sistema: "Despiértame en 5 segundos, mientras tanto, **ejecuta otras tareas** que estén pendientes".

### Una advertencia sobre `requests`

Cuidado `requests` es **síncrona (bloqueante)**.

Si usas `requests.get()` dentro de una función `async`, bloquearás todo el programa y perderás las ventajas de velocidad.

Para hacer peticiones HTTP asíncronas, se suelen usar librerías externas diseñadas para `asyncio` como **`aiohttp`** o **`httpx`**.

**Ejemplo conceptual (Simulado):**

```python
# MALA PRÁCTICA en asyncio
# async def obtener_web():
#     requests.get("google.com") # <-- Esto frena a todos los demás

# BUENA PRÁCTICA
# async def obtener_web():
#     async with aiohttp.ClientSession() as session:
#         async with session.get("google.com") as resp:
#             data = await resp.text()
```

