### Sockets

El módulo `socket` proporciona una interfaz de bajo nivel para la comunicación por red. Permite crear **puntos finales** (endpoints) de comunicación para enviar y recibir datos entre computadoras (o procesos en la misma computadora).

Imagina que un socket es como un **teléfono**:
*   Necesitas uno para llamar (Cliente).
*   Necesitas uno para recibir (Servidor).
*   Necesitas un número (Dirección IP) y una extensión (Puerto).

##### Conceptos Clave (Antes del código)

Para usar esta librería, necesitas entender 3 parámetros que verás siempre:

1.  **Familia (Address Family):**
    *   `socket.AF_INET`: Usar direcciones IPv4 (ej: `192.168.1.5`). **Es el estándar.**
    *   `socket.AF_INET6`: Usar direcciones IPv6.
2.  **Tipo (Socket Type):**
    *   `socket.SOCK_STREAM`: Usar protocolo **TCP**. Es seguro, garantiza que los datos llegan en orden y sin errores (usado en Web, Email, Chat).
    *   `socket.SOCK_DGRAM`: Usar protocolo **UDP**. Es rápido pero no garantiza la entrega (usado en Streaming de video, Juegos online).
3.  **Puerto:** Un número del 1 al 65535 que identifica a tu programa dentro de la PC (ej: Web es 80, MySQL es 3306).

### Funciones Principales

Las funciones cambian dependiendo de si actúas como **Servidor** (el que espera) o **Cliente** (el que llama).

##### Configuración General

| Función                        | Descripción                                                             |
| :----------------------------- | :---------------------------------------------------------------------- |
| `socket.socket(familia, tipo)` | Crea el objeto socket (el "teléfono").                                  |
| `socket.gethostname()`         | Devuelve el nombre de tu computadora.                                   |
| `socket.gethostbyname(host)`   | Obtiene la IP a partir de un nombre (ej: "google.com" -> "142.250..."). |

##### Para el Servidor (El que escucha)

| Función | Descripción |
| :--- | :--- |
| `bind((host, puerto))` | **Vincula** el socket a una dirección y puerto específicos ("Me sentaré a esperar en esta dirección"). |
| `listen()` | Empieza a **escuchar** conexiones entrantes. |
| `accept()` | **Acepta** una llamada. Detiene el código hasta que alguien se conecta. Devuelve un nuevo socket para hablar con ese cliente. |

##### Para el Cliente (El que conecta)

| Función                   | Descripción                              |
| :------------------------ | :--------------------------------------- |
| `connect((host, puerto))` | Intenta conectarse a un servidor activo. |

##### Comunicación (Ambos)

| Función        | Descripción                                                         |
| :------------- | :------------------------------------------------------------------ |
| `send(bytes)`  | Envía datos. ¡Ojo! Deben ser **bytes**, no texto (usa `.encode()`). |
| `recv(buffer)` | Recibe datos. Debes indicar cuántos bytes máximos leer (ej: 1024).  |
| `close()`      | Cuelga la llamada y libera el puerto.                               |

### Ejemplos de Código

##### Ejemplo 1: Información básica de Red

Antes de conectar nada, veamos quiénes somos.

```python
import socket

nombre_host = socket.gethostname()
ip_local = socket.gethostbyname(nombre_host)

print(f"Nombre de mi PC: {nombre_host}")
print(f"Mi IP local: {ip_local}")

# Obtener IP de una web
ip_google = socket.gethostbyname("google.com")
print(f"La IP de Google es: {ip_google}")
```

##### Ejemplo 2: Chat Básico (Cliente - Servidor)

Este ejemplo requiere dos scripts separados. Uno debe correr primero (Servidor) y luego el otro (Cliente).

**SCRIPT A: El Servidor (`server.py`)**
*Ejecuta esto en una terminal y déjalo corriendo.*

```python
import socket

# 1. Crear el socket (IPv4, TCP)
servidor = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 2. Bind: Asociar a una dirección y puerto
# 'localhost' significa que solo aceptamos conexiones de la misma PC
# Puerto 12345 (usa uno alto, >1024)
servidor.bind(('localhost', 12345))

# 3. Listen: Empezar a escuchar (cola de espera de 1)
servidor.listen(1)
print("Esperando conexión...")

# 4. Accept: Se bloquea aquí hasta que alguien entre
conn, addr = servidor.accept()
print(f"Conectado con: {addr}")

while True:
    # 5. Recibir datos (máximo 1024 bytes)
    datos = conn.recv(1024)
    if not datos: break # Si no hay datos, el cliente desconectó
    
    mensaje = datos.decode('utf-8')
    print(f"Cliente dice: {mensaje}")
    
    # 6. Responder
    conn.send("Recibido, cambio.".encode('utf-8'))

conn.close()
```

**SCRIPT B: El Cliente (`client.py`)**
*Ejecuta esto en otra terminal.*

```python
import socket

# 1. Crear socket
cliente = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 2. Conectar al servidor
cliente.connect(('localhost', 12345))

# 3. Enviar mensaje (Convertir string a bytes con encode)
mensaje = "Hola Servidor, soy el Cliente"
cliente.send(mensaje.encode('utf-8'))

# 4. Recibir respuesta
respuesta = cliente.recv(1024)
print(f"Servidor respondió: {respuesta.decode('utf-8')}")

cliente.close()
```

##### Ejemplo 3: Escáner de Puertos simple

Una herramienta clásica de seguridad. Verifica si un puerto está abierto en una máquina.

```python
import socket

objetivo = "google.com"
puerto = 80 # Puerto Web HTTP

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.settimeout(2) # Si tarda más de 2 segundos, cancelamos

# connect_ex devuelve 0 si tuvo éxito, o un error si falló
resultado = s.connect_ex((objetivo, puerto))

if resultado == 0:
    print(f"El puerto {puerto} en {objetivo} está ABIERTO.")
else:
    print(f"El puerto {puerto} está CERRADO o filtrado.")

s.close()
```

### Diferencias Clave: `socket` vs `requests`

Es probable que escuches sobre la librería `requests` (muy popular). ¿Cuál es la diferencia?

1.  **`socket`**:
    *   Es **bajo nivel**.
    *   Tú construyes los paquetes, manejas la conexión, decides los bytes.
    *   Sirve para cualquier protocolo (Chat, Email, FTP, Juegos, HTTP).
    *   *Analogía:* Construir tu propio walkie-talkie.
2.  **`requests` (o `urllib`)**:
    *   Es **alto nivel** (específica para HTTP/Web).
    *   Ya sabe cómo navegar webs, bajar HTML o JSON.
    *   Por debajo usa `socket`, pero te facilita la vida.
    *   *Analogía:* Usar un navegador web.