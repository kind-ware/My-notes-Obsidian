### Argparse

El módulo `argparse` facilita la escritura de herramientas de línea de comandos amigables.
Tú defines qué argumentos requiere tu programa, y `argparse` se encarga de averiguar cómo sacarlos de `sys.argv`.

**Sus superpoderes:**
*   Genera mensajes de ayuda (`-h` o `--help`) automáticamente.
*   Convierte tipos de datos (ej: de texto a número) automáticamente.
*   Lanza errores si el usuario ingresa opciones inválidas.
*   Maneja argumentos obligatorios y opcionales.

### El Flujo de Trabajo (La Receta)

A diferencia de otras librerías, `argparse` siempre sigue estos 3 pasos:

1.  **Crear el parser:** Instanciar el objeto que manejará la lógica.
2.  **Agregar argumentos:** Definir qué esperamos recibir (`.add_argument`).
3.  **Parsear:** Procesar la entrada del usuario (`.parse_args`).

### Funciones Principales

##### Inicialización

| Función                                    | Descripción                                                                                 |
| :----------------------------------------- | :------------------------------------------------------------------------------------------ |
| `argparse.ArgumentParser(description=...)` | Crea el objeto principal. El texto en `description` aparecerá cuando el usuario pida ayuda. |

##### Agregar Argumentos (`add_argument`)
Esta es la función más importante. Acepta muchos parámetros:

| Parámetro  | Ejemplo                        | Descripción                                                                          |
| :--------- | :----------------------------- | :----------------------------------------------------------------------------------- |
| **Nombre** | `'archivo'` o `'-f', '--file'` | Si empieza con guion (`-`), es **Opcional**. Si no, es **Posicional** (Obligatorio). |
| `help`     | `'Nombre del usuario'`         | El texto de ayuda que explica para qué sirve.                                        |
| `type`     | `int`, `float`                 | Convierte la entrada automáticamente (por defecto es `str`).                         |
| `default`  | `'valor_defecto'`              | El valor que se usa si el usuario no escribe nada.                                   |
| `required` | `True`                         | Fuerza a que un argumento opcional sea obligatorio.                                  |
| `choices`  | `['txt', 'json']`              | Solo permite valores dentro de esa lista.                                            |
| `action`   | `'store_true'`                 | Se usa para banderas (flags) sin valor. Si está presente es `True`, si no, `False`.  |

##### Procesar

| Función                      | Descripción                                                                                  |
| :--------------------------- | :------------------------------------------------------------------------------------------- |
| `args = parser.parse_args()` | Lee lo que el usuario escribió en la terminal y lo guarda en un objeto (ej: `args.archivo`). |

### Ejemplos de Código

##### Ejemplo 1: El programa más básico (Posicional)

Imagina un script que calcula el cuadrado de un número.
*Uso esperado:* `python script.py 5`

```python
import argparse

# 1. Crear el parser
parser = argparse.ArgumentParser(description="Calcula el cuadrado de un número.")

# 2. Agregar argumento (Posicional: es obligatorio y sin guiones)
parser.add_argument("numero", type=int, help="El número base a calcular")

# 3. Parsear
args = parser.parse_args()

# 4. Usar el dato
resultado = args.numero ** 2
print(f"El cuadrado de {args.numero} es {resultado}")
```

##### Ejemplo 2: Argumentos Opcionales y Banderas

Vamos a crear un script de saludo que acepta nombre y una bandera para gritar.
*Uso esperado:* `python script.py --nombre Juan --gritar`

```python
import argparse

parser = argparse.ArgumentParser(description="Script de saludos personalizado.")

# Argumento opcional con valor (requiere escribir --nombre ALGO)
# El 'default' se usa si el usuario no pone nada.
parser.add_argument("--nombre", type=str, default="Mundo", help="A quién saludar")

# Bandera booleana (Switch)
# action="store_true" significa: si pones --gritar, vale True. Si no, False.
parser.add_argument("--gritar", action="store_true", help="Imprime el saludo en mayúsculas")

args = parser.parse_args()

mensaje = f"Hola, {args.nombre}!"

if args.gritar:
    print(mensaje.upper())
else:
    print(mensaje)
```

##### Ejemplo 3: Un CLI "Profesional" (Restricciones y Tipos)

Vamos a simular una herramienta que procesa archivos, usando lo que sabemos de otras librerías.
*Uso:* `python script.py data.csv --formato json --lineas 10`

```python
import argparse

parser = argparse.ArgumentParser(description="Convertidor de archivos v1.0")

# 1. Archivo de entrada (Obligatorio)
parser.add_argument("archivo", help="Ruta del archivo a leer")

# 2. Formato de salida (Opcional pero restringido a opciones fijas)
parser.add_argument("-f", "--formato", 
                    choices=['json', 'csv', 'xml'], 
                    default='json',
                    help="Formato de salida deseado")

# 3. Número de líneas (Opcional, debe ser entero)
parser.add_argument("-n", "--lineas", 
                    type=int, 
                    default=5,
                    help="Cantidad de líneas a procesar")

args = parser.parse_args()

print(f"--- Iniciando Proceso ---")
print(f"Leyendo archivo: {args.archivo}")
print(f"Convirtiendo a:  {args.formato.upper()}")
print(f"Procesando:      {args.lineas} líneas")
# Aquí iría tu lógica con 'os', 'json', etc.
```

### Diferencia Clave: `sys.argv` vs `argparse`

¿Por qué dejar de usar `sys.argv`?

**Con `sys.argv` (La forma difícil):**
```python
import sys
# Si el usuario olvida el argumento, el programa explota con IndexError
if len(sys.argv) > 1:
    nombre = sys.argv[1]
else:
    nombre = "Mundo"
```

**Con `argparse` (La forma correcta):**
```python
# Maneja errores, ayuda, tipos y valores por defecto solo.
parser.add_argument("--nombre", default="Mundo")
```

