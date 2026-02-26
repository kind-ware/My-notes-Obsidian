### Tkinter
`tkinter` es la interfaz estándar de Python para el kit de herramientas GUI Tcl/Tk.
Es ligera, estable y funciona en cualquier sistema operativo sin instalar nada extra.

**Filosofía:**

Funciona mediante una **Jerarquía de Widgets** (elementos).
1.  Creas la **Ventana Raíz** (`root`).
2.  Creas un **Widget** (ej: un Botón) y le dices quién es su padre (`root`).
3.  Usas un **Gestor de Geometría** para decirle dónde colocarse (`pack`, `grid`, `place`).
4.  Inicias el **Bucle Principal** (`mainloop`).

### Los 3 Gestores de Geometría

Este es el concepto más difícil y más importante. Si no eliges uno, el widget no aparece.

| Gestor     | Descripción                                              | Cuándo usarlo                                                       |
| :--------- | :------------------------------------------------------- | :------------------------------------------------------------------ |
| `.pack()`  | Apila los elementos (Arriba, Abajo, Izquierda, Derecha). | Diseños simples, barras de herramientas, listas verticales.         |
| `.grid()`  | Divide la ventana en filas y columnas (como Excel).      | Formularios, calculadoras, diseños estructurados. **El más usado.** |
| `.place()` | Coordenadas exactas en píxeles (x=50, y=100).            | Casi nunca (no es responsivo si redimensionas la ventana).          |

### Widgets Principales (`tk` vs `ttk`)

Python incluye dos sets de widgets:
*   **`tk` (Clásicos):** Se ven viejos (estilo Windows 95).
*   **`ttk` (Themed Tkinter):** Se ven modernos y nativos del sistema operativo. **Usa estos siempre que puedas.**

| Widget         | Descripción                                                                     |
| :------------- | :------------------------------------------------------------------------------ |
| `tk.Tk`        | La ventana principal.                                                           |
| `ttk.Label`    | Texto estático.                                                                 |
| `ttk.Button`   | Botón clickeable.                                                               |
| `ttk.Entry`    | Campo de texto de una línea.                                                    |
| `tk.Text`      | Campo de texto multilínea (Bloc de notas).                                      |
| `ttk.Frame`    | Un contenedor invisible para agrupar otros widgets.                             |
| `tk.StringVar` | Variable especial que actualiza la interfaz automáticamente si cambia su valor. |

### Ejemplos de Código

##### Ejemplo 1: Hola Mundo (`pack`)

Lo mínimo indispensable.

```python
import tkinter as tk
from tkinter import ttk # Importamos los widgets modernos

# 1. Crear ventana raíz
root = tk.Tk()
root.title("Mi Primera App")
root.geometry("300x200")

# 2. Crear widgets
etiqueta = ttk.Label(root, text="¡Hola, Tkinter!")
boton = ttk.Button(root, text="Cerrar", command=root.destroy)

# 3. Empaquetar (Mostrar en pantalla)
# pady=10 agrega espacio vertical
etiqueta.pack(pady=20) 
boton.pack(pady=10)

# 4. Iniciar bucle
root.mainloop()
```

##### Ejemplo 2: Formulario con Grilla (`grid`)

Aquí usamos filas y columnas para alinear etiquetas y campos de entrada.

```python
import tkinter as tk
from tkinter import ttk

def guardar_datos():
    nombre = entrada_nombre.get()
    edad = entrada_edad.get()
    print(f"Guardando: {nombre}, {edad} años")

root = tk.Tk()
root.title("Formulario")

# --- FILA 0 ---
lbl_nombre = ttk.Label(root, text="Nombre:")
lbl_nombre.grid(row=0, column=0, padx=10, pady=10)

entrada_nombre = ttk.Entry(root)
entrada_nombre.grid(row=0, column=1, padx=10, pady=10)

# --- FILA 1 ---
lbl_edad = ttk.Label(root, text="Edad:")
lbl_edad.grid(row=1, column=0, padx=10, pady=10)

entrada_edad = ttk.Entry(root)
entrada_edad.grid(row=1, column=1, padx=10, pady=10)

# --- FILA 2 (Botón que ocupa 2 columnas) ---
btn_guardar = ttk.Button(root, text="Guardar", command=guardar_datos)
# columnspan=2 hace que el botón se centre ocupando el ancho de las 2 columnas
btn_guardar.grid(row=2, column=0, columnspan=2, pady=20)

root.mainloop()
```

### Ejemplo 3: Variables de Control (`StringVar`)

En `tkinter`, para cambiar el texto de una etiqueta dinámicamente, usas variables especiales.

```python
import tkinter as tk
from tkinter import ttk

root = tk.Tk()
root.geometry("300x150")

# Variable especial de Tkinter
texto_dinamico = tk.StringVar()
texto_dinamico.set("Texto inicial")

def cambiar_texto():
    texto_dinamico.set("¡Texto cambiado mágicamente!")

# Vinculamos la etiqueta a la variable
etiqueta = ttk.Label(root, textvariable=texto_dinamico, font=("Arial", 14))
etiqueta.pack(pady=20)

boton = ttk.Button(root, text="Cambiar", command=cambiar_texto)
boton.pack()

root.mainloop()
```

### Ejemplo 4: Programación Orientada a Objetos (Nivel Pro)

Así es como se escriben las aplicaciones reales. Heredamos de `tk.Tk`.

```python
import tkinter as tk
from tkinter import ttk
import tkinter.messagebox as msg

class Aplicacion(tk.Tk):
    def __init__(self):
        super().__init__() # Inicializa tk.Tk
        
        self.title("App Profesional")
        self.geometry("400x300")
        
        # Crear menú
        self.crear_menu()
        
        # Crear contenido principal
        self.lbl = ttk.Label(self, text="Bienvenido", font=("Arial", 20))
        self.lbl.pack(expand=True) # expand=True centra el elemento
        
        self.btn = ttk.Button(self, text="Acción", command=self.mostrar_mensaje)
        self.btn.pack(pady=20)

    def crear_menu(self):
        barra_menu = tk.Menu(self)
        self.config(menu=barra_menu)
        
        menu_archivo = tk.Menu(barra_menu, tearoff=0)
        menu_archivo.add_command(label="Nuevo", command=lambda: print("Nuevo archivo"))
        menu_archivo.add_separator()
        menu_archivo.add_command(label="Salir", command=self.quit)
        
        barra_menu.add_cascade(label="Archivo", menu=menu_archivo)

    def mostrar_mensaje(self):
        msg.showinfo("Información", "Has presionado el botón")

if __name__ == "__main__":
    app = Aplicacion()
    app.mainloop()
```

### Integración con `threading` (Anti-Congelamiento)

Si intentas ejecutar una tarea larga (como un `sleep` o una descarga) en el hilo principal, la ventana se congelará (no podrás moverla ni cerrarla).

```python
import tkinter as tk
from tkinter import ttk
import time
import threading

def tarea_pesada():
    boton.config(state="disabled") # Desactivar botón
    
    for i in range(1, 6):
        time.sleep(1) # Simulamos trabajo duro
        # Actualizamos la etiqueta desde el hilo
        var_estado.set(f"Procesando... {i}/5")
        
    var_estado.set("¡Tarea terminada!")
    boton.config(state="normal") # Reactivar botón

def iniciar_hilo():
    # Creamos un hilo separado para la tarea
    t = threading.Thread(target=tarea_pesada)
    t.start()

root = tk.Tk()
root.geometry("300x150")

var_estado = tk.StringVar(value="Listo para empezar")

lbl = ttk.Label(root, textvariable=var_estado)
lbl.pack(pady=20)

boton = ttk.Button(root, text="Iniciar Tarea Larga", command=iniciar_hilo)
boton.pack()

root.mainloop()
```

### Comparativa Final de GUIs

Ahora que hemos tocado varias librerias UI:

| Característica  | `tkinter`                                                                | `customtkinter`                        | `PySimpleGUI`                            |
| :-------------- | :----------------------------------------------------------------------- | :------------------------------------- | :--------------------------------------- |
| **Nivel**       | Bajo (Base)                                                              | Medio (Wrapper moderno)                | Alto (Wrapper simplificado)              |
| **Código**      | Verboso (`pack`, `grid`, `config`)                                       | Orientado a Objetos                    | Listas (`layout = [[...]]`)              |
| **Estética**    | Antigua (Win 95/XP)                                                      | Moderna (Win 11 / Mac)                 | Funcional pero simple                    |
| **Instalación** | NINGUNA (Viene con Python)                                               | `pip install`                          | `pip install` (Licencia requerida en v5) |
| **Uso Ideal**   | Aprender bases, apps ligeras, empresas con restricciones de instalación. | Apps de escritorio bonitas y modernas. | Scripts rápidos, herramientas internas.  |

### Resumen

Dominar `tkinter` te da el "cinturón negro" en interfaces gráficas.
*   Sabes cómo funciona el **Event Loop**.
*   Sabes cómo funciona el **Geometry Manager**.
*   Entiendes por qué `customtkinter` funciona como funciona (porque hereda de aquí).

