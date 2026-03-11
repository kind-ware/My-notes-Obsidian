## Descripción General

`ring` es una librería de criptografía enfocada en la **seguridad y el rendimiento**. Su filosofía es "menos es más": no intenta soportar todos los algoritmos existentes (especialmente los antiguos o inseguros como MD5 o SHA-1), sino que ofrece implementaciones impecables de los algoritmos modernos más robustos.

Es la base de proyectos críticos como `rustls` (la alternativa a OpenSSL en Rust). Gran parte de su código de bajo nivel está optimizado en ensamblador para diversas arquitecturas (x86_64, ARM, etc.), lo que la hace increíblemente eficiente.

### Casos de Uso

*   **Hasing de Datos:** Generar huellas digitales de archivos o mensajes (SHA-256, SHA-512).
*   **Cifrado Autenticado (AEAD):** Encriptar datos de forma que no solo sean privados, sino que también se detecte si han sido manipulados (AES-GCM, ChaCha20-Poly1305).
*   **Firmas Digitales:** Autenticar la autoría de un mensaje (Ed25519, RSA).
*   **Acuerdo de Claves:** Establecer una clave compartida secreta sobre un canal inseguro (ECDH, X25519).
*   **Generación de Números Aleatorios:** Obtener entropía segura para claves criptográficas.

## Módulos y Funciones Principales

*   **`ring::digest`**: Funciones para hashing (SHA-2, SHA-3).
*   **`ring::aead`**: Cifrado simétrico de alta seguridad (Authenticated Encryption with Associated Data).
*   **`ring::signature`**: Verificación y creación de firmas digitales (Ed25519, P-256, RSA).
*   **`ring::hmac`**: Códigos de autenticación de mensajes basados en hash.
*   **`ring::rand`**: Generador de números aleatorios criptográficamente seguros (`SystemRandom`).
*   **`ring::agreement`**: Intercambio de claves (Diffie-Hellman).

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
ring = "0.17"
```

### Ejemplo 1: Generar un Hash SHA-256
Es la operación más básica: obtener un resumen único de una entrada de datos.

```rust
use ring::digest;

fn main() {
    let mensaje = "Hola, este es un mensaje secreto";
    
    // Calcular el digest (hash)
    let actual_hash = digest::digest(&digest::SHA256, mensaje.as_bytes());

    println!("Hash (HEX): {:?}", hex::encode(actual_hash.as_ref()));
}

// Nota: Para usar hex::encode necesitas la crate 'hex'
```

### Ejemplo 2: Firmar y Verificar con Ed25519
Ed25519 es uno de los algoritmos de firma más rápidos y seguros de la actualidad.

```rust
use ring::{
    rand,
    signature::{self, KeyPair},
};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let rng = rand::SystemRandom::new();

    // 1. Generar un par de claves (Pública/Privada) en formato PKCS8
    let pkcs8_bytes = signature::Ed25519KeyPair::generate_pkcs8(&rng)?;
    let key_pair = signature::Ed25519KeyPair::from_pkcs8(pkcs8_bytes.as_ref())?;

    // 2. Firmar un mensaje
    let mensaje = b"Documento de identidad";
    let firma = key_pair.sign(mensaje);

    // 3. Verificar la firma con la clave pública
    let peer_public_key_bytes = key_pair.public_key().as_ref();
    let public_key = signature::UnparsedPublicKey::new(&signature::ED25519, peer_public_key_bytes);

    public_key.verify(mensaje, firma.as_ref())
        .map_err(|_| "La firma no es válida")?;

    println!("¡Firma verificada con éxito!");
    Ok(())
}
```

### Ejemplo 3: Cifrado Simétrico con ChaCha20-Poly1305
Este es un ejemplo de cifrado AEAD, que garantiza privacidad e integridad.

```rust
use ring::aead::{self, BoundKey, Nonce, SealingKey, UnboundKey, Aad, NONCE_LEN};
use ring::rand::{SystemRandom, SecureRandom};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let rng = SystemRandom::new();

    // 1. Generar una clave aleatoria
    let mut key_bytes = [0u8; 32];
    rng.fill(&mut key_bytes)?;
    let unbound_key = UnboundKey::new(&aead::CHACHA20_POLY1305, &key_bytes)?;

    // 2. Crear un Nonce (Número único usado una sola vez)
    let mut nonce_bytes = [0u8; NONCE_LEN];
    rng.fill(&mut nonce_bytes)?;
    let nonce = Nonce::assume_unique_for_key(nonce_bytes);

    // 3. Preparar datos y encriptar (el buffer debe tener espacio para el tag de autenticación)
    let mut data = String::from("Datos súper secretos").into_bytes();
    let mut sealing_key = SealingKey::new(unbound_key, nonce);
    
    // Seala (encripta) los datos "in-place"
    sealing_key.seal_in_place_append_tag(Aad::empty(), &mut data)?;

    println!("Datos cifrados (con tag): {:?}", data);
    Ok(())
}
```

## Buenas Prácticas y Consideraciones

1.  **Manejo de Errores Intencional**: `ring` devuelve un tipo de error genérico llamado `Unspecified`. Esto es **por diseño**. Dar detalles específicos de por qué falló una operación criptográfica (ej. "el padding es incorrecto") puede ayudar a atacantes en ataques de oráculo o de tiempo.
2.  **No reutilices Nonces**: En algoritmos como AES-GCM o ChaCha20, usar el mismo *nonce* con la misma clave para dos mensajes diferentes rompe totalmente la seguridad. Usa `SystemRandom` para generar nonces únicos.
3.  **In-place Operations**: Para maximizar la velocidad, `ring` suele trabajar "in-place" (sobre el mismo buffer de datos). Asegúrate de que tus vectores tengan la capacidad suficiente para añadir el "tag" de autenticación al final (normalmente 16 bytes extra).
4.  **Usa Algoritmos Modernos**: Si no tienes una restricción técnica, prefiere **Ed25519** sobre RSA para firmas y **ChaCha20-Poly1305** sobre AES-GCM si el procesador no tiene aceleración por hardware para AES.
5.  **Compilación**: `ring` requiere un compilador de C y, en ocasiones, Perl para compilar los archivos de ensamblador durante el proceso de build. Asegúrate de tener las herramientas de construcción instaladas en tu entorno.