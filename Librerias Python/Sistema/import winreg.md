## WinReg

`winreg` (anteriormente `_winreg` en Python 2) expone la API del Registro de Windows a Python.

Permite:
1.  **Leer** configuraciones del sistema (ej: ruta de instalación de un programa).
2.  **Escribir** tus propias configuraciones (ej: guardar opciones de tu app).
3.  **Añadir** programas al inicio de Windows.

## Conceptos Clave (Hives y Claves)

El registro se divide en carpetas principales llamadas **Hives** (Colmenas). Las dos que usarás son:

1.  **`HKEY_CURRENT_USER` (HKCU):** Configuraciones solo para **tu usuario**.
    *   *Seguro:* Puedes escribir aquí sin ser Administrador.
    *   *Uso:* Guardar el tema oscuro/claro de tu app `customtkinter`.
2.  **`HKEY_LOCAL_MACHINE` (HKLM):** Configuraciones para **toda la PC**.
    *   *Peligroso:* Requiere permisos de Administrador.
    *   *Uso:* Instalar drivers, servicios del sistema.

**Tipos de Datos:**
*   `REG_SZ`: Texto (String).
*   `REG_DWORD`: Número (Entero de 32 bits).
*   `REG_BINARY`: Datos binarios.

## Funciones Principales

Casi siempre necesitas abrir una llave (`OpenKey`) antes de hacer nada.

| Función                                   | Descripción                                                                               |
| :---------------------------------------- | :---------------------------------------------------------------------------------------- |
| `ConnectRegistry(pc, hive)`               | Conecta al registro (usualmente `None` para PC local).                                    |
| `OpenKey(hive, ruta, ...)`                | Abre una carpeta existente. Necesitas especificar permisos (ej: `KEY_READ`, `KEY_WRITE`). |
| `CreateKey(hive, ruta)`                   | Crea una carpeta nueva (o la abre si ya existe).                                          |
| `QueryValueEx(key, nombre)`               | **Lee** el valor de un dato. Devuelve `(valor, tipo)`.                                    |
| `SetValueEx(key, nombre, 0, tipo, valor)` | **Escribe** o actualiza un dato.                                                          |
| `DeleteKey(key, ruta)`                    | Borra una carpeta (debe estar vacía).                                                     |
| `DeleteValue(key, nombre)`                | Borra un dato específico dentro de una carpeta.                                           |

## Ejemplos de Código

#### Ejemplo 1: Escribir en el Registro (Crear Configuración)

Vamos a guardar una configuración de prueba en el usuario actual. Esto es seguro y no rompe Windows.

```python
import winreg

# Ruta donde guardaremos nuestros datos
ruta = r"Software\MiAppPython"

try:
    # 1. Crear (o abrir) la llave en HKEY_CURRENT_USER
    # winreg.KEY_WRITE es necesario para poder guardar datos
    key = winreg.CreateKey(winreg.HKEY_CURRENT_USER, ruta)

    # 2. Guardar valores
    # SetValueEx(llave, nombre_dato, reservado, tipo_dato, valor)
    
    # Guardamos un texto
    winreg.SetValueEx(key, "Tema", 0, winreg.REG_SZ, "Oscuro")
    
    # Guardamos un número
    winreg.SetValueEx(key, "Volumen", 0, winreg.REG_DWORD, 80)
    
    # 3. Cerrar (Vital si no usas 'with')
    winreg.CloseKey(key)
    
    print(f"Datos guardados exitosamente en HKCU\\{ruta}")

except PermissionError:
    print("Error: No tienes permisos. Ejecuta como Administrador.")
except Exception as e:
    print(f"Error inesperado: {e}")
```

#### Ejemplo 2: Leer del Registro

Ahora vamos a leer lo que acabamos de escribir. Usaremos el bloque `with` que es más seguro (cierra la llave automáticamente).

```python
import winreg

ruta = r"Software\MiAppPython"

try:
    # 1. Abrir la llave con permiso de LECTURA (KEY_READ)
    with winreg.OpenKey(winreg.HKEY_CURRENT_USER, ruta, 0, winreg.KEY_READ) as key:
        
        # 2. Leer valores
        # QueryValueEx devuelve una tupla: (valor, tipo_dato)
        tema, tipo_tema = winreg.QueryValueEx(key, "Tema")
        volumen, tipo_vol = winreg.QueryValueEx(key, "Volumen")
        
        print("--- Configuración Leída ---")
        print(f"Tema: {tema} (Tipo: {tipo_tema})")
        print(f"Volumen: {volumen} (Tipo: {tipo_vol})")

except FileNotFoundError:
    print("La configuración no existe. Ejecuta el Ejemplo 1 primero.")
```

#### Ejemplo 3: Iniciar tu Script con Windows (Startup)

Este es el truco clásico para que tu programa arranque solo al encender la PC.
*Agregamos una entrada en `Software\Microsoft\Windows\CurrentVersion\Run`.*

```python
import winreg
import sys
import os

# Ruta del ejecutable actual (o de tu script .py)
ruta_exe = sys.executable 
# Si tuvieras un script: ruta_script = os.path.abspath("mi_script.py")

ruta_run = r"Software\Microsoft\Windows\CurrentVersion\Run"

def agregar_al_inicio():
    try:
        key = winreg.OpenKey(winreg.HKEY_CURRENT_USER, ruta_run, 0, winreg.KEY_WRITE)
        
        # Nombre de la entrada: "MiBotPython"
        winreg.SetValueEx(key, "MiBotPython", 0, winreg.REG_SZ, ruta_exe)
        
        winreg.CloseKey(key)
        print("¡Programa añadido al inicio de Windows!")
        
    except Exception as e:
        print(f"Error: {e}")

def quitar_del_inicio():
    try:
        key = winreg.OpenKey(winreg.HKEY_CURRENT_USER, ruta_run, 0, winreg.KEY_WRITE)
        
        # Borrar la entrada
        winreg.DeleteValue(key, "MiBotPython")
        
        winreg.CloseKey(key)
        print("Programa eliminado del inicio.")
        
    except FileNotFoundError:
        print("El programa no estaba en el inicio.")

# Descomenta la que quieras probar:
# agregar_al_inicio()
# quitar_del_inicio()
```

#### Ejemplo 4: Listar Sub-Claves (Exploración)

¿Quieres ver qué software tienes instalado según Windows? (Es un ejemplo simplificado).

```python
import winreg

ruta_software = r"Software"

print(f"Explorando HKCU\\{ruta_software}...")

with winreg.OpenKey(winreg.HKEY_CURRENT_USER, ruta_software) as key:
    i = 0
    while True:
        try:
            # EnumKey obtiene el nombre de la subcarpeta en la posición 'i'
            subcarpeta = winreg.EnumKey(key, i)
            print(f"- {subcarpeta}")
            i += 1
        except OSError:
            # Cuando 'i' se pasa del total de carpetas, Windows lanza error
            # y rompemos el bucle.
            break
```

## Integración con Stack

¿Cómo encaja `winreg` con todo lo que has aprendido?

1.  **Validación de Sistema (`sys`, `platform`):**
    Antes de importar `winreg`, siempre debes verificar el SO, o tu código fallará en Linux/Mac de tus compañeros.

```python
    import sys
    if sys.platform != "win32":
        print("Este script solo corre en Windows")
        sys.exit()
    import winreg
```

2.  **Instaladores (`PySimpleGUI` / `customtkinter`):**
    Si creas un instalador para tu app, usarás `winreg` para guardar dónde instaló el usuario la aplicación, o para asociar una extensión de archivo (ej: que los `.miapp` se abran con tu programa).

3.  **Seguridad (`pycryptodome`):**
    Puedes leer una clave encriptada del registro, desencriptarla con una llave maestra y usarla. (Aunque `.env` suele ser mejor para desarrollo, en apps de escritorio nativas el Registro es muy común).

4.  **Hacking Ético / SysAdmin:**
    Los administradores usan `winreg` para auditar máquinas, ver qué USBs se han conectado o verificar si hay malware en la carpeta "Run".

#### Resumen

`winreg` es una herramienta de poder para el entorno **Windows**.

*   **Usos:** Configuración persistente, Auto-arranque, Instalación.
*   **Peligro:** Alto. Siempre haz backup del registro (`regedit.exe` -> Exportar) antes de jugar con claves importantes.

sys.executable solo funciona con el PE executable