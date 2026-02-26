### Descripción General

Es un paquete autocontenido de primitivas criptográficas de bajo nivel.
Permite:

1.  **Encriptación Simétrica (AES):** Una sola clave para cerrar y abrir (muy rápido, ideal para archivos o bases de datos).
2.  **Encriptación Asimétrica (RSA):** Una clave pública para cerrar y una privada para abrir (ideal para compartir secretos).
3.  **Hashing (SHA):** Crear una huella digital única de un archivo para asegurar que no fue modificado.
4.  **Firmas Digitales:** Probar que un mensaje viene de ti.

**Nota Importante:**
*   **Instalación:** `pip install pycryptodome`
*   **Importación:** Aunque se instala como `pycryptodome`, en el código se importa como `Crypto`.
*   **Conflicto:** Si tienes instalada una librería vieja llamada `pycrypto`, desinstálala primero. Son incompatibles.

### Conceptos Clave

*   **Bytes, no Strings:** La criptografía trabaja con matemáticas sobre datos binarios. Siempre tendrás que usar `.encode()` antes de encriptar y `.decode()` después.
*   **Nonce / IV (Vector de Inicialización):** Es un número aleatorio que se usa junto con la clave para que, si encriptas dos veces "Hola", el resultado sea diferente cada vez.
*   **Modo (EAX, GCM, CBC):** Es la "fórmula" matemática usada. Recomendamos **EAX** o **GCM** porque son modernos y verifican la integridad.

### Ejemplos de Código

##### Ejemplo 1: Encriptación Simétrica (AES) - El estándar de oro

Ideal para guardar datos sensibles en tu base de datos `sqlite3`.
Necesitas una clave de 16, 24 o 32 bytes.

```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
import base64

# 1. Generar una clave (Guárdala en tu .env con python-dotenv!)
clave = get_random_bytes(16) # 16 bytes = 128 bits

datos = "Mi secreto super oscuro"

# --- ENCRIPTAR ---
def encriptar(msg, key):
    # Convertir texto a bytes
    data_bytes = msg.encode('utf-8')
    
    # Crear cifrador en modo EAX (Moderno y seguro)
    cipher = AES.new(key, AES.MODE_EAX)
    
    # Encriptar y obtener el "tag" (firma de seguridad)
    ciphertext, tag = cipher.encrypt_and_digest(data_bytes)
    
    # Necesitamos guardar el 'nonce' para poder desencriptar luego
    return cipher.nonce, ciphertext, tag

nonce, texto_cifrado, tag = encriptar(datos, clave)

print(f"Texto original: {datos}")
print(f"Encriptado (Bytes): {texto_cifrado}")
# Usamos base64 para verlo bonito
print(f"Encriptado (B64): {base64.b64encode(texto_cifrado).decode('utf-8')}")


# --- DESENCRIPTAR ---
def desencriptar(nonce, ciphertext, tag, key):
    cipher = AES.new(key, AES.MODE_EAX, nonce=nonce)
    try:
        data = cipher.decrypt_and_verify(ciphertext, tag)
        return data.decode('utf-8')
    except ValueError:
        return "¡Error! Clave incorrecta o datos corruptos."

recuperado = desencriptar(nonce, texto_cifrado, tag, clave)
print(f"Recuperado: {recuperado}")
```

##### Ejemplo 2: Hashing (SHA-256) - Integridad

Para verificar que un archivo no ha sido alterado o para guardar contraseñas (aunque para passwords es mejor `bcrypt`).

```python
from Crypto.Hash import SHA256

mensaje = "Contrato_Final_v2.pdf" # Imaginemos que es el contenido del archivo

# Crear el objeto hash
h = SHA256.new()
h.update(mensaje.encode('utf-8'))

# Obtener la huella digital (Digest) en hexadecimal
huella = h.hexdigest()

print(f"Huella SHA-256: {huella}")

# Si cambiamos una sola letra...
mensaje_falso = "Contrato_Final_v3.pdf"
h2 = SHA256.new(mensaje_falso.encode('utf-8'))
print(f"Huella Falsa:   {h2.hexdigest()}")

if h.hexdigest() != h2.hexdigest():
    print("¡ALERTA! El archivo ha sido modificado.")
```

##### Ejemplo 3: Encriptación Asimétrica (RSA)

Aquí generamos un par de llaves. Lo que encriptas con la Pública, solo lo abre la Privada.

```python
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_OAEP

# 1. Generar par de claves (Pública y Privada)
key = RSA.generate(2048)
private_key = key.export_key()
public_key = key.publickey().export_key()

# --- ENCRIPTAR (Alguien te quiere mandar un secreto) ---
# Usan TU llave pública
recipient_key = RSA.import_key(public_key)
cipher_rsa = PKCS1_OAEP.new(recipient_key)

secreto = "La reunión es a medianoche"
encriptado = cipher_rsa.encrypt(secreto.encode('utf-8'))

print(f"Mensaje encriptado (oculto): {encriptado[:20]}...")

# --- DESENCRIPTAR (Solo tú puedes leerlo) ---
# Usas TU llave privada
my_private_key = RSA.import_key(private_key)
cipher_rsa = PKCS1_OAEP.new(my_private_key)

desencriptado = cipher_rsa.decrypt(encriptado)
print(f"Mensaje leído: {desencriptado.decode('utf-8')}")
```

### Integración con Stack (El Sistema Seguro)

Aquí es donde juntamos todo para hacer una caja fuerte blindada:

1.  **`python-dotenv`**: Guardas la `AES_KEY` (clave maestra) en el archivo `.env`. Nunca en el código.
2.  **`pydantic`**: Recibes los datos del usuario (ej: número de tarjeta de crédito) y validas que sean números.
3.  **`pycryptodome`**:
    *   Tomas ese número de tarjeta.
    *   Lo encriptas con AES usando la clave del `.env`.
    *   Obtienes bytes encriptados.
4.  **`base64`**: Conviertes esos bytes encriptados a un string base64 para que no rompa la base de datos.
5.  **`sqlmodel` / `sqlite3`**: Guardas ese string base64 en la base de datos.
    *   *Si un hacker roba el archivo `database.db`, solo verá basura ininteligible.*
6.  **`logging`**: Registras "Tarjeta guardada exitosamente" (sin guardar el número, obvio).

### Advertencias de Seguridad 

1.  **Gestión de Claves:** Si pierdes la clave (`AES_KEY` o `private_key`), **pierdes los datos para siempre**. No hay "recuperar contraseña" en criptografía.
2.  **No inventes:** No trates de crear tu propio algoritmo de encriptación. Usa siempre AES (Simétrico) o RSA/ECC (Asimétrico).
3.  **Aleatoriedad:** Usa siempre `get_random_bytes` de esta librería, no uses `random.randint` de Python estándar para generar claves (no es criptográficamente seguro).

