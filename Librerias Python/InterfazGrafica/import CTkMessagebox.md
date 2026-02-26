### CTkMessagebox

Es un widget que crea una ventana emergente modal (bloquea la ventana principal hasta que se cierra).
Se adapta automáticamente al tema (`set_appearance_mode`) que hayas configurado en `customtkinter`.

**Características:**
*   **Iconos:** Check, Warning, Error, Info.
*   **Sonidos:** Opción de reproducir sonido de sistema al abrir.
*   **Botones:** Totalmente personalizables ("Sí", "No", "Tal vez", "Cancelar").
*   **Retorno:** Devuelve el valor del botón presionado para que tomes decisiones.

Instalación: `pip install CTkMessagebox`

### Sintaxis Básica

```python
from CTkMessagebox import CTkMessagebox

# Crear y mostrar (Se destruye sola al cerrar)
msg = CTkMessagebox(title="Título", message="Mensaje", icon="info")
```

| Parámetro                 | Descripción                                                                  |
| :------------------------ | :--------------------------------------------------------------------------- |
| `title`                   | El texto en la barra superior de la ventana.                                 |
| `message`                 | El cuerpo del mensaje.                                                       |
| `icon`                    | El icono visual: `"check"`, `"cancel"`, `"info"`, `"warning"`, `"question"`. |
| `option_1`, `option_2`... | Textos para los botones.                                                     |
| `width`, `height`         | Tamaño manual (opcional).                                                    |
| `fade_in_duration`        | Animación de desvanecimiento (0 para desactivar).                            |

### Ejemplos de Código

##### Ejemplo 1: Tipos de Iconos (Visualización)

Un script simple para ver los diferentes estilos.

```python
import customtkinter as ctk
from CTkMessagebox import CTkMessagebox

# Configuración básica
ctk.set_appearance_mode("Dark")
ctk.set_default_color_theme("blue")

def mostrar_info():
    CTkMessagebox(title="Info", message="Operación completada con éxito.", icon="check")

def mostrar_error():
    # option_1="Aceptar" cambia el texto del botón por defecto (que es 'OK')
    CTkMessagebox(title="Error", message="No se pudo conectar a la base de datos.", icon="cancel", option_1="Entendido")

def mostrar_advertencia():
    CTkMessagebox(title="Cuidado", message="El disco está casi lleno.", icon="warning")

# Ventana Principal
app = ctk.CTk()
app.geometry("300x200")

btn1 = ctk.CTkButton(app, text="Ver Éxito", command=mostrar_info)
btn1.pack(pady=10)

btn2 = ctk.CTkButton(app, text="Ver Error", command=mostrar_error, fg_color="red")
btn2.pack(pady=10)

btn3 = ctk.CTkButton(app, text="Ver Alerta", command=mostrar_advertencia, fg_color="orange")
btn3.pack(pady=10)

app.mainloop()
```

##### Ejemplo 2: Tomar Decisiones (Sí / No / Cancelar)

Este es el uso más importante: Preguntar al usuario y actuar según su respuesta.

**El método `.get()`** devuelve el texto del botón presionado.

```python
import customtkinter as ctk
from CTkMessagebox import CTkMessagebox

def salir():
    # 1. Crear la alerta
    # option_1, option_2 definen los botones
    msg = CTkMessagebox(title="Salir", message="¿Estás seguro que deseas cerrar el programa?",
                        icon="question", option_1="Cancelar", option_2="No", option_3="Sí")
    
    # 2. Capturar la respuesta
    respuesta = msg.get()
    
    print(f"El usuario eligió: {respuesta}")
    
    if respuesta == "Sí":
        app.destroy() # Cerrar la app
    else:
        print("Operación de cierre cancelada.")

app = ctk.CTk()
app.geometry("300x150")

btn = ctk.CTkButton(app, text="Cerrar Aplicación", command=salir)
btn.place(relx=0.5, rely=0.5, anchor="center")

app.mainloop()
```

##### Ejemplo 3: Integración en una Clase (POO)

Así es como lo usarías en una aplicación real estructurada.

```python
import customtkinter as ctk
from CTkMessagebox import CTkMessagebox

class SistemaLogin(ctk.CTk):
    def __init__(self):
        super().__init__()
        self.title("Sistema v1.0")
        self.geometry("400x300")
        
        self.entry_user = ctk.CTkEntry(self, placeholder_text="Usuario")
        self.entry_user.pack(pady=20)
        
        self.entry_pass = ctk.CTkEntry(self, placeholder_text="Contraseña", show="*")
        self.entry_pass.pack(pady=10)
        
        self.btn_login = ctk.CTkButton(self, text="Ingresar", command=self.validar_login)
        self.btn_login.pack(pady=20)

    def validar_login(self):
        usuario = self.entry_user.get()
        password = self.entry_pass.get()
        
        # Simulación de validación
        if usuario == "admin" and password == "1234":
            # Mensaje de Éxito
            CTkMessagebox(title="Bienvenido", message=f"Hola {usuario}, has entrado al sistema.", icon="check")
            # Aquí abrirías la siguiente ventana...
        else:
            # Mensaje de Error
            CTkMessagebox(title="Error de Acceso", message="Usuario o contraseña incorrectos.", icon="cancel")

if __name__ == "__main__":
    app = SistemaLogin()
    app.mainloop()
```

### Comparativa: `tkinter.messagebox` vs `CTkMessagebox`

| Característica  | `tkinter.messagebox` (Nativo)        | `CTkMessagebox` (Librería Externa)     |
| :-------------- | :----------------------------------- | :------------------------------------- |
| **Estética**    | Ventana gris estilo Windows 98/XP.   | Moderna, redondeada, minimalista.      |
| **Tema Oscuro** | No soporta (siempre es blanca/gris). | Sí, cambia automáticamente con la app. |
| **Iconos**      | Estándar del sistema operativo.      | Iconos vectoriales modernos.           |
| **Código**      | `messagebox.askyesno(...)`           | `CTkMessagebox(option_1="Yes", ...)`   |
| **Integración** | Se siente "ajeno" a CustomTkinter.   | Se siente nativo y coherente.          |

### Resumen del Stack de Escritorio

Ahora tu aplicación de escritorio es perfecta visualmente:

1.  **Ventana Principal:** `customtkinter` (Diseño moderno).
2.  **Popups/Alertas:** `CTkMessagebox` (Feedback al usuario).
3.  **Lógica:** `threading` (Para que no se congele).
4.  **Backend:** `sqlite3` / `requests` (Datos y Redes).
