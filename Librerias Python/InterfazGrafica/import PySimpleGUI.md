### PySimpleUi

`PySimpleGUI` (a menudo importada como `sg`) permite crear ventanas usando listas de Python.

**El Concepto "Layout" (Diseño):**
En lugar de coordenadas complejas (x=10, y=50), aquí diseñas la ventana como si fuera una **tabla de filas y columnas** (una lista de listas).

```python
layout = [
    [ Texto, Input ],  # Fila 1
    [ Botón, Botón ]   # Fila 2
]
```

**Nota Importante:**
Desde 2024, **PySimpleGUI 5** requiere registro (es gratis para uso personal/hobbista, pero de pago para empresas).
*   Si quieres usar la última versión oficial: `pip install PySimpleGUI`
*   Si prefieres la versión totalmente libre (fork de la v4): `pip install FreeSimpleGUI` (funciona igual, solo cambias el import).

### Los 3 Pilares

Para que una app funcione, siempre necesitas estos tres pasos:

1.  **Layout:** Definir qué elementos (Widgets) tendrá la ventana.
2.  **Window:** Crear la ventana con ese layout.
3.  **Event Loop:** Un bucle `while True` que espera a que el usuario haga algo (clic, escribir, cerrar).

### Elementos Principales (Widgets)

Todos empiezan con `sg.` (si importas `import PySimpleGUI as sg`).

| Elemento                   | Descripción                                                                           |
| :------------------------- | :------------------------------------------------------------------------------------ |
| `sg.Text("Hola")`          | Etiqueta de texto estático (Labels).                                                  |
| `sg.Input(key="-IN-")`     | Campo para que el usuario escriba (Entry). La `key` es vital para leerlo.             |
| `sg.Button("Ok")`          | Botón simple. El texto del botón es su evento.                                        |
| `sg.Multiline()`           | Área de texto grande (como un Notepad).                                               |
| `sg.Listbox(values=[...])` | Lista seleccionable.                                                                  |
| `sg.FileBrowse()`          | Botón mágico que abre el explorador de archivos y escribe la ruta en el Input vecino. |
| `sg.ProgressBar()`         | Barra de carga visual.                                                                |

### Ejemplos de Código

##### Ejemplo 1: Tu Primera Ventana ("Hola Mundo")

Fíjate en la estructura: Layout -> Window -> Loop -> Close.

```python
import PySimpleGUI as sg

# 1. Definir el tema (Colores)
sg.theme('DarkAmber') 

# 2. Definir el diseño (Lista de Listas)
layout = [
    [sg.Text('Escribe tu nombre:'), sg.InputText(key='-NOMBRE-')],
    [sg.Button('Saludar'), sg.Button('Salir')]
]

# 3. Crear la ventana
window = sg.Window('Mi Primera App', layout)

# 4. El Bucle de Eventos (Event Loop)
while True:
    # window.read() detiene el código hasta que pasa algo
    event, values = window.read()
    
    # Si cierran la ventana o dan clic en Salir
    if event == sg.WIN_CLOSED or event == 'Salir':
        break
        
    # Si dan clic en Saludar
    if event == 'Saludar':
        # values es un diccionario con los datos de los inputs
        nombre = values['-NOMBRE-']
        sg.popup(f'¡Hola, {nombre}! Bienvenido al GUI.')

# 5. Cerrar limpiamente
window.close()
```

##### Ejemplo 2: Un "Compresor de Archivos" Real

Vamos a unir lo que aprendiste de **`zipfile`** y **`os`** con una interfaz gráfica.

