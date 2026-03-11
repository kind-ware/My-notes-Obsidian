## Descripción General

`sqlx` es un toolkit SQL moderno, asíncrono y de "puro Rust". Su característica estrella es la capacidad de **validar tus consultas SQL en tiempo de compilación** conectándose a tu base de datos mientras compilas. Si escribes mal un nombre de columna o un tipo no coincide, el compilador de Rust te dará un error antes de que el programa se ejecute.

Soporta PostgreSQL, MySQL, SQLite y MSSQL. No es un ORM (como Diesel o Hibernate), sino un "SQL-first" wrapper que te da control total sobre las consultas.

### Casos de Uso

*   **Backends Asíncronos:** Integración perfecta con `axum`, `actix-web` o `tokio`.
*   **Sistemas Críticos:** Donde la seguridad de tipos es vital para evitar errores en producción.
*   **Microservicios de Alto Rendimiento:** Gracias a su pool de conexiones eficiente.
*   **Gestión de Migraciones:** Incluye herramientas para manejar versiones del esquema de la base de datos.

## Funciones y Macros Principales

*   **`PgPool` / `MySqlPool` / `SqlitePool`**: Gestores de un grupo de conexiones (Pool) para manejar múltiples peticiones concurrentes.
*   **`query!`**: Macro que valida el SQL contra la base de datos en tiempo de compilación y devuelve un struct anónimo.
*   **`query_as!`**: Similar a `query!`, pero mapea el resultado directamente a un struct de Rust.
*   **`execute()`**: Para comandos que no devuelven filas (INSERT, UPDATE, DELETE).
*   **`fetch_one`, `fetch_all`, `fetch_optional`**: Métodos para recuperar los datos.
*   **`FromRow`**: Trait para derivar el mapeo automático de columnas a campos de un struct.

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
sqlx = { version = "0.7", features = ["runtime-tokio", "tls-native-tls", "postgres", "macros"] }
tokio = { version = "1", features = ["full"] }
```

>Nota: Para que las macros funcionen al compilar, debes tener una variable de entorno `DATABASE_URL` apuntando a una DB real.*

### Ejemplo 1: Conexión y Pool
Configuración básica para empezar a operar.

```rust
use sqlx::postgres::PgPoolOptions;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let db_url = "postgres://usuario:password@localhost/mi_db";

    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect(db_url)
        .await?;

    println!("Conexión establecida con éxito.");
    Ok(())
}
```

### Ejemplo 2: Consulta con Validación en Tiempo de Compilación
Uso de la macro `query!` para máxima seguridad.

```rust
async fn obtener_usuario(pool: &sqlx::PgPool, id: i32) -> Result<(), sqlx::Error> {
    // Si la columna 'email' no existe, esto NO COMPILARÁ.
    let fila = sqlx::query!("SELECT nombre, email FROM usuarios WHERE id = $1", id)
        .fetch_one(pool)
        .await?;

    println!("Usuario: {} <{}>", fila.nombre, fila.email);
    Ok(())
}
```

### Ejemplo 3: Mapeo a Structs con `query_as!`
Combinando `FromRow` para transformar filas en objetos de Rust.

```rust
#[derive(sqlx::FromRow, Debug)]
struct Producto {
    id: i32,
    nombre: String,
    precio: f64,
}

async fn listar_productos(pool: &sqlx::PgPool) -> Result<Vec<Producto>, sqlx::Error> {
    let productos = sqlx::query_as!(
        Producto,
        "SELECT id, nombre, precio FROM productos WHERE precio > $1",
        10.50
    )
    .fetch_all(pool)
    .await?;

    Ok(productos)
}
```

## Buenas Prácticas y Consideraciones

1.  **Variable `DATABASE_URL`**: Recuerda que para usar `query!` o `query_as!`, necesitas tener la base de datos corriendo durante el desarrollo. Si quieres compilar sin una base de datos activa (ej. en CI/CD), debes usar el modo **"offline"** de sqlx (`sqlx-data.json`).
2.  **Usa el Pool, no conexiones individuales**: Crea un `Pool` una sola vez y clónalo para pasarlo a tus diferentes funciones o rutas web. Clonar un `Pool` es muy barato (es un puntero inteligente).
3.  **Manejo de Optional**: Si una columna puede ser NULL en la DB, asegúrate de que el campo en tu struct de Rust sea un `Option<T>`. `sqlx` fallará si intenta meter un NULL en un tipo que no es opcional.
4.  **Migraciones**: Usa la carpeta `migrations/` y la herramienta de línea de comandos `sqlx-cli`. Esto permite que tu esquema de base de datos esté versionado junto con tu código de Rust.
5.  **SQL Dinámico**: Las macros `query!` no funcionan con SQL generado dinámicamente (strings que construyes en runtime). Para esos casos, usa `sqlx::query` (sin el signo de exclamación), aunque perderás la validación en tiempo de compilación.
