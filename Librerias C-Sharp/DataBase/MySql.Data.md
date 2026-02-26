## Descripción General

Es el controlador (driver) de ADO.NET para MySQL. Permite que las aplicaciones de C# se comuniquen con servidores MySQL o MariaDB. Implementa las interfaces estándar de base de datos de .NET, lo que permite ejecutar comandos SQL, consultar datos y manejar transacciones de forma eficiente.

#### Casos de Uso Comunes

*   **Aplicaciones con Base de Datos:** Guardar información de usuarios, inventarios o ventas.
*   **APIs REST:** Servir datos desde MySQL hacia aplicaciones web o móviles.
*   **Sistemas de Logs:** Almacenar registros históricos de gran volumen.
*   **Reportes:** Consultar grandes conjuntos de datos para generar estadísticas.

## Clases y Métodos Principales

*   **`MySqlConnection`:** Representa la conexión física a la base de datos.
    *   `.Open()` / `.OpenAsync()`: Abre la conexión.
    *   `.Close()`: La cierra (aunque es mejor usar `using`).
*   **`MySqlCommand`:** Representa la consulta SQL que quieres ejecutar.
    *   `.ExecuteNonQuery()`: Para `INSERT`, `UPDATE`, `DELETE` (devuelve filas afectadas).
    *   `.ExecuteReader()`: Para `SELECT` (devuelve un lector de datos).
    *   `.ExecuteScalar()`: Para consultas que devuelven un solo valor (ej. `COUNT(*)`).
*   **`MySqlDataReader`:** Un flujo de solo lectura que recorre los resultados de una consulta fila por fila.
*   **`MySqlParameter`:** Crucial para la seguridad; permite insertar variables en el SQL sin riesgo de **Inyección SQL**.

## Ejemplos Prácticos

> **Instalación:** Necesitas el paquete NuGet `MySqlConnector`.
#### Ejemplo 1: Conexión y Consulta (SELECT)

```csharp
using MySqlConnector; // O MySql.Data.MySqlClient

string connectionString = "Server=localhost;Database=mi_db;Uid=root;Pwd=password;";

using (var connection = new MySqlConnection(connectionString))
{
    await connection.OpenAsync();
    
    string sql = "SELECT id, nombre FROM usuarios WHERE activo = @estado";
    using (var command = new MySqlCommand(sql, connection))
    {
        // Usamos parámetros por seguridad
        command.Parameters.AddWithValue("@estado", 1);

        using (var reader = await command.ExecuteReaderAsync())
        {
            while (await reader.ReadAsync())
            {
                Console.WriteLine($"ID: {reader["id"]}, Nombre: {reader["nombre"]}");
            }
        }
    }
}
```

#### Ejemplo 2: Inserción de Datos (INSERT)

```csharp
string sql = "INSERT INTO productos (nombre, precio) VALUES (@nom, @pre)";

using (var command = new MySqlCommand(sql, connection))
{
    command.Parameters.AddWithValue("@nom", "Teclado Mecánico");
    command.Parameters.AddWithValue("@pre", 45.50);

    int filas = await command.ExecuteNonQueryAsync();
    Console.WriteLine($"Se insertaron {filas} filas.");
}
```

## Buenas Prácticas y Consideraciones

1.  **¡PARÁMETROS SIEMPRE!:** Nunca concatenes strings para crear el SQL (ej: `"WHERE id=" + id`). Esto te hace vulnerable a ataques de Inyección SQL. Usa siempre `Parameters.AddWithValue`.
2.  **Usa Bloques `using`:** Las conexiones a bases de datos son recursos escasos. El bloque `using` asegura que la conexión se devuelva al "pool" de conexiones inmediatamente, incluso si ocurre un error.
3.  **Aprovecha el Async:** Las operaciones de base de datos son de entrada/salida (I/O) y pueden ser lentas. Usa siempre las versiones `Async` (`OpenAsync`, `ExecuteReaderAsync`) para no bloquear el hilo principal de tu app.
4.  **Cadena de Conexión Segura:** No pongas la contraseña directamente en el código. Usa archivos de configuración (`appsettings.json`) o variables de entorno.
5.  **Connection Pooling:** Por defecto, estas librerías reutilizan conexiones abiertas anteriormente para ganar velocidad. No intentes implementar tu propio sistema de caché de conexiones; el driver ya lo hace muy bien por ti.
