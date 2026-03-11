## Descripción General

`rusqlite` proporciona una interfaz segura y muy similar a Rust para interactuar con SQLite. Se encarga de la gestión de conexiones, la preparación de sentencias SQL y la conversión automática de tipos entre SQL y Rust. Es una librería sincrónica y muy ligera, ideal para aplicaciones de escritorio, herramientas CLI o almacenamiento local.

### Casos de Uso

*   **Almacenamiento Local:** Guardar configuraciones o datos de usuario en aplicaciones de escritorio.
*   **Caché de Aplicaciones:** Almacenar temporalmente datos descargados de internet.
*   **Bases de Datos Embebidas:** Aplicaciones que necesitan una base de datos potente sin depender de un servidor externo (como PostgreSQL).
*   **Procesamiento de Datos Offline:** Herramientas que analizan grandes volúmenes de datos locales.

## Funciones y Estructuras Principales

*   **`Connection`**: El objeto principal que representa la conexión a la base de datos (en archivo o en memoria).
*   **`execute()`**: Para sentencias que no devuelven filas (INSERT, UPDATE, DELETE, CREATE).
*   **`query_row()`**: Para obtener un único registro.
*   **`prepare()`**: Crea una sentencia preparada para ser ejecutada múltiples veces (más eficiente).
*   **`params![]`**: Una macro para pasar argumentos de forma segura, evitando inyecciones SQL.
*   **`OptionalExtension`**: Un trait que facilita manejar casos donde una consulta puede no devolver resultados (retornando `Option<T>`).

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
# La feature "bundled" compila SQLite internamente (no necesitas instalar nada en el sistema)
rusqlite = { version = "0.31", features = ["bundled"] }
```

### Ejemplo 1: Crear Tabla e Insertar Datos
Uso básico de una conexión en memoria y ejecución de comandos.

```rust
use rusqlite::{params, Connection, Result};

fn main() -> Result<()> {
    // Crear una base de datos en memoria (desaparece al cerrar el programa)
    let conn = Connection::open_in_memory()?;

    conn.execute(
        "CREATE TABLE persona (
            id INTEGER PRIMARY KEY,
            nombre TEXT NOT NULL,
            data BLOB
        )",
        [],
    )?;

    let nueva_persona = "Alex";
    conn.execute(
        "INSERT INTO persona (nombre) VALUES (?1)",
        params![nueva_persona],
    )?;

    println!("Tabla creada y registro insertado.");
    Ok(())
}
```

### Ejemplo 2: Consultar Múltiples Filas (Iteradores)
Cómo mapear filas de la base de datos a estructuras de Rust.

```rust
use rusqlite::{Connection, Result};

#[derive(Debug)]
struct Persona {
    id: i32,
    nombre: String,
}

fn main() -> Result<()> {
    let conn = Connection::open("mi_base_de_datos.db")?;

    let mut stmt = conn.prepare("SELECT id, nombre FROM persona")?;
    
    // map() transforma cada fila en un struct Persona
    let persona_iter = stmt.query_map([], |row| {
        Ok(Persona {
            id: row.get(0)?,
            nombre: row.get(1)?,
        })
    })?;

    for persona in persona_iter {
        println!("Encontrado: {:?}", persona?);
    }

    Ok(())
}
```

### Ejemplo 3: Transacciones
Asegurar que un grupo de operaciones se complete correctamente o no se haga ninguna.

```rust
use rusqlite::{Connection, Result};

fn main() -> Result<()> {
    let mut conn = Connection::open("data.db")?;

    // Iniciar una transacción
    let tx = conn.transaction()?;

    tx.execute("INSERT INTO persona (nombre) VALUES (?1)", ["Juan"])?;
    tx.execute("INSERT INTO persona (nombre) VALUES (?1)", ["Maria"])?;

    // Si algo fallara aquí, tx se descarta automáticamente al salir del scope
    // pero si llamamos a commit(), los cambios se guardan.
    tx.commit()?;

    println!("Transacción completada con éxito.");
    Ok(())
}
```

## Buenas Prácticas y Consideraciones

1.  **Feature `bundled`**: En Windows y macOS, es muy recomendable usar la feature `bundled` en `Cargo.toml`. Esto hace que `rusqlite` compile su propia copia de SQLite, evitando errores de "librería no encontrada" en el sistema del usuario.
2.  **Usa `params!`**: Nunca concatenes strings para crear SQL (`format!("... WHERE id = {}", id)`). Usa siempre la macro `params![id]` para prevenir ataques de inyección SQL y dejar que la librería maneje el escapado de caracteres.
3.  **Sentencias Preparadas**: Si vas a ejecutar la misma consulta muchas veces (por ejemplo, dentro de un bucle), usa `conn.prepare()`. Es mucho más rápido porque SQLite solo tiene que analizar el SQL una vez.
4.  **Manejo de Tipos**: `rusqlite` soporta los tipos básicos (integers, floats, text, blobs). Si necesitas guardar tipos complejos (como un struct), combínalo con **`serde_json`** para guardar el objeto como un string JSON en una columna de texto.
5.  **Conexiones y Hilos**: Una `Connection` de `rusqlite` **no** se puede compartir entre hilos de forma segura (`!Sync`). Si usas un entorno multihilo (como `tokio`), considera usar un pool de conexiones como **`r2d2_sqlite`** o cambiar a la librería **`sqlx`**, que es asíncrona por naturaleza.
