## Descripción General

`reqwest` es un cliente HTTP de alto nivel para Rust. Está diseñado para ser fácil de usar y maneja la mayoría de las tareas que esperarías de un cliente moderno: soporte para HTTPS (TLS), manejo de JSON, cookies, redirecciones, proxies y pools de conexiones. 

Por defecto es **asíncrona** y utiliza el runtime `tokio`, aunque ofrece una versión **blocking** (síncrona) opcional mediante *feature flags*.

### Casos de Uso

*   **Consumo de APIs REST:** Enviar y recibir datos en formato JSON.
*   **Web Scraping:** Descargar contenido HTML de sitios web.
*   **Carga/Descarga de Archivos:** Enviar formularios multipart o descargar binarios.
*   **Microservicios:** Comunicación eficiente entre servicios backend.

## Funciones y Estructuras Principales

*   **`reqwest::Client`**: El motor principal. Se recomienda crear uno solo y reutilizarlo para aprovechar el "connection pooling".
*   **`reqwest::get()`**: Una función de conveniencia para realizar peticiones GET rápidas sin configurar un cliente.
*   **`reqwest::RequestBuilder`**: Permite configurar paso a paso la petición (headers, query params, body).
*   **`reqwest::Response`**: Objeto que contiene el status, los headers y el cuerpo de la respuesta.
*   **`json::<T>()`**: Método para deserializar automáticamente el cuerpo de la respuesta en una estructura de Rust (requiere la feature `json`).

## Ejemplos de Código

### Preparación (Cargo.toml)
Para estos ejemplos, necesitas añadir estas dependencias:

```toml
[dependencies]
reqwest = { version = "0.12", features = ["json"] }
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
```

### Ejemplo 1: Petición GET Básica (Asíncrona)
Obtener el contenido de una URL y manejar el resultado.

```rust
#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let url = "https://httpbin.org/get";
    
    // Petición rápida usando la función de conveniencia
    let res = reqwest::get(url).await?;

    println!("Status: {}", res.status());
    let cuerpo = res.text().await?;
    println!("Cuerpo: {}", cuerpo);

    Ok(())
}
```

### Ejemplo 2: Petición POST con JSON y Tipado Fuerte
Uso de `serde` para enviar y recibir datos estructurados.

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Usuario {
    id: Option<u32>,
    nombre: String,
    email: String,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let cliente = reqwest::Client::new();
    let nuevo_usuario = Usuario {
        id: None,
        nombre: "Alex".into(),
        email: "alex@ejemplo.com".into(),
    };

    let respuesta: Usuario = cliente
        .post("https://jsonplaceholder.typicode.com/users")
        .json(&nuevo_usuario) // Serializa automáticamente
        .send()
        .await?
        .json() // Deserializa automáticamente la respuesta
        .await?;

    println!("Usuario creado con ID: {:?}", respuesta.id);
    Ok(())
}
```

### Ejemplo 3: Configuración de Cliente (Headers y Timeouts)
Personalizar el comportamiento del cliente para producción.

```rust
use std::time::Duration;
use reqwest::header::{HeaderMap, HeaderValue, USER_AGENT};

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    // Configurar headers por defecto para todas las peticiones
    let mut headers = HeaderMap::new();
    headers.insert(USER_AGENT, HeaderValue::from_static("MiAppRust/1.0"));

    let cliente = reqwest::Client::builder()
        .default_headers(headers)
        .timeout(Duration::from_secs(5)) // Timeout global
        .build()?;

    let res = cliente.get("https://httpbin.org/headers").send().await?;
    
    if res.status().is_success() {
        println!("Petición exitosa");
    }

    Ok(())
}
```

## Buenas Prácticas y Consideraciones

1.  **Reutiliza el `Client`**: Crear un `Client` nuevo para cada petición es costoso (abre y cierra sockets constantemente). Lo ideal es instanciarlo una vez y pasarlo por referencia o clonarlo (clonar un `Client` es barato porque usa un `Arc` internamente).
2.  **Manejo de Errores con `error_for_status()`**: `reqwest` no lanza error automáticamente si el servidor responde con un 404 o 500 (la petición fue "exitosa" técnicamente). Usa `.error_for_status()` después de `.send()` para convertir esos códigos de estado en errores de Rust.
3.  **Features en Cargo.toml**: Por defecto, `reqwest` es muy ligero. Si necesitas JSON, debes activar la feature `json`. Si no quieres usar async, activa la feature `blocking`.
4.  **Timeouts**: Siempre define un timeout (global en el builder o específico en la petición). Por defecto, `reqwest` no tiene timeout, lo que podría dejar tu aplicación colgada indefinidamente si el servidor no responde.
5.  **Seguridad (TLS)**: Por defecto utiliza `native-tls` (las librerías del sistema). Si buscas mayor portabilidad o compilar de forma estática, considera usar la feature `rustls`, que es una implementación pura de Rust.