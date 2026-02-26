### Datetime

El módulo `datetime` proporciona clases para manipular fechas y horas.
A diferencia de tratar las fechas como simples textos ("25/12/2023"), este módulo permite tratar el tiempo como **objetos matemáticos**.

Esto te permite responder preguntas como:
*   ¿Qué día será dentro de 30 días?
*   ¿Cuántos segundos pasaron entre el evento A y el evento B?
*   ¿Cómo convierto "2023-01-01" en un objeto real de fecha?
### Clases Principales

El módulo `datetime` no tiene muchas funciones sueltas, sino **Clases** (tipos de datos) que debes instanciar.

| Clase | Descripción | Qué contiene |
| :--- | :--- | :--- |
| `datetime.date` | Maneja solo fechas (calendario). | Año, Mes, Día. |
| `datetime.time` | Maneja solo horas (reloj). | Hora, Minuto, Segundo, Microsegundo. |
| `datetime.datetime` | Combina fecha y hora. Es la más usada. | Año, Mes, Día, Hora, Min., Seg. |
| `datetime.timedelta` | Representa una **duración** o diferencia entre dos fechas. | Días, segundos y microsegundos de diferencia. |

### Códigos de Formato (Esencial)

Para convertir fechas a texto (y viceversa), usarás estos códigos frecuentemente:
*   `%Y`: Año (2024)
*   `%m`: Mes (01-12)
*   `%d`: Día (01-31)
*   `%H`: Hora (00-23)
*   `%M`: Minuto (00-59)
*   `%S`: Segundo (00-59)

### Ejemplos de Código

Para estos ejemplos, es común importar las clases específicas para no escribir `datetime.datetime` todo el tiempo.

##### Ejemplo 1: Obtener la fecha y hora actual

Lo básico: saber "cuándo" es ahora mismo.

```python
from datetime import datetime, date

# 1. Fecha y hora exacta actual
ahora = datetime.now()
print(f"Ahora mismo es: {ahora}")

# 2. Solo la fecha de hoy
hoy = date.today()
print(f"Hoy es: {hoy}")

# 3. Acceder a partes individuales
print(f"Estamos en el año: {ahora.year}")
print(f"Son las {ahora.hour} horas con {ahora.minute} minutos.")
```

##### Ejemplo 2: Crear una fecha específica

Útil para cumpleaños, vencimientos o fechas límite.

```python
from datetime import datetime, date

# Crear una fecha fija (Año, Mes, Día)
navidad = date(2024, 12, 25)

# Crear una fecha y hora fija (Año, Mes, Día, Hora, Min)
lanzamiento = datetime(2024, 1, 15, 14, 30)

print(f"Navidad cae en: {navidad}")
print(f"El lanzamiento es: {lanzamiento}")
```

##### Ejemplo 3: Formateo (Texto <-> Fecha)

Esta es **la parte más importante** de la librería.
*   **`strftime`** (String Format Time): De Objeto Fecha -> Texto.
*   **`strptime`** (String Parse Time): De Texto -> Objeto Fecha.

```python
from datetime import datetime

ahora = datetime.now()

# A. Convertir OBJETO a TEXTO (Para nombrar archivos, mostrar en pantalla)
# Queremos formato: "30-01-2024_14:00"
texto_bonito = ahora.strftime("%d-%m-%Y_%H:%M")
print(f"Fecha formateada: {texto_bonito}")

# Ejemplo práctico con lo aprendido antes:
archivo_backup = f"respaldo_{texto_bonito}.zip"
print(f"Nombre del archivo a generar: {archivo_backup}")


# B. Convertir TEXTO a OBJETO (Para leer bases de datos o inputs de usuario)
fecha_texto = "2023/05/20"
# Le decimos a Python cómo interpretar ese texto
fecha_objeto = datetime.strptime(fecha_texto, "%Y/%m/%d")

print(f"Objeto recuperado: {fecha_objeto.month}") # Ahora podemos operar con él
```

##### Ejemplo 4: Matemáticas con fechas (`timedelta`)

Aquí es donde la magia ocurre. Sumar y restar tiempo.

```python
from datetime import datetime, timedelta

ahora = datetime.now()

# 1. Sumar días (Vencimiento de una factura en 15 días)
vencimiento = ahora + timedelta(days=15)
print(f"La factura vence el: {vencimiento.strftime('%d/%m/%Y')}")

# 2. Restar tiempo (¿Qué hora era hace 2 horas?)
hace_rato = ahora - timedelta(hours=2)
print(f"Hace dos horas eran las: {hace_rato.strftime('%H:%M')}")

# 3. Calcular diferencia entre dos fechas
año_nuevo = datetime(2025, 1, 1)
falta_tiempo = año_nuevo - ahora # Esto devuelve un objeto timedelta

print(f"Faltan {falta_tiempo.days} días para Año Nuevo.")
```

### Diferencias Clave: `time` vs `datetime`

1.  **`datetime`**:
    *   Orientado a **calendarios y relojes humanos**.
    *   Sabe de años bisiestos, meses de 30/31 días.
    *   Uso: Fechas de archivos, logs, bases de datos.
2.  **`time`** (Librería estándar distinta):
    *   Orientado al **tiempo de la CPU** (Unix Timestamp).
    *   Maneja "segundos desde 1970" (un número gigante).
    *   Uso principal: `time.sleep(5)` (pausar el programa) o medir rendimiento bruto.