```python
import PySimpleGUI as sg
import zipfile
import os

# Función lógica (Backend)
def comprimir(archivo_origen, nombre_zip):
    if not archivo_origen or not nombre_zip:
        return "Faltan datos."
        
    with zipfile.ZipFile(nombre_zip, 'w', zipfile.ZIP_DEFLATED) as zf:
        # arcname= es para que no guarde toda la ruta de carpetas
        zf.write(archivo_origen, arcname=os.path.basename(archivo_origen))
    return "¡Éxito!"

# Interfaz (Frontend)
sg.theme('BluePurple')

layout = [
    [sg.Text("Selecciona archivo a comprimir:")],
    # Input visible + Botón FileBrowse invisible que llena el input
    [sg.Input(key="-ARCHIVO-"), sg.FileBrowse("Buscar")],
    
    [sg.Text("Nombre del ZIP destino:")],
    [sg.Input(key="-DESTINO-"), sg.FileSaveAs("Guardar como", file_types=(("ZIP", "*.zip"),))],
    
    [sg.Button("Comprimir", size=(10, 1)), sg.Exit()]
]

window = sg.Window("Zipper Pro 3000", layout)

while True:
    event, values = window.read()
    
    if event in (sg.WIN_CLOSED, "Exit"):
        break
        
    if event == "Comprimir":
        resultado = comprimir(values["-ARCHIVO-"], values["-DESTINO-"])
        sg.popup(resultado)

window.close()
```

##### Ejemplo 3: Actualizar la ventana en tiempo real (`window[key].update`)

A veces quieres cambiar un texto o una imagen sin cerrar la ventana.

```python
import PySimpleGUI as sg

layout = [
    [sg.Text("Contador: 0", key="-TEXTO-", font=("Helvetica", 20))],
    [sg.Button("Sumar"), sg.Button("Restar")]
]

window = sg.Window("Contador", layout)
contador = 0

while True:
    event, values = window.read()
    
    if event == sg.WIN_CLOSED:
        break
    
    if event == "Sumar":
        contador += 1
    elif event == "Restar":
        contador -= 1
        
    # AQUÍ ESTÁ LA MAGIA: Actualizamos el elemento buscando por su Key
    window["-TEXTO-"].update(f"Contador: {contador}")

window.close()
```

### Integración Stack (El Problema del Bloqueo)

Aquí hay un concepto crítico.
`window.read()` espera eventos. Pero si ejecutas una tarea pesada (como descargar un archivo grande con `httpx` o escanear puertos con `scapy`) dentro del `while True`, **la ventana se congelará** (se pondrá blanca y dirá "No responde").

**Solución:** Usar **`threading`** (que ya aprendiste).

```python
import PySimpleGUI as sg
import time
import threading

def tarea_larga(window):
    # Simulamos descarga
    for i in range(100):
        time.sleep(0.05)
        # Enviamos un evento personalizado a la ventana principal
        window.write_event_value('-PROGRESO-', i+1)
    window.write_event_value('-FIN-', 'Descarga completa')

layout = [[sg.Text("Esperando action...")],
          [sg.ProgressBar(100, orientation='h', size=(20, 20), key='-BARRA-')],
          [sg.Button('Iniciar Descarga')]]

window = sg.Window('Async GUI', layout)

while True:
    event, values = window.read()
    
    if event == sg.WIN_CLOSED:
        break
        
    if event == 'Iniciar Descarga':
        # Lanzamos el hilo para no congelar la ventana
        threading.Thread(target=tarea_larga, args=(window,), daemon=True).start()
        
    # Escuchamos los eventos que manda el hilo
    if event == '-PROGRESO-':
        window['-BARRA-'].update(values[event]) # values[event] trae el número (i+1)
        
    if event == '-FIN-':
        sg.popup("¡Terminado!")

window.close()
```

### Resumen

`PySimpleGUI` es el puente entre tus scripts de backend y el usuario final de escritorio.

*   **Ventaja:** Extremadamente rápido de programar.
*   **Desventaja:** No es tan potente como Qt (para apps tipo Photoshop) o Web (para apps mundiales).
*   **Uso ideal:** Herramientas internas, scripts de automatización para compañeros de trabajo, prototipos rápidos.

Usos:

1.  **Web:** FastAPI (Backend) + HTML/JS (Frontend).
2.  **Terminal:** Typer + Rich.
3.  **Escritorio:** PySimpleGUI.

