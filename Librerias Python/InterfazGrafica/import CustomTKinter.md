### CustomTKinter

`customtkinter` (abreviado a menudo como `ctk`) es un wrapper de `tkinter`.
Si ya sabes algo de `tkinter`, ya sabes usar el 90% de esta librería. La diferencia es que en lugar de llamar a `tk.Button`, llamas a `ctk.CTkButton` y automáticamente se ve genial.

**Filosofía:**

Se basa en **Widgets** (elementos) que se colocan dentro de una **Ventana Principal**.
Para organizarlos, se usan "Gestores de Geometría": `.pack()` (apilar) o `.grid()` (cuadrícula).

### Configuración Inicial

Antes de crear la ventana, configuras el "look & feel".

| Función                         | Descripción                                                                                          |
| :------------------------------ | :--------------------------------------------------------------------------------------------------- |
| `set_appearance_mode(modo)`     | Define el tema. Opciones: `"System"` (Sigue al OS), `"Dark"`, `"Light"`.                             |
| `set_default_color_theme(tema)` | Define el color de acento (bordes, botones). Opciones: `"blue"` (default), `"green"`, `"dark-blue"`. |

### Widgets Principales

Todos los widgets llevan el prefijo `CTk`.

| Widget               | Descripción                                            |
| :------------------- | :----------------------------------------------------- |
| `CTk`                | La ventana principal (reemplaza a `tk.Tk`).            |
| `CTkFrame`           | Un contenedor rectangular para agrupar otros widgets.  |
| `CTkButton`          | Botón con esquinas redondeadas y efectos hover.        |
| `CTkEntry`           | Campo de entrada de texto.                             |
| `CTkLabel`           | Texto estático.                                        |
| `CTkCheckBox`        | Casilla de verificación cuadrada.                      |
| `CTkSwitch`          | Interruptor estilo móvil (Toggle) muy moderno.         |
| `CTkSlider`          | Barra deslizante.                                      |
| `CTkScrollableFrame` | Un marco que tiene barra de desplazamiento automática. |

### Ejemplos de Código

##### Ejemplo 1: Hola Mundo Moderno (Estilo Script)

Lo más básico para ver la diferencia visual.

```python
import customtkinter as ctk

# 1. Configuración global
ctk.set_appearance_mode("System")  # Detecta si tu PC está en modo oscuro
ctk.set_default_color_theme("blue")

# 2. Crear la ventana
app = ctk.CTk()
app.geometry("400x240")
app.title("Mi App Moderna")

# 3. Función del botón
def evento_boton():
    print("¡Botón presionado!")

# 4. Agregar Widgets
# master=app indica dónde vive el botón.
# place() usa coordenadas absolutas (x, y)
boton = ctk.CTkButton(master=app, text="Haz Clic", command=evento_boton)
boton.place(relx=0.5, rely=0.5, anchor=ctk.CENTER)

# 5. Loop de ejecución
app.mainloop()
```

##### Ejemplo 2: Programación Orientada a Objetos (La forma Profesional)

En GUIs complejas, no se usan scripts sueltos. Se crea una **Clase** que hereda de `CTk`. Esto permite compartir datos entre botones y funciones fácilmente usando `self`.

```python
import customtkinter as ctk

class AppCalculadora(ctk.CTk):
    def __init__(self):
        super().__init__()

        # Config ventana
        self.title("Calculadora Pro")
        self.geometry("300x200")

        # Layout GRID (Grilla de filas y columnas)
        # Configurar peso de columnas para que se estiren
        self.grid_columnconfigure(0, weight=1)
        self.grid_columnconfigure(1, weight=1)

        # Widget 1: Entrada
        self.entrada = ctk.CTkEntry(self, placeholder_text="Escribe un número")
        self.entrada.grid(row=0, column=0, columnspan=2, padx=20, pady=20, sticky="ew")

        # Widget 2: Botón Sumar
        self.btn_sumar = ctk.CTkButton(self, text="Duplicar", command=self.duplicar_valor)
        self.btn_sumar.grid(row=1, column=0, padx=10, pady=10)

        # Widget 3: Botón Limpiar
        # fg_color="transparent" crea un botón 'fantasma' (solo borde)
        self.btn_clear = ctk.CTkButton(self, text="Limpiar", fg_color="transparent", border_width=2, command=self.limpiar)
        self.btn_clear.grid(row=1, column=1, padx=10, pady=10)

        # Widget 4: Resultado
        self.lbl_resultado = ctk.CTkLabel(self, text="Resultado: 0", font=("Arial", 16, "bold"))
        self.lbl_resultado.grid(row=2, column=0, columnspan=2)

    def duplicar_valor(self):
        try:
            valor = int(self.entrada.get())
            nuevo_valor = valor * 2
            self.lbl_resultado.configure(text=f"Resultado: {nuevo_valor}")
        except ValueError:
            self.lbl_resultado.configure(text="Error: Ingresa un número")

    def limpiar(self):
        self.entrada.delete(0, 'end')
        self.lbl_resultado.configure(text="Resultado: 0")

if __name__ == "__main__":
    app = AppCalculadora()
    app.mainloop()
```

##### Ejemplo 3: Login Moderno con Frames

Aquí veremos cómo agrupar elementos usando `CTkFrame` para centrar todo.

```python
import customtkinter as ctk

ctk.set_appearance_mode("Dark")
ctk.set_default_color_theme("green")

class LoginApp(ctk.CTk):
    def __init__(self):
        super().__init__()
        self.geometry("500x400")
        self.title("Sistema Seguro")

        # Crear un Frame centrado
        self.frame = ctk.CTkFrame(master=self)
        self.frame.pack(pady=20, padx=60, fill="both", expand=True)

        # Título dentro del frame
        self.label = ctk.CTkLabel(master=self.frame, text="Iniciar Sesión", font=("Roboto", 24))
        self.label.pack(pady=12, padx=10)

        # Inputs
        self.user_entry = ctk.CTkEntry(master=self.frame, placeholder_text="Usuario")
        self.user_entry.pack(pady=12, padx=10)

        # show="*" oculta la contraseña
        self.pass_entry = ctk.CTkEntry(master=self.frame, placeholder_text="Contraseña", show="*")
        self.pass_entry.pack(pady=12, padx=10)

        # Switch "Recordarme"
        self.checkbox = ctk.CTkCheckBox(master=self.frame, text="Recordarme")
        self.checkbox.pack(pady=12, padx=10)

        # Botón
        self.button = ctk.CTkButton(master=self.frame, text="Entrar", command=self.login)
        self.button.pack(pady=12, padx=10)

    def login(self):
        print(f"Usuario: {self.user_entry.get()}")
        print(f"Password: {self.pass_entry.get()}")
        print(f"Recordar: {self.checkbox.get()}")

if __name__ == "__main__":
    app = LoginApp()
    app.mainloop()
```

### Diferencia: `PySimpleGUI` vs `customtkinter`

| Característica           | PySimpleGUI                             | customtkinter                                   |
| :----------------------- | :-------------------------------------- | :---------------------------------------------- |
| **Código**               | Basado en Listas (Layouts simples).     | Basado en Objetos y Clases (Más control).       |
| **Curva de aprendizaje** | Muy Baja (Fácil).                       | Media (Requiere entender POO).                  |
| **Estética**             | Clásica (a menos que configures mucho). | Moderna por defecto (Estilo macOS/Win11).       |
| **Event Loop**           | Lo escribes tú (`while True`).          | Lo maneja la librería (`mainloop`).             |
| **Uso Ideal**            | Scripts rápidos, herramientas internas. | Aplicaciones de escritorio completas y bonitas. |

### Integración con Stack

1.  **Con `threading`**:
    Al igual que en PySimpleGUI, **no puedes** ejecutar tareas lentas (como descargar archivos) dentro de los métodos de la clase, o la ventana se congelará.
    Debes lanzar un hilo y usar colas o variables de control para actualizar la UI.

2.  **Con `sqlite3`**:
    En el método `def login(self):`, capturas el usuario y contraseña, y llamas a tu función de base de datos para verificar credenciales.

3.  **Con `CTkMessagebox`**:
    `customtkinter` no trae popups por defecto (como `sg.popup`). Existe una librería extra llamada `CTkMessagebox` para mostrar alertas bonitas.

### Resumen

Con `customtkinter`, tus aplicaciones de Python ya no parecen hechas por un científico en los años 90. Parecen aplicaciones nativas modernas.

Opciones:

*   **CLI:** `Rich` + `Typer` (Para terminal).
*   **GUI Rápida:** `PySimpleGUI` (Para scripts).
*   **GUI Profesional:** `customtkinter` (Para productos finales).

Para seguir, podríamos ver una librería muy divertida llamada **`tqdm`** (barras de carga para la terminal, más simple que Rich) o quizás **`schedule`** (para ejecutar tareas cada X tiempo, como un cronjob). ¿Te animas?