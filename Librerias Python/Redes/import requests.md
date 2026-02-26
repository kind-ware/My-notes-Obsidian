### Requests 

`requests` está diseñada para hacer peticiones **HTTP** (navegar por internet) de la forma más sencilla y humana posible.
Se utiliza para:
*   Consumir **APIs** (obtener datos del clima, criptomonedas, usuarios).
*   Descargar archivos (imágenes, PDFs).
*   Enviar formularios (Login, subir datos).
*   Automatizar interacciones con sitios web (Scraping básico).

### Funciones Principales (Verbos HTTP)

En internet, las acciones se definen por "verbos". `requests` tiene una función para cada uno.

| Función | Descripción | Equivalente |
| :--- | :--- | :--- |
| `requests.get(url)` | **Solicita** datos a un servidor. Es como escribir una dirección en el navegador. | "Dame esto" |
| `requests.post(url, data)` | **Envía** nuevos datos al servidor (ej: crear un usuario). | "Toma esto" |
| `requests.put(url, data)` | **Actualiza** datos existentes. | "Cambia esto" |
| `requests.delete(url)` | **Borra** datos en el servidor. | "Elimina esto" |

##### El Objeto `Response` (La Respuesta)
Cuando haces una petición, `requests` te devuelve un objeto (generalmente llamado `response` o `r`) que contiene todo lo que pasó.

| Atributo / Método | Descripción |
| :--- | :--- |
| `r.status_code` | El número que dice cómo salió todo (200=OK, 404=No encontrado, 500=Error Servidor). |
| `r.text` | El contenido de la respuesta en formato **Texto** (String). |
| `r.json()` | Método mágico que convierte el texto JSON directamente a **Diccionario Python**. |
| `r.content` | El contenido en **Bytes** (Binario). Vital para descargar imágenes o PDFs. |

### Ejemplos de Código

Usaremos una API de prueba gratuita llamada `jsonplaceholder.typicode.com` para que los ejemplos sean reales y funcionen.

##### Ejemplo 1: Petición Básica (GET)

Obtener información de una página web o API.

```python
import requests

url = "https://jsonplaceholder.typicode.com/posts/1"

# 1. Hacemos la petición
respuesta = requests.get(url)

# 2. Verificamos si funcionó (Código 200 significa Éxito)
if respuesta.status_code == 200:
    print("¡Conexión exitosa!")
    
    # 3. Leemos el contenido
    # .text nos da la cadena cruda
    print("Contenido (Texto):", respuesta.text)
else:
    print(f"Error: {respuesta.status_code}")
```

##### Ejemplo 2: Trabajando con JSON (Lo más común)

Aquí combinamos lo que aprendiste de `json` con `requests`.

```python
import requests

# URL que devuelve una lista de usuarios ficticios
url = "https://jsonplaceholder.typicode.com/users"

response = requests.get(url)

if response.status_code == 200:
    # .json() hace el trabajo sucio de json.loads() automáticamente
    datos = response.json() 
    
    # Como 'datos' ya es una lista de diccionarios, podemos iterar
    print(f"Se encontraron {len(datos)} usuarios:")
    
    for usuario in datos[:3]: # Solo los primeros 3
        print(f"- Nombre: {usuario['name']} | Email: {usuario['email']}")
```

##### Ejemplo 3: Enviar Datos (POST)

Simulamos que estamos creando un nuevo post en un blog.

```python
import requests

url = "https://jsonplaceholder.typicode.com/posts"

# Datos que queremos enviar (Diccionario Python)
nuevo_post = {
    "title": "Aprendiendo Python",
    "body": "Requests hace que HTTP sea fácil.",
    "userId": 1
}

# Usamos el parámetro 'json=' para que requests lo formatee automáticamente
respuesta = requests.post(url, json=nuevo_post)

print(f"Código de estado: {respuesta.status_code}") 
# Nota: En muchas APIs, '201' significa 'Creado exitosamente'

print("Respuesta del servidor:", respuesta.json())
```

##### Ejemplo 4: Pasar parámetros (Query Params)

Cuando ves URLs así: `busqueda?q=python&page=2`. `requests` lo hace elegante.

```python
import requests

url = "https://jsonplaceholder.typicode.com/comments"

# En lugar de escribir la URL larga, usamos un diccionario
parametros = {
    "postId": 1  # Solo queremos los comentarios del post #1
}

# requests construye la URL final automáticamente
r = requests.get(url, params=parametros)

print(f"URL generada: {r.url}") # https://.../comments?postId=1
print(f"Comentarios encontrados: {len(r.json())}")
```

##### Ejemplo 5: Descargar una Imagen (Binario)

Aquí usamos `r.content` y lo combinamos con lo que sabes de manejo de archivos.

```python
import requests

url_imagen = "https://via.placeholder.com/150"
nombre_archivo = "imagen_descargada.png"

print("Descargando imagen...")
r = requests.get(url_imagen)

if r.status_code == 200:
    # Abrimos en modo 'wb' (Write Binary)
    with open(nombre_archivo, 'wb') as f:
        f.write(r.content)
    print(f"Imagen guardada como {nombre_archivo}")
else:
    print("No se pudo descargar la imagen.")
```

##### Ejemplo 6: Manejo de Errores Profesional

A veces el internet falla. `requests` tiene una forma rápida de lanzar alertas.

```python
import requests

url = "https://pagina-que-no-existe-123.com"

try:
    r = requests.get(url, timeout=5) # Si tarda más de 5 seg, corta
    
    # Esta línea lanza un error SI el status code es 4XX o 5XX
    r.raise_for_status()
    
    print("Todo salió bien")

except requests.exceptions.HTTPError as err:
    print(f"Error HTTP (404, 500, etc): {err}")
except requests.exceptions.ConnectionError:
    print("Error de Conexión: ¿Tienes internet?")
except requests.exceptions.Timeout:
    print("El servidor tardó demasiado en responder.")
```

### Diferencia Clave: `requests` vs `socket`

Esta comparación es vital para entender tu progreso:

1.  **`socket`**: Tuviste que definir IPv4, TCP, conectar, codificar a bytes, recibir paquetes de 1024 bytes y decodificar. **Es manual.**
2.  **`requests`**: Solo dijiste `get(url)` y obtuviste el resultado. **Es automático.**

*   Usa `socket` si estás creando un juego multijugador o un protocolo nuevo.
*   Usa `requests` para el 99% de las tareas web (APIs, Scraping, descargas).
