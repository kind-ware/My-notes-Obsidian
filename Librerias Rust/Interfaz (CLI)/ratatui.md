## Descripción General

`ratatui` es una librería para construir interfaces de usuario ricas en la terminal. Sigue un modelo de **renderizado en modo inmediato** (*immediate mode*): en cada iteración del bucle principal, la interfaz se dibuja por completo desde cero. 

Es "agnóstica" respecto al backend, lo que significa que no maneja directamente el teclado o el dibujo en pantalla, sino que delega esa tarea a librerías como **`crossterm`** (la más común), `termion` o `termwiz`.

### Casos de Uso

*   **Dashboards de Monitoreo:** Visualizar métricas de servidores o bases de datos en tiempo real.
*   **Gestores de Archivos:** Navegación por directorios con paneles laterales y previsualizaciones.
*   **Herramientas de Desarrollo:** Clientes de Git, editores de texto ligeros o depuradores.
*   **Instaladores Avanzados:** Interfaces con pasos, formularios y barras de progreso complejas.

## Estructuras y Conceptos Principales

*   **`Terminal`**: El punto de entrada que conecta el backend con `ratatui`.
*   **`Frame`**: El lienzo temporal donde dibujas durante un ciclo de renderizado.
*   **`Layout`**: Sistema para dividir la pantalla en áreas (rectángulos) usando restricciones (porcentajes, píxeles fijos, ratios).
*   **`Widget`**: Componentes visuales listos para usar: `Paragraph`, `List`, `Table`, `Tabs`, `Gauge` (barra de progreso), `Block` (bordes y títulos).
*   **`Buffer`**: Representación interna de la pantalla (celdas con caracteres y estilos).

## Ejemplos de Código

### Preparación (Cargo.toml)
Necesitas `ratatui` y un backend como `crossterm`.

```toml
[dependencies]
ratatui = "0.26"
crossterm = "0.27"
```

### Ejemplo 1: Estructura Básica (Hello World)
Configuración del terminal y renderizado de un bloque simple.

```rust
use ratatui::{
    backend::CrosstermBackend,
    widgets::{Block, Borders, Paragraph},
    Terminal,
};
use crossterm::{execute, terminal::{enable_raw_mode, disable_raw_mode, EnterAlternateScreen, LeaveAlternateScreen}};
use std::io;

fn main() -> Result<(), io::Error> {
    // 1. Preparar la terminal
    enable_raw_mode()?;
    let mut stdout = io::stdout();
    execute!(stdout, EnterAlternateScreen)?;
    let backend = CrosstermBackend::new(stdout);
    let mut terminal = Terminal::new(backend)?;

    // 2. Bucle de renderizado
    terminal.draw(|f| {
        let size = f.size();
        let block = Block::default()
            .title("Mi App en Rust")
            .borders(Borders::ALL);
        let texto = Paragraph::new("¡Hola Ratatui!").block(block);
        f.render_widget(texto, size);
    })?;

    // 3. Restaurar la terminal (Pausa para ver el resultado)
    std::thread::sleep(std::time::Duration::from_secs(2));
    disable_raw_mode()?;
    execute!(terminal.backend_mut(), LeaveAlternateScreen)?;
    Ok(())
}
```

### Ejemplo 2: Creación de Layouts (División de Pantalla)
Dividir la terminal en una cabecera, un cuerpo central y un pie de página.

```rust
use ratatui::{
    layout::{Constraint, Direction, Layout},
    widgets::{Block, Borders},
    Frame,
};

fn ui(f: &mut Frame) {
    // Definir la división vertical
    let chunks = Layout::default()
        .direction(Direction::Vertical)
        .margin(1)
        .constraints([
            Constraint::Length(3),    // Cabecera fija
            Constraint::Min(0),       // Cuerpo flexible
            Constraint::Length(3),    // Footer fijo
        ].as_ref())
        .split(f.size());

    let header = Block::default().title(" Cabecera ").borders(Borders::ALL);
    let body = Block::default().title(" Contenido Principal ").borders(Borders::ALL);
    let footer = Block::default().title(" Footer ").borders(Borders::ALL);

    f.render_widget(header, chunks[0]);
    f.render_widget(body, chunks[1]);
    f.render_widget(footer, chunks[2]);
}
```

### Ejemplo 3: Widget de Lista con Estado (Seleccionable)
Uso de `List` para crear un menú interactivo.

```rust
use ratatui::widgets::{List, ListItem, ListState};
// ... (imports de layout y estilo)

fn render_lista(f: &mut Frame, estado: &mut ListState) {
    let items = [ListItem::new("Opción 1"), ListItem::new("Opción 2"), ListItem::new("Opción 3")];
    
    let lista = List::new(items)
        .block(Block::default().title("Menú").borders(Borders::ALL))
        .highlight_symbol(">> ")
        .highlight_style(ratatui::style::Style::default().fg(ratatui::style::Color::Yellow));

    // f.render_stateful_widget permite que la lista "recuerde" cuál está seleccionado
    f.render_stateful_widget(lista, f.size(), estado);
}
```

## Buenas Prácticas y Consideraciones

1.  **Limpieza de Terminal**: Usa siempre un "panic hook" o asegúrate de restaurar la terminal (desactivar *raw mode* y salir de la *alternate screen*) al terminar o si el programa falla. Si no, la terminal del usuario quedará inservible (sin cursor o sin eco de teclas).
2.  **Arquitectura App/UI**: Separa tu lógica. Crea un struct `App` que guarde el estado (datos, posición del cursor) y una función `ui(&mut Frame, &App)` que se encargue exclusivamente del dibujo.
3.  **Manejo de Eventos**: `ratatui` no maneja eventos de teclado. Debes usar el sistema de eventos de tu backend (ej: `crossterm::event::read`) en un hilo separado o en un bucle asíncrono para actualizar el estado de tu app.
4.  **Minimizar Renderizados**: Aunque sea modo inmediato, no redibujes a máxima velocidad (60fps) si no es necesario. Redibuja solo cuando recibas un evento (tecla, click, timer) para ahorrar CPU.
5.  **Constraints Flexibles**: Usa `Constraint::Min` o `Constraint::Percentage` siempre que sea posible para que tu UI se adapte correctamente cuando el usuario cambie el tamaño de la ventana de la terminal.