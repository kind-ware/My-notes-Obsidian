## Descripción General

`tracing` es un framework para instrumentar programas de Rust con el fin de obtener información de diagnóstico estructurada y basada en eventos. A diferencia del logging tradicional (que solo imprime líneas de texto), `tracing` se basa en **Spans** (lapsos de tiempo) y **Eventos**.

Es especialmente potente en entornos **asíncronos**. Mientras que los logs normales se mezclan de forma caótica cuando varias tareas corren a la vez, `tracing` permite seguir el rastro de una ejecución específica a través de diferentes hilos y tareas, manteniendo el contexto.

### Casos de Uso

*   **Depuración de Sistemas Asíncronos:** Seguir el flujo de una petición web a través de múltiples microservicios o tareas de Tokio.
*   **Perfilado (Profiling):** Medir cuánto tiempo tarda exactamente una función o un bloque de código.
*   **Auditoría Estructurada:** Generar logs en formato JSON para ser consumidos por herramientas como ELK Stack, Honeycomb o Jaeger.
*   **Monitoreo de Producción:** Identificar cuellos de botella mediante el análisis de la jerarquía de llamadas.

## Conceptos y Macros Principales

*   **`Span`**: Representa un periodo de tiempo con un inicio y un fin (ej: el tiempo que tarda una consulta SQL). Los spans son jerárquicos (un span puede tener hijos).
*   **`Event`**: Un punto en el tiempo, similar a un log tradicional, pero con datos estructurados (campos clave-valor).
*   **`Subscriber`**: El componente que decide qué hacer con los datos (imprimirlos, guardarlos en un archivo, enviarlos por red).
*   **`#[instrument]`**: La macro más famosa. Se coloca sobre una función y crea automáticamente un span cada vez que se llama, capturando sus argumentos.
*   **Macros de nivel**: `error!`, `warn!`, `info!`, `debug!`, `trace!`.

## Ejemplos de Código

### Preparación (Cargo.toml)
Necesitas la librería base y un "suscriptor" para ver los resultados en consola.

```toml
[dependencies]
tracing = "0.1"
tracing-subscriber = "0.3" # Para formatear y ver los logs
```

### Ejemplo 1: Configuración Básica y Eventos
Cómo inicializar el sistema y emitir información estructurada.

```rust
use tracing::{info, warn};
use tracing_subscriber;

fn main() {
    // 1. Inicializar el suscriptor para ver los logs en la terminal
    tracing_subscriber::fmt::init();

    let usuario = "Alex86";
    
    // Log estructurado: el mensaje y los datos van por separado
    info!(user = usuario, "Un usuario ha iniciado sesión");

    let intento = 3;
    warn!(attempt = intento, "Fallo en el intento de conexión");
}
```

### Ejemplo 2: Uso de la macro `#[instrument]`
La forma más fácil y potente de seguir funciones (especialmente asíncronas).

```rust
use tracing::{info, instrument};

#[instrument] // Crea un span llamado "procesar_pago" y captura 'id' y 'monto'
async fn procesar_pago(id: u32, monto: f64) {
    info!("Validando tarjeta...");
    // Simular trabajo asíncrono
    tokio::time::sleep(std::time::Duration::from_millis(100)).await;
    info!("Pago procesado con éxito");
}

#[tokio::main]
async fn main() {
    tracing_subscriber::fmt::init();
    procesar_pago(42, 150.50).await;
}
```

### Ejemplo 3: Spans Manuales para Bloques de Código
Si no quieres instrumentar una función entera, puedes crear un span para un bloque específico.

```rust
use tracing::{info, span, Level};

fn realizar_calculo_pesado() {
    // Crear un span manual
    let span = span!(Level::INFO, "calculo_matematico", iteraciones = 1000);
    
    // "Entrar" al span. Todo lo que pase dentro de este scope pertenece al span.
    let _guard = span.enter();
    
    info!("Iniciando fase 1");
    // ... lógica del cálculo ...
    info!("Fase 1 completada");
} // Al salir del scope, el _guard se destruye y el span se cierra automáticamente.
```

## Buenas Prácticas y Consideraciones

1.  **Datos, no texto**: En lugar de escribir `info!("Usuario {} conectado", user)`, prefiere `info!(user = %user, "Conexión detectada")`. Esto permite que herramientas externas puedan filtrar logs por el campo `user` sin tener que usar expresiones regulares complejas.
2.  **Instrumenta funciones `async`**: Usa siempre `#[instrument]` en tus funciones asíncronas de Axum o de base de datos. Esto permite que el ID del span se mantenga consistente incluso cuando Tokio pausa la tarea y la reanuda en otro hilo.
3.  **Evita `println!`**: Una vez que uses `tracing`, destierra `println!`. `tracing` es mucho más rápido (es no bloqueante si se configura con `tracing-appender`) y permite controlar el nivel de verbosidad sin recompilar.
4.  **Cuidado con datos sensibles**: `#[instrument]` captura los argumentos de la función. Si tu función recibe una contraseña o un token, usa `#[instrument(skip(password))]` para evitar que datos privados terminen en los logs.
5.  **Capas (Layers)**: Con `tracing-subscriber` puedes apilar capas. Por ejemplo: una capa para imprimir logs bonitos en la consola durante desarrollo y otra capa para enviar logs en JSON a un archivo o a un servidor de telemetría en producción.