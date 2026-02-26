### Base64

El módulo `base64` provee funciones para codificar datos binarios a caracteres ASCII imprimibles (y viceversa).

**¿Por qué existe?**
Los protocolos antiguos (como el Email) y los formatos de texto (como JSON o XML) se rompen si intentas meterles "bytes crudos" de una imagen.
**Base64** toma esos bytes y los traduce a un alfabeto seguro de 64 caracteres (A-Z, a-z, 0-9, `+`, `/`).

**¡OJO! No es encriptación.**
Cualquiera puede decodificar Base64. No sirve para ocultar secretos, solo para **transportar datos**.

### Funciones Principales

Todas estas funciones trabajan con **bytes** (`b'...'`), no con strings normales.

| Función                            | Descripción                                                                               |
| :--------------------------------- | :---------------------------------------------------------------------------------------- |
| `base64.b64encode(bytes)`          | Convierte datos binarios a Base64.                                                        |
| `base64.b64decode(bytes)`          | Convierte Base64 de vuelta a datos binarios originales.                                   |
| `base64.urlsafe_b64encode(bytes)`  | Versión especial para URLs. Reemplaza `+` y `/` por `-` y `_` para no romper enlaces web. |
| `base64.standard_b64encode(bytes)` | La versión estándar (usa `+` y `/`).                                                      |

### Ejemplos de Código

##### Ejemplo 1: Codificación Básica de Texto

Para usar Base64, primero debes convertir tu texto a bytes.

```python
import base64

mensaje = "Hola Mundo, esto es secreto"

# 1. Convertir String -> Bytes
bytes_mensaje = mensaje.encode('utf-8')

# 2. Codificar a Base64
# El resultado sigue siendo bytes (b'SG9sY...')
codificado_bytes = base64.b64encode(bytes_mensaje)

# 3. Convertir Bytes Base64 -> String (Para imprimir o enviar en JSON)
codificado_str = codificado_bytes.decode('utf-8')

print(f"Original: {mensaje}")
print(f"Base64:   {codificado_str}") 
# Salida esperada: SG9sYSBNdW5kbywgZXN0byBlcyBzZWNyZXRv
```

##### Ejemplo 2: Decodificar (Recuperar el original)

Imagina que recibes ese código raro y quieres saber qué dice.

```python
import base64

codigo_recibido = "U2VjcmV0byBkZSBQeXRob24="

# 1. Decodificar (De Base64 -> Bytes originales)
datos_bytes = base64.b64decode(codigo_recibido)

# 2. Convertir a texto legible
texto_final = datos_bytes.decode('utf-8')

print(f"El mensaje oculto era: {texto_final}")
```

##### Ejemplo 3: Manejo de Imágenes (El uso más común)

Este es el caso de uso real. Tienes una imagen y quieres enviarla en un JSON a una API.

```python
import base64
import os

# Simulamos que tenemos una imagen (creamos un archivo binario dummy)
# En la vida real, esto sería 'foto.png'
with open("imagen_prueba.bin", "wb") as f:
    f.write(b'\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR...')

# --- CODIFICAR IMAGEN PARA ENVIAR ---
with open("imagen_prueba.bin", "rb") as f:
    contenido = f.read()
    
    # Codificamos
    b64_bytes = base64.b64encode(contenido)
    b64_string = b64_bytes.decode('utf-8')

# Así es como se manda en un JSON a una API
payload_api = {
    "nombre": "foto_perfil.png",
    "data": b64_string  # ¡La imagen ahora es texto!
}

print("Payload JSON listo para enviar:")
print(payload_api)

# --- DECODIFICAR (Guardar la imagen recibida) ---
# Supongamos que recibimos el JSON
imagen_recuperada = base64.b64decode(payload_api["data"])

with open("imagen_recuperada.bin", "wb") as f:
    f.write(imagen_recuperada)

print("Imagen recuperada y guardada en disco.")
```

##### Ejemplo 4: URLs Seguras (`urlsafe_b64encode`)

El Base64 estándar usa el símbolo `+` y `/`.
*   El `/` rompe las rutas de URL (`tusitio.com/usuario/codigo/`).
*   El `+` se interpreta como espacio en algunas URLs.

Para evitar esto, usamos la versión `urlsafe`.

```python
import base64

data = b'\xfb\xff\xff'  # Unos bytes binarios raros

# Estándar (Puede contener + y /)
std = base64.standard_b64encode(data)
print(f"Estándar: {std}") 
# Salida podría ser b'+///' (Rompería una URL)

# URL Safe (Usa - y _)
safe = base64.urlsafe_b64encode(data)
print(f"URL Safe: {safe}")
# Salida sería b'-___' (Seguro para poner en barra de direcciones)
```

### Integración con Stack

Mira cómo `base64` se une a lo que ya sabes:

1.  **Con `requests` / `httpx`**:
    *   **Autenticación Basic:** Muchas APIs viejas piden usuario y contraseña. Estos se envían en el header así: `Authorization: Basic <base64(user:pass)>`.
    *   Aunque `requests` lo hace automático con `auth=('user', 'pass')`, internamente usa `base64`.

2.  **Con `FastAPI`**:
    *   Si haces una API que recibe archivos pequeños, a veces es más fácil recibir un JSON con un string Base64 que lidiar con `multipart/form-data`.

3.  **Con `sqlite3` / `SQLModel`**:
    *   Si quieres guardar una imagen pequeña (ej: thumbnail) directamente en la base de datos, puedes convertirla a Base64 y guardarla en un campo de texto (`str`), aunque lo ideal es guardar solo la ruta del archivo.

4.  **Con `cryptography` (Futuro)**:
    *   Las claves de encriptación y los hashes de contraseñas casi siempre se almacenan codificados en Base64 para que sean legibles.
