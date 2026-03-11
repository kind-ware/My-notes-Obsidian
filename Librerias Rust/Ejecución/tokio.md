## Descripción General

`tokio` es el **runtime asíncrono** más popular y robusto para Rust. Proporciona los bloques de construcción necesarios para escribir aplicaciones de red y de I/O (entrada/salida) que sean extremadamente rápidas y escalables.

A diferencia de otros lenguajes donde el runtime asíncrono viene integrado (como Node.js o Go), en Rust es una librería externa. `tokio` gestiona el "hilo de eventos" (event loop), la planificación de tareas (scheduling), y ofrece versiones asíncronas de primitivas de red, archivos y temporizadores.

### Casos de Uso

*   **Servidores Web de Alto Rendimiento:** Capaces de manejar miles o millones de conexiones simultáneas (usado por `axum`, `warp`, etc.).
*   **Sistemas de Mensajería:** Implementación de protocolos como MQTT, WebSockets o gRPC.
*   **Microservicios:** Comunicación eficiente y no bloqueante entre servicios.
*   **Proxies y Balanceadores de Carga:** Procesamiento de tráfico de red con baja latencia.

## Componentes y Funciones Principales

*   **`#[tokio::main]`**: Macro que transforma la función `main` en un punto de entrada asíncrono.
*   **`tokio::spawn`**: Crea una "tarea" (green thread) que se ejecuta de forma independiente y concurrente en el runtime.
*   **`tokio::time`**: Funciones para manejar el tiempo sin bloquear el hilo (`sleep`, `timeout`, `interval`).
*   **`tokio::sync`**: Primitivas de sincronización asíncronas (Canales `mpsc`, `oneshot`, `broadcast`, y `Mutex` asíncrono).
*   **`tokio::net`**: Versiones asíncronas de `TcpListener` y `TcpStream`.
*   **`tokio::fs`**: Operaciones con archivos que no bloquean el pool de hilos principal.

## Ejemplos de Código

### Preparación (Cargo.toml)
`tokio` es modular. Normalmente querrás todas sus funcionalidades básicas:

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

### Ejemplo 1: Tareas Concurrentes con `tokio::spawn`
Cómo ejecutar varias tareas al mismo tiempo y esperar a que terminen.

```rust
#[tokio::main]
async fn main() {
    let tarea1 = tokio::spawn(async {
        tokio::time::sleep(tokio::time::Duration::from_secs(2)).await;
        println!("Tarea 1 finalizada después de 2s");
    });

    let tarea2 = tokio::spawn(async {
        tokio::time::sleep(tokio::time::Duration::from_secs(1)).await;
        println!("Tarea 2 finalizada después de 1s");
    });

    // Esperar a que ambas tareas terminen (similar a Promise.all)
    let _ = tokio::join!(tarea1, tarea2);
    println!("Todas las tareas terminadas.");
}
```

### Ejemplo 2: Comunicación entre Tareas (Channels)
Uso de canales `mpsc` (Multi-Producer, Single-Consumer) para enviar mensajes de un lugar a otro.

```rust
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel(32); // Buffer de 32 mensajes

    tokio::spawn(async move {
        for i in 1..=5 {
            let mensaje = format!("Mensaje número {}", i);
            tx.send(mensaje).await.unwrap();
        }
    });

    while let Some(valor) = rx.recv().await {
        println!("Recibido: {}", valor);
    }
}
```

### Ejemplo 3: Servidor TCP Eco Básico
Un servidor que recibe texto y lo devuelve, manejando cada cliente en una tarea separada.

```rust
use tokio::net::TcpListener;
use tokio::io::{AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Servidor escuchando en el puerto 8080...");

    loop {
        let (mut socket, _) = listener.accept().await?;

        tokio::spawn(async move {
            let mut buf = [0; 1024];
            loop {
                let n = socket.read(&mut buf).await.expect("Fallo al leer");
                if n == 0 { return; } // Conexión cerrada
                
                socket.write_all(&buf[0..n]).await.expect("Fallo al escribir");
            }
        });
    }
}
```

## 5. Buenas Prácticas y Consideraciones

1.  **¡No bloquees el Runtime!**: Esta es la regla de oro. Nunca uses funciones bloqueantes de la librería estándar (`std::fs::File`, `std::thread::sleep`, o bucles de cálculo intensivo muy largos) dentro de una función `async`. Si lo haces, detendrás a todos los demás usuarios de ese hilo. Para cálculos pesados, usa `tokio::task::spawn_blocking`.
2.  **Usa `select!` para cancelación**: La macro `tokio::select!` permite esperar múltiples operaciones asíncronas y actuar sobre la primera que termine, cancelando automáticamente las demás. Es ideal para implementar timeouts manuales o señales de apagado.
3.  **Tareas vs Hilos**: Una tarea de `tokio` es mucho más ligera que un hilo del sistema operativo. Puedes crear decenas de miles de tareas sin problemas, pero no deberías crear miles de hilos de OS.
4.  **Mutex de `std` vs Mutex de `tokio`**: 
    *   Usa el `std::sync::Mutex` si solo vas a proteger datos pequeños y no vas a mantener el bloqueo a través de un `.await`.
    *   Usa el `tokio::sync::Mutex` solo si necesitas mantener el bloqueo mientras esperas otra operación asíncrona.
5.  **Graceful Shutdown**: Diseña tus aplicaciones para que las tareas terminen limpiamente (por ejemplo, usando canales de "broadcast" para avisar a todas las tareas que el servidor se está cerrando).