### Threading

El módulo `threading` permite ejecutar múltiples operaciones "al mismo tiempo" (concurrentemente) dentro del mismo programa.

Imagina que tu script es una fábrica:

*   **Sin threading:** Tienes 1 solo trabajador. Si tiene que esperar a que llegue un camión, toda la fábrica se detiene.
*   **Con threading:** Contratas 5 trabajadores. Si uno espera al camión, los otros 4 siguen empaquetando cajas.

**La Regla de Oro (GIL):**

En Python, los hilos son ideales para tareas de **I/O Bound** (Red, Disco, esperar respuestas). No son buenos para tareas de **CPU Bound** (Cálculos matemáticos pesados), debido a una limitación llamada *Global Interpreter Lock (GIL)*. Para matemática pesada, se usa `multiprocessing`.

### Clases y Funciones Principales

##### La Clase `Thread` (El Trabajador)

Es la pieza central. Creas una instancia de `Thread` y le asignas una función para ejecutar.

| Método                                   | Descripción                                                                                         |
| :--------------------------------------- | :-------------------------------------------------------------------------------------------------- |
| `threading.Thread(target=..., args=...)` | Crea el hilo. `target` es la función a ejecutar, `args` son los argumentos que recibe esa función.  |
| `.start()`                               | **¡Acción!** Inicia el hilo. El código principal sigue corriendo inmediatamente sin esperar.        |
| `.join()`                                | Le dice al programa principal: "Detente aquí y **espera** a que este hilo termine antes de seguir". |
| `.is_alive()`                            | Devuelve `True` si el hilo sigue trabajando.                                                        |

##### Sincronización (`Lock`)

Cuando varios hilos intentan escribir en la misma variable a la vez, ocurren desastres (Race Conditions). Un `Lock` es como la llave del baño: solo uno puede entrar a la vez.

| Objeto             | Descripción                                                         |
| :----------------- | :------------------------------------------------------------------ |
| `threading.Lock()` | Crea un candado.                                                    |
| `.acquire()`       | Cierra el candado (si ya está cerrado, espera hasta que se libere). |
| `.release()`       | Abre el candado para que otro lo use.                               |

### Ejemplos de Código

##### Ejemplo 1: Hilos vs Secuencial (La diferencia básica)

Vamos a simular una tarea lenta (como bajar un archivo) que tarda 2 segundos.

```python
import threading
import time

def tarea_lenta(nombre):
    print(f"Inicio tarea: {nombre}")
    time.sleep(2) # Simula bloqueo (red/disco)
    print(f"Fin tarea: {nombre}")

print("--- MODO NORMAL (Secuencial) ---")
inicio = time.time()
tarea_lenta("A")
tarea_lenta("B")
print(f"Tiempo total: {time.time() - inicio:.2f} segs") # Tardará 4 segs

print("\n--- MODO THREADING (Paralelo) ---")
inicio = time.time()

# 1. Crear los hilos
hilo1 = threading.Thread(target=tarea_lenta, args=("Hilo_A",))
hilo2 = threading.Thread(target=tarea_lenta, args=("Hilo_B",))

# 2. Arrancarlos (Aquí empieza la magia)
hilo1.start()
hilo2.start()

# 3. Esperar a que terminen (Join)
# Si no ponemos join, el programa principal terminaría antes que los hilos.
hilo1.join()
hilo2.join()

print(f"Tiempo total: {time.time() - inicio:.2f} segs") # Tardará ~2 segs
```

##### Ejemplo 2: Scapy + Threading (Un caso real)

Como `scapy` es lento escuchando la red (`sniff`), si lo pones en tu script principal, el script se congela. La solución es mandarlo a un hilo.

```python
import threading
import time
# from scapy.all import sniff (Comentado para que no de error si no eres root)

# Bandera para controlar el hilo
continuar_escuchando = True

def sniffer_en_segundo_plano():
    print("[+] Sniffer iniciado (Simulado)...")
    while continuar_escuchando:
        # Aquí iría: sniff(count=1, prn=procesar_paquete)
        time.sleep(1) 
        print("   (Capturando paquetes...)")
    print("[x] Sniffer detenido.")

# Creamos el hilo
hilo_sniffer = threading.Thread(target=sniffer_en_segundo_plano)

# start() lanza el sniffer y deja que el código de abajo siga corriendo
hilo_sniffer.start() 

print("[+] El programa principal sigue activo.")
for i in range(5):
    print(f"[+] Procesando datos del usuario... {i+1}")
    time.sleep(0.8)

print("[-] Deteniendo sniffer...")
continuar_escuchando = False
hilo_sniffer.join() # Esperamos a que limpie y cierre
print("Fin del programa.")
```

##### Ejemplo 3: El peligro de la concurrencia (Race Condition)

Mira lo que pasa cuando muchos hilos tocan una variable compartida sin protección.

```python
import threading

# Cuenta bancaria compartida
saldo = 0
candado = threading.Lock() # <-- La solución

def depositar_dinero():
    global saldo
    for _ in range(1000):
        # INICIO SECCIÓN CRÍTICA
        candado.acquire() # Cierra la puerta
        saldo += 1
        candado.release() # Abre la puerta
        # FIN SECCIÓN CRÍTICA
        
        # SIN CANDADO (candado.acquire/release comentados), 
        # el resultado final NO SERÍA 200,000. Sería aleatorio (ej: 154,320).

hilo1 = threading.Thread(target=depositar_dinero)
hilo2 = threading.Thread(target=depositar_dinero)

hilo1.start()
hilo2.start()

hilo1.join()
hilo2.join()

print(f"Saldo final esperado: 2000")
print(f"Saldo final real:     {saldo}")
```

### Diferencia Vital: `threading` vs `asyncio`

Esta es la pregunta del millón en entrevistas técnicas.

| Característica | `threading` (Hilos)                                                                | `asyncio` (Corutinas)                                            |
| :------------- | :--------------------------------------------------------------------------------- | :--------------------------------------------------------------- |
| **Modelo**     | **Preemptivo** (El Sistema Operativo decide cuándo cortar un hilo y pasar a otro). | **Cooperativo** (La tarea decide cuándo pausar usando `await`).  |
| **Bloqueo**    | Ideal para librerías que **bloquean** (`requests`, `time.sleep`, `scapy`).         | Ideal para librerías **no bloqueantes** (`httpx`, `aiohttp`).    |
| **Peso**       | Los hilos consumen más memoria RAM.                                                | Las corutinas son ultra ligeras (puedes tener miles).            |
| **Seguridad**  | Necesitas `Locks` para evitar errores de datos.                                    | Más seguro por defecto (tú controlas cuándo cambia el contexto). |

### Resumen de integración

Ahora tienes dos formas de hacer cosas a la vez:

1.  **Escenario A (Web Scraping masivo):** Tienes que bajar 1000 páginas.
    *   Usas **`asyncio` + `httpx`**. Es más eficiente.

2.  **Escenario B (Herramienta de Hacking/Redes):** Tienes que escanear puertos con `scapy` mientras mantienes una interfaz de usuario activa.
    *   Usas **`threading`**. Pones a `scapy` en un hilo aparte porque `scapy` no soporta `asyncio`.

