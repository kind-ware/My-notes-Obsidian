### HTTPX

`httpx` es un cliente HTTP de última generación para Python.
Lo que lo hace especial es su **dualidad**:

1.  **Modo Síncrono:** Funciona igual que `requests` (paso a paso).
2.  **Modo Asíncrono:** Funciona perfectamente con `asyncio` (concurrente).

Es la herramienta estándar hoy en día si quieres construir aplicaciones rápidas que consuman APIs.

### Los Dos Modos de Uso

##### Modo "Requests" (Síncrono)

Si no quieres usar `asyncio`, puedes usar `httpx` exactamente igual que `requests`.

| Función           | Descripción                                                       |
| :---------------- | :---------------------------------------------------------------- |
| `httpx.get(url)`  | Petición GET estándar. Bloquea el script hasta recibir respuesta. |
| `httpx.post(url)` | Petición POST estándar.                                           |

##### Modo "Async" (Asíncrono)

Aquí es donde brilla. Para usarlo de forma asíncrona, se recomienda usar un **Cliente** (una sesión).

| Clase / Método          | Descripción                                                                                |
| :---------------------- | :----------------------------------------------------------------------------------------- |
| `httpx.AsyncClient()`   | Crea una sesión asíncrona (como abrir un navegador). Se usa con `async with`.              |
| `await client.get(url)` | Petición que **no bloquea**. Permite que otras tareas corran mientras espera la respuesta. |

### Ejemplos de Código

##### Ejemplo 1: El uso Básico (Igual a `requests`)

Si vienes de `requests`, esto es idéntico.

```python
import httpx

url = "https://jsonplaceholder.typicode.com/posts/1"

# Modo Síncrono (sin async/await)
respuesta = httpx.get(url)

print(f"Status: {respuesta.status_code}")
print(f"Titulo: {respuesta.json()['title']}")
```

##### Ejemplo 2: El uso Asíncrono (La forma correcta)

Aquí usamos `async with`. Esto abre y cierra la conexión de forma eficiente.

```python
import httpx
import asyncio

async def obtener_usuario():
    url = "https://jsonplaceholder.typicode.com/users/1"
    
    # Abrimos un cliente asíncrono
    async with httpx.AsyncClient() as client:
        print("Enviando petición...")
        # Usamos await para esperar la respuesta sin bloquear
        response = await client.get(url)
        
        datos = response.json()
        print(f"Recibido: {datos['name']} ({datos['email']})")

if __name__ == "__main__":
    asyncio.run(obtener_usuario())
```

##### Ejemplo 3: Velocidad Pura (Peticiones Concurrentes)

Este es el ejemplo definitivo. Vamos a pedir datos a 3 URLs **al mismo tiempo**.
Combinamos `httpx` con `asyncio.gather` (que aprendiste en la lección anterior).

```python
import httpx
import asyncio
import time

async def descargar_pokemon(client, id_pokemon):
    url = f"https://pokeapi.co/api/v2/pokemon/{id_pokemon}"
    print(f"[-] Pidiendo datos de Pokémon #{id_pokemon}...")
    
    response = await client.get(url)
    datos = response.json()
    
    print(f"[+] Llegó: {datos['name'].capitalize()}")
    return datos['name']

async def main():
    inicio = time.time()
    
    # Creamos UNA sola sesión para todas las peticiones (muy eficiente)
    async with httpx.AsyncClient() as client:
        # Preparamos las tareas, pero no las lanzamos aún
        tareas = [
            descargar_pokemon(client, 1),  # Bulbasaur
            descargar_pokemon(client, 4),  # Charmander
            descargar_pokemon(client, 7)   # Squirtle
        ]
        
        # ¡FUEGO! Lanzamos las 3 a la vez
        nombres = await asyncio.gather(*tareas)
    
    fin = time.time()
    print(f"\nResumen: {nombres}")
    print(f"Tiempo total: {fin - inicio:.2f} segundos") 
    # Notarás que tarda lo que tarda el más lento, no la suma de los tres.

if __name__ == "__main__":
    asyncio.run(main())
```

##### Ejemplo 4: Manejo de Errores y Timeouts

A diferencia de `requests`, `httpx` tiene un **timeout por defecto** de 5 segundos (por seguridad). `requests` espera infinitamente si no le dices nada.

```python
import httpx
import asyncio

async def probar_timeout():
    # URL que fuerza una demora de 3 segundos
    url = "https://httpbin.org/delay/3" 
    
    async with httpx.AsyncClient() as client:
        try:
            # Configuramos un timeout estricto de 1 segundo
            # Como la URL tarda 3s, esto fallará.
            response = await client.get(url, timeout=1.0)
            print("Éxito")
            
        except httpx.TimeoutException:
            print("¡Error! La petición tardó demasiado y se canceló.")
        except httpx.RequestError as e:
            print(f"Ocurrió un error de red: {e}")

if __name__ == "__main__":
    asyncio.run(probar_timeout())
```

### Comparativa: `requests` vs `httpx`

| Característica            | `requests`                  | `httpx`                         |
| :------------------------ | :-------------------------- | :------------------------------ |
| **Sintaxis**              | `requests.get()`            | `httpx.get()` (Idéntica)        |
| **Asíncrono (`asyncio`)** | ❌ No (Bloqueante)           | ✅ Sí (Nativo)                   |
| **HTTP/2**                | ❌ No (Solo HTTP/1.1)        | ✅ Sí (Más rápido y moderno)     |
| **Timeouts**              | ❌ No tiene (peligroso)      | ✅ Sí (5 segundos por defecto)   |
| **Popularidad**           | Masiva (Estándar histórico) | Creciente (El estándar moderno) |

### Cuándo usar cuál

1.  **Usa `requests` si:**
    *   Estás haciendo un script simple y rápido.
    *   Estás aprendiendo y no quieres lidiar con `async/await`.
    *   Es un proyecto antiguo.

2.  **Usa `httpx` si:**
    *   Estás usando `asyncio` (ej: Bots de Discord, Telegram, Web Scraping masivo, FastAPI).
    *   Necesitas rendimiento (descargar 100 archivos a la vez).
    *   Quieres aprovechar HTTP/2.
