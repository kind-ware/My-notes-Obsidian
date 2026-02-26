## Descripción General

Este namespace proporciona servicios criptográficos que incluyen la protección de datos mediante cifrado y descifrado, así como el hashing, la generación de números aleatorios seguros y la firma digital. En .NET moderno, esta librería sirve como una capa de abstracción sobre las librerías nativas del sistema operativo (como CNG en Windows o OpenSSL en Linux/macOS), lo que la hace increíblemente rápida y segura.

#### Casos de Uso Comunes

*   **Hashing de Datos:** Crear "huellas digitales" de archivos o textos para verificar que no han sido alterados.
*   **Cifrado Simétrico:** Proteger archivos o mensajes usando una clave compartida (ej. AES).
*   **Cifrado Asimétrico:** Uso de llaves públicas y privadas para intercambio de información segura (ej. RSA).
*   **Generación de Tokens:** Crear identificadores o contraseñas temporales que son imposibles de predecir.
*   **Firmas Digitales:** Autenticar que un mensaje realmente proviene de quien dice ser.

## Clases y Métodos Principales

La librería se organiza principalmente en tres familias de algoritmos:

*   **Hashing (Resumen):**
    *   `SHA256`, `SHA512`: Algoritmos estándar para crear hashes seguros.
    *   `HMACSHA256`: Hash con clave (útil para firmar tokens JWT).
*   **Cifrado Simétrico (Misma clave para cifrar/descifrar):**
    *   `Aes`: El estándar de oro actual (Advanced Encryption Standard).
*   **Cifrado Asimétrico (Llave pública/privada):**
    *   `RSA`: Utilizado para firmas y protección de pequeñas cantidades de datos.
*   **Utilidades:**
    *   `RandomNumberGenerator`: Para crear valores aleatorios con seguridad criptográfica (mucho mejor que `System.Random`).
    *   `RFC2898DeriveBytes`: Implementación de **PBKDF2**, ideal para derivar claves a partir de contraseñas de usuario.

## Ejemplos Prácticos

#### Ejemplo 1: Crear un Hash SHA256 (Integridad de datos)

```csharp
using System;
using System.Security.Cryptography;
using System.Text;

class Program
{
    static void Main()
    {
        string mensaje = "Hola Mundo";
        
        using (SHA256 sha256Hash = SHA256.Create())
        {
            // Convertimos el texto a bytes y calculamos el hash
            byte[] bytes = sha256Hash.ComputeHash(Encoding.UTF8.GetBytes(mensaje));

            // Convertimos los bytes a una cadena hexadecimal
            StringBuilder builder = new StringBuilder();
            foreach (byte b in bytes) builder.Append(b.ToString("x2"));
            
            Console.WriteLine($"Hash de '{mensaje}': {builder.ToString()}");
        }
    }
}
```

#### Ejemplo 2: Cifrado Simétrico con AES

*Nota: En un caso real, la clave y el IV (Vector de Inicialización) deben manejarse con extremo cuidado.*

```csharp
using System.IO;
using System.Security.Cryptography;

public static byte[] CifrarTexto(string textoPlano, byte[] clave, byte[] iv)
{
    using (Aes aes = Aes.Create())
    {
        aes.Key = clave;
        aes.IV = iv;

        ICryptoTransform cifrador = aes.CreateEncryptor(aes.Key, aes.IV);

        using (MemoryStream ms = new MemoryStream())
        {
            using (CryptoStream cs = new CryptoStream(ms, cifrador, CryptoStreamMode.Write))
            {
                using (StreamWriter sw = new StreamWriter(cs))
                {
                    sw.Write(textoPlano);
                }
                return ms.ToArray();
            }
        }
    }
}
```

#### Ejemplo 3: Generación de un Token Aleatorio Seguro

```csharp
using System.Security.Cryptography;

// Genera 32 bytes aleatorios seguros (apropiado para tokens o sales)
byte[] buffer = new byte[32];
RandomNumberGenerator.Fill(buffer);
string token = Convert.ToBase64String(buffer);
```

## Buenas Prácticas y Consideraciones

1.  **NO inventes tu propia criptografía:** Este es el mandamiento #1. Usa siempre las implementaciones estándar de la librería.
2.  **Evita algoritmos obsoletos:** No uses `MD5`, `SHA1`, `DES` o `TripleDES` para seguridad. Son vulnerables a ataques modernos. Usa al menos **SHA256** para hashes y **AES** para cifrado.
3.  **Usa `using` para liberar recursos:** Las clases de criptografía suelen manejar recursos nativos del sistema operativo. Siempre envuélvelas en un bloque `using` para asegurar que las llaves se borren de la memoria lo antes posible.
4.  **Sales (Salts) e Iteraciones:** Si vas a guardar contraseñas, nunca uses solo un hash. Usa `PBKDF2` (vía `RFC2898DeriveBytes`) con una "sal" aleatoria y al menos 100,000 iteraciones para hacer que los ataques de fuerza bruta sean lentos.
5.  **IV (Vector de Inicialización):** En el cifrado AES, nunca uses el mismo IV para dos mensajes diferentes con la misma clave. El IV debe ser aleatorio y puede viajar junto al mensaje cifrado sin problema.
