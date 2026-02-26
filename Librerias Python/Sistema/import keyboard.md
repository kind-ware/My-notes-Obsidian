### Keyboard

`keyboard` permite conectar funciones de Python a teclas globales.
"Global" significa que **tu script funciona aunque la ventana de Python esté minimizada**. Puedes estar en Excel, en Chrome o en un videojuego, y si presionas la tecla configurada, Python reaccionará.

### Funciones Principales

Las dividimos en dos: **Escribir** (Output) y **Escuchar** (Input).

##### Simular Teclado (Output)
| Función                   | Descripción                                                               |
| :------------------------ | :------------------------------------------------------------------------ |
| `keyboard.write("texto")` | Escribe el texto tal cual (como si lo teclearas muy rápido).              |
| `keyboard.send("teclas")` | Presiona y suelta una combinación. Ej: `"ctrl+s"`, `"alt+f4"`, `"enter"`. |
| `keyboard.press("k")`     | Mantiene presionada una tecla (sin soltarla).                             |
| `keyboard.release("k")`   | Suelta la tecla.                                                          |

##### Escuchar Teclado (Input)

| Función                             | Descripción                                                                |
| :---------------------------------- | :------------------------------------------------------------------------- |
| `keyboard.add_hotkey("comb", func)` | Ejecuta una función cuando presionas la combinación. Ej: `"ctrl+shift+a"`. |
| `keyboard.wait("tecla")`            | Pausa el programa hasta que se presione esa tecla.                         |
| `keyboard.is_pressed("tecla")`      | Devuelve `True` si la tecla está siendo presionada en ese instante.        |
| `keyboard.record(until="esc")`      | Graba todo lo que escribes hasta que presionas "esc".                      |
| `keyboard.play(grabacion)`          | Reproduce lo que grabaste (Macro).                                         |

## 3. Ejemplos de Código

##### Ejemplo 1: Escribir automáticamente (`write` y `send`)

Imagina que tienes que llenar el mismo formulario 100 veces.
*Nota: Ponemos un `sleep` al inicio para darte tiempo de cambiar de ventana (ej: abrir el Bloc de Notas).*

```python
import keyboard
import time

print("Tienes 5 segundos para abrir el Bloc de Notas...")
time.sleep(5)

# 1. Escribir texto
keyboard.write("Hola, esto lo está escribiendo Python.\n")

# 2. Simular retardo (para que parezca humano)
keyboard.write("Escribiendo lento...", delay=0.1)

# 3. Comandos especiales
keyboard.send("enter")
keyboard.write("Ahora voy a guardar (falso)...")
# keyboard.send("ctrl+s") # Cuidado, esto guardaría de verdad
```

##### Ejemplo 2: Hotkeys Globales (Tu propio Stream Deck)

Este script corre en segundo plano y ejecuta acciones cuando presionas teclas, sin importar qué programa estés usando.

```python
import keyboard

def lanzar_mensaje():
    print("¡Has presionado el atajo secreto!")

def cerrar_programa():
    print("Saliendo...")
    # Esto es necesario para romper el wait() de abajo si quisieras
    pass

print("Presiona 'ctrl+alt+p' para probar.")
print("Presiona 'esc' para salir.")

# 1. Configurar el Hotkey
# Cuando presiones Ctrl + Alt + P, se ejecuta la función
keyboard.add_hotkey('ctrl+alt+p', lanzar_mensaje)

# 2. Bloquear el script para que no termine
# El programa se queda "durmiendo" hasta que presiones Esc
keyboard.wait('esc')
```

##### Ejemplo 3: Grabadora de Macros (`record` y `play`)

Un "Macro" es grabar una secuencia de acciones y repetirla.

```python
import keyboard

print("Presiona 'enter' para empezar a grabar.")
keyboard.wait('enter')

print("[+] Grabando... Escribe algo y presiona 'esc' para terminar.")

# Graba todos los eventos (teclas bajadas, subidas, tiempos)
eventos = keyboard.record(until='esc')

print(f"Grabación finalizada. {len(eventos)} eventos capturados.")
print("Presiona 'enter' para reproducir la grabación.")
keyboard.wait('enter')

print("[+] Reproduciendo...")
# speed_factor=2 hace que se reproduzca al doble de velocidad
keyboard.play(eventos, speed_factor=1)
```

##### Ejemplo 4: Bloquear Teclas (Cuidado ☠️)

Puedes usar esto para "Modo Kiosco" (evitar que el usuario salga) o bromas.
**Advertencia:** Asegúrate de saber cómo cerrar tu script (Ctrl+C en la terminal) si algo sale mal.

```python
import keyboard
import time

print("Voy a bloquear la tecla 'a' por 5 segundos.")

# Bloqueamos la tecla. Si la presionas, no pasará nada en ninguna app.
keyboard.block_key('a')

for i in range(5, 0, -1):
    print(i)
    time.sleep(1)

# IMPORTANTE: Desbloquearla
keyboard.unblock_key('a')
print("Tecla 'a' liberada.")
```

### Peligros y Consideraciones

1.  **Keyloggers (Ética):** Con `keyboard.record()` es trivial hacer un keylogger (programa que roba contraseñas). **No lo uses para fines maliciosos.** Los antivirus suelen detectar scripts que usan esta librería como sospechosos.
2.  **Permisos:** Si tu script falla y dice "Permission denied", necesitas ejecutar la terminal como Administrador.
3.  **Bucle Infinito:** Si configuras `keyboard.send('enter')` dentro de un hotkey que se activa con `enter`, crearás un bucle infinito que colgará tu PC.

### Integración 

¿Cómo encaja esto con lo que ya sabes?

1.  **Con `threading`**:
    *   `keyboard.wait()` bloquea el programa. Si quieres escuchar teclas *mientras* haces otra cosa (como descargar archivos con `httpx`), debes poner el listener de teclado en un hilo (`threading.Thread`).

2.  **Con `typer` / `rich`**:
    *   Puedes hacer una CLI bonita que diga: `[bold green]Presiona F1 para iniciar el servidor[/]`.

3.  **Con `sqlite3`**:
    *   Puedes hacer un script que funcione en segundo plano y cada vez que presiones `Ctrl+C` (copiar), guarde el texto del portapapeles en una base de datos histórica.

