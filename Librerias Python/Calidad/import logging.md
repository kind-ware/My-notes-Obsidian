### Logging

El módulo `logging` es la herramienta estándar para rastrear eventos que ocurren cuando el software se ejecuta.
Permite generar mensajes de estado, advertencias y errores, los cuales pueden ser mostrados en consola, guardados en archivos `.log` o enviados por correo, todo al mismo tiempo.

### Los 5 Niveles de Gravedad (Hierarchy)

`logging` no trata todos los mensajes igual. Usa niveles para filtrar la importancia:

| Nivel        | Valor | Cuándo usarlo                                                                                        |
| :----------- | :---- | :--------------------------------------------------------------------------------------------------- |
| **DEBUG**    | 10    | Información detallada, solo útil para diagnosticar problemas (variables, pasos internos).            |
| **INFO**     | 20    | Confirmación de que las cosas funcionan como se espera ("Iniciando sesión", "Correo enviado").       |
| **WARNING**  | 30    | **(Por defecto)** Algo inesperado pasó, pero el programa sigue funcionando (ej: "Disco casi lleno"). |
| **ERROR**    | 40    | Debido a un problema grave, el programa no pudo realizar una función específica.                     |
| **CRITICAL** | 50    | Error fatal. El programa no puede continuar ejecutándose.                                            |

*Nota: Si configuras el nivel en `INFO`, verás INFO, WARNING, ERROR y CRITICAL. Los mensajes DEBUG serán ignorados.*

### Ejemplos de codigo(1): Configuración Básica (`basicConfig`)

Para empezar rápido, configuramos todo en una sola línea.

##### Ejemplo 1: logging vs print (Consola)

Por defecto, `logging` solo muestra de WARNING para arriba.

```python
import logging

# Configuración por defecto
logging.warning("¡Cuidado! Algo podría fallar.")  # Se imprime
logging.critical("¡El sistema se cayó!")          # Se imprime
logging.info("El sistema está estable.")          # NO se imprime (Nivel muy bajo)
```

##### Ejemplo 2: Guardar en un Archivo (`filename`)

Aquí es donde ocurre la magia. Tu script puede correr de noche, y al día siguiente lees el archivo para ver qué pasó.

```python
import logging

# Configuración:
# - filename: Nombre del archivo donde guardar.
# - level: Capturar desde DEBUG en adelante (todo).
# - filemode: 'w' (sobrescribir cada vez) o 'a' (append/agregar al final).
logging.basicConfig(filename='app_reporte.log', level=logging.DEBUG, filemode='w')

logging.debug("Iniciando diagnóstico...")
logging.info("Conectando a base de datos...")
logging.warning("La conexión es lenta.")
logging.error("No se pudo guardar el usuario.")

print("Revisa el archivo 'app_reporte.log', la consola está limpia.")
```

##### Ejemplo 3: El Formato Profesional (`format`)

`logging` puede insertar datos automáticamente, como la hora exacta, el nivel y el mensaje.

```python
import logging

# Definimos el formato:
# %(asctime)s  -> Fecha y hora automática
# %(levelname)s -> INFO, ERROR, etc.
# %(message)s   -> Tu mensaje
formato = '%(asctime)s - %(levelname)s - %(message)s'

logging.basicConfig(level=logging.INFO, format=formato, datefmt='%H:%M:%S')

logging.info("Servidor iniciado.")
logging.error("Intento de acceso denegado.")

# Salida esperada:
# 14:30:05 - INFO - Servidor iniciado.
# 14:30:06 - ERROR - Intento de acceso denegado.
```

### Ejemplos de codigo(2): Capturando Excepciones (El uso maestro)

¿Recuerdas los bloques `try/except`?
Si usas `print(e)`, solo ves el error corto.
Si usas **`logging.exception("msg")`**, Python guarda automáticamente toda la traza del error (el Traceback completo) en el archivo log.

```python
import logging

logging.basicConfig(filename='errores.log', level=logging.ERROR)

try:
    # Simulamos un error matemático
    resultado = 10 / 0
except ZeroDivisionError:
    # .exception agrega automáticamente el stack trace al log
    logging.exception("¡Ocurrió una división por cero!")
    
print("El programa no se detuvo, pero el error quedó registrado.")
```
*En `errores.log` verás no solo el mensaje, sino la línea exacta del código donde falló.*

### Ejemplos de codigo(3): (Loggers Nombrados)

Cuando tu programa crece y tienes varios archivos (`main.py`, `database.py`, `network.py`), usar `logging.basicConfig` en todos lados causa conflictos.
La forma correcta es crear un "Logger" para cada módulo.

```python
import logging

# 1. Crear un logger personalizado
logger = logging.getLogger("Modulo_Facturacion")
logger.setLevel(logging.INFO)

# 2. Crear un manejador (Handler) para consola
console_handler = logging.StreamHandler()

# 3. Crear un formato
formatter = logging.Formatter('%(name)s - %(levelname)s -> %(message)s')
console_handler.setFormatter(formatter)

# 4. Añadir el handler al logger
logger.addHandler(console_handler)

# Uso
logger.info("Creando factura...")
# Salida: Modulo_Facturacion - INFO -> Creando factura...
```


### Integración con otras librerias 

Mira cómo `logging` mejora todo lo que has aprendido:

1.  **Con `threading`:**
    *   Si usas `print` en hilos, las letras se mezclan. `logging` es "thread-safe" (hilo-seguro), garantiza que las líneas no se corten entre sí.
2.  **Con `scapy` / `requests`:**
    *   En lugar de `print(f"Paquete recibido: {ip}")`, usas `logging.info(f"Paquete recibido: {ip}")`. Así puedes dejar el escáner corriendo días y revisar el archivo luego.
3.  **Con `time` / `datetime`:**
    *   Ya no necesitas calcular `datetime.now()` manualmente para cada mensaje. `logging` lo hace solo con `%(asctime)s`.