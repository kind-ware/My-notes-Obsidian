## Descripción General

`axum` es un framework de desarrollo web diseñado para ser ergonómico, modular y robusto. Su gran ventaja es que aprovecha el ecosistema de **Tower** (una librería de componentes modulares para clientes y servidores) y se basa en **Hyper** (la implementación de HTTP de bajo nivel).

A diferencia de otros frameworks, `axum` no utiliza macros complejas; se basa en el sistema de tipos de Rust para manejar las peticiones, lo que significa que si tu código compila, es muy probable que funcione correctamente a nivel de rutas y tipos de datos.

## Casos de Uso

*   **APIs REST de alto rendimiento:** Microservicios que necesitan procesar miles de peticiones por segundo.
*   **Servidores de WebSockets:** Aplicaciones en tiempo real (chats, dashboards).
*   **Backends para SPAs:** Servir datos JSON a frontends hechos en React, Vue o Angular.
*   **Servicios de Proxy o Gateways:** Gracias a su arquitectura basada en Tower.

## Funciones y Conceptos Principales

*   **`Router`**: El núcleo donde se definen las rutas y se asocian con los manejadores (handlers).
*   **`Handler`**: Funciones asíncronas normales que reciben argumentos y devuelven respuestas.
*   **Extractores (`Json`, `Path`, `Query`, `State`)**: Argumentos en las funciones que "extraen" datos de la petición automáticamente.
*   **`IntoResponse`**: Un trait que permite que casi cualquier cosa (un String, un JSON, un código de estado) se convierta automáticamente en una respuesta HTTP.
*   **`State`**: El mecanismo oficial para compartir datos (como conexiones a bases de datos) entre diferentes rutas de forma segura.

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
axum = "0.7"
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
```

### Ejemplo 1: "Hola Mundo" y Rutas Básicas
Un servidor mínimo que responde a una petición GET.

```rust
use axum::{routing::get, Router};

#[tokio::main]
async fn main() {
    // Definir las rutas
    let app = Router::new()
        .route("/", get(|| async { "¡Hola desde Axum!" }));

    // Lanzar el servidor con Tokio
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    println!("Servidor corriendo en http://localhost:3000");
    axum::serve(listener, app).await.unwrap();
}
```

### Ejemplo 2: API JSON (Envío y Recepción)
Uso de `serde` y extractores para manejar datos estructurados.

```rust
use axum::{routing::post, Json, Router};
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct LoginRequest {
    usuario: String,
    clave: String,
}

#[derive(Serialize)]
struct LoginResponse {
    token: String,
    status: String,
}

async fn login_handler(Json(payload): Json<LoginRequest>) -> Json<LoginResponse> {
    println!("Login para: {}", payload.usuario);
    
    Json(LoginResponse {
        token: "abc-123-secret".to_string(),
        status: "success".to_string(),
    })
}

#[tokio::main]
async fn main() {
    let app = Router::new().route("/login", post(login_handler));
    
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

### Ejemplo 3: Uso de Estado Compartido (`State`)
Cómo pasar una "Base de Datos" (o cualquier objeto) a tus rutas.

```rust
use ax_state::{State, routing::get, Router};
use std::sync::Arc;

struct AppState {
    db_connection: String,
}

async fn mostrar_db(State(estado): State<Arc<AppState>>) -> String {
    format!("Conectado a: {}", estado.db_connection)
}

#[tokio::main]
async fn main() {
    let compartido = Arc::new(AppState {
        db_connection: "sqlite://datos.db".to_string(),
    });

    let app = Router::new()
        .route("/db", get(mostrar_db))
        .with_state(compartido); // Inyectamos el estado

    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

## Buenas Prácticas y Consideraciones

1.  **Orden de los Extractores**: Axum es estricto con el orden. Los extractores que consumen el cuerpo de la petición (como `Json`) deben ser siempre **el último argumento** de tu función handler.
2.  **Usa `Arc` para el Estado**: Cuando compartas datos a través de `.with_state()`, envuélvelos en un `Arc` (Atomic Reference Counter). Esto permite que el estado se comparta entre múltiples hilos de forma eficiente y segura.
3.  **Manejo de Errores**: En lugar de usar `unwrap()` dentro de los handlers, haz que tus funciones devuelvan `Result<impl IntoResponse, StatusCode>`. Esto permite enviar errores HTTP (como 404 o 500) de forma limpia.
4.  **Aprovecha Tower Middleware**: Axum permite añadir capas (layers) fácilmente. Puedes usar `tower-http` para añadir compresión (Gzip), manejo de CORS, o logging de peticiones con solo una línea en tu `Router`.
5.  **Tipado fuerte**: Define structs para tus parámetros de consulta (`Query<T>`) y de ruta (`Path<T>`). No intentes parsear strings manualmente; deja que el sistema de tipos de Axum lo haga por ti y devuelva un error 400 automático si el formato es incorrecto.
