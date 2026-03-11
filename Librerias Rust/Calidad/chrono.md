## Descripción General

`chrono` permite realizar operaciones complejas con fechas y horas. Su diseño separa claramente los conceptos de:
1.  **Puntos en el tiempo con zona horaria**: Como el "ahora" en UTC o en tu hora local.
2.  **Fechas y horas "ingenuas" (Naive)**: Valores que no tienen una zona horaria asociada (ej: "15 de mayo a las 10:00 AM", sin saber si es en Madrid o Tokio).
3.  **Duraciones**: La cantidad de tiempo que pasa entre dos puntos.

Es una librería robusta que cumple con los estándares ISO 8601 y es compatible con `serde` para serialización.

### Casos de Uso

*   **Registros (Logging):** Añadir marcas de tiempo precisas a los eventos del sistema.
*   **Bases de Datos:** Almacenar y recuperar fechas de creación o actualización de registros.
*   **Programación de Tareas:** Calcular cuándo debe ejecutarse la siguiente tarea (ej: "dentro de 3 días a las 8:00 AM").
*   **Interfaces de Usuario:** Mostrar fechas formateadas según el idioma y zona horaria del usuario.

## Tipos y Funciones Principales

*   **`DateTime<Utc>` / `DateTime<Local>`**: Representa un punto absoluto en el tiempo.
*   **`NaiveDate` / `NaiveTime` / `NaiveDateTime`**: Para fechas y horas sin zona horaria (útil para bases de datos que no guardan el offset).
*   **`Duration`**: (Re-exportado o usado mediante la crate `time` o el propio `chrono`) para representar intervalos.
*   **`Datelike` / `Timelike`**: Traits que permiten extraer el día, mes, año, hora, minuto, etc.
*   **`format()`**: Para convertir fechas en strings usando sintaxis tipo `strftime`.
*   **`parse_from_str()`**: Para convertir strings en objetos de fecha.

## Ejemplos de Código

### Preparación (Cargo.toml)

```toml
[dependencies]
chrono = "0.4"
```

### Ejemplo 1: Obtener la Hora Actual y Formatear
Cómo obtener el tiempo actual y mostrarlo en un formato legible.

```rust
use chrono::{Utc, Local};

fn main() {
    // Hora actual en UTC
    let ahora_utc = Utc::now();
    println!("UTC: {}", ahora_utc);

    // Hora actual en la zona horaria del sistema
    let ahora_local = Local::now();
    println!("Local: {}", ahora_local);

    // Formateo personalizado (Día-Mes-Año Hora:Minuto:Segundo)
    let formateada = ahora_local.format("%d/%m/%Y %H:%M:%S").to_string();
    println!("Formato amigable: {}", formateada);
}
```

### Ejemplo 2: Aritmética de Fechas y Duraciones
Calcular una fecha futura o la diferencia entre dos momentos.

```rust
use chrono::{Utc, Days, Duration};

fn main() {
    let hoy = Utc::now();

    // Sumar 10 días usando el método checked_add_days (más seguro)
    let en_diez_dias = hoy.checked_add_days(Days::new(10)).unwrap();
    
    // Restar 5 horas
    let hace_cinco_horas = hoy - Duration::hours(5);

    println!("Hoy: {}", hoy.format("%Y-%m-%d"));
    println!("En 10 días será: {}", en_diez_dias.format("%Y-%m-%d"));
    println!("Hace 5 horas era: {}", hace_cinco_horas);

    // Diferencia entre fechas
    let duracion = en_diez_dias.signed_duration_since(hoy);
    println!("Diferencia en días: {}", duracion.num_days());
}
```

### Ejemplo 3: Parsing (De String a Objeto Fecha)
Convertir una cadena de texto en un objeto con el que podamos trabajar.

```rust
use chrono::{NaiveDate, NaiveDateTime};

fn main() {
    // Parsear solo una fecha
    let fecha_str = "2023-12-25";
    let fecha = NaiveDate::parse_from_str(fecha_str, "%Y-%m-%d").expect("Fecha inválida");
    println!("Año parseado: {}", fecha.year());

    // Parsear fecha y hora
    let fecha_hora_str = "2023-12-25 15:30:00";
    let fecha_hora = NaiveDateTime::parse_from_str(fecha_hora_str, "%Y-%m-%d %H:%M:%S")
        .expect("Formato incorrecto");

    println!("La cita es el: {}", fecha_hora.format("%A, %e de %B"));
}
```

## Buenas Prácticas y Consideraciones

1.  **UTC por defecto**: En el backend y en la base de datos, almacena **siempre** los tiempos en UTC. Solo convierte a `Local` en la capa de presentación cuando necesites mostrárselo al usuario final.
2.  **Cuidado con los tipos `Naive`**: Úsalos solo si estás seguro de que la zona horaria no importa (ej: un cumpleaños, que es el mismo día sin importar dónde estés). Para eventos globales (logs, transacciones), usa siempre `DateTime<Utc>`.
3.  **Manejo de errores**: El parsing de fechas suele fallar si el formato no es exacto. No uses `unwrap()` en strings que vengan del usuario; usa `Result` y maneja el posible error de formato.
4.  **Feature `serde`**: Si necesitas enviar fechas en un JSON, activa la feature en tu `Cargo.toml`: `chrono = { version = "0.4", features = ["serde"] }`. Esto permitirá que `serde` serialice las fechas automáticamente en formato ISO 8601.
5.  **Aritmética Segura**: Prefiere los métodos `checked_add_*` o `checked_sub_*` en lugar de los operadores `+` o `-` si existe la posibilidad de que la fecha resultante sea inválida (aunque en el calendario humano es difícil que ocurra, es una buena práctica de Rust).