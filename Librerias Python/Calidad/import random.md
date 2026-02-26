### Random

El módulo random implementa generadores de números pseudo-aleatorios.  
Permite realizar tareas como:

- Generar números al azar (enteros o decimales).
- Elegir elementos al azar de una lista.
- Mezclar (barajar) datos.
- Realizar muestreos estadísticos.

**Nota Importante:** Se llaman "pseudo-aleatorios" porque son generados por una fórmula matemática. Si conoces la "semilla" (seed) inicial, puedes predecir los números. **No uses esta librería para contraseñas o seguridad bancaria** (para eso existe secrets).

### Funciones Principales

Las dividimos en dos grupos: Generación de Números y Operaciones con Secuencias (Listas).

##### Generación de Números

|                      |                                                                               |                                  |
| -------------------- | ----------------------------------------------------------------------------- | -------------------------------- |
| Función              | Descripción                                                                   | Ejemplo                          |
| random.randint(a, b) | Devuelve un entero aleatorio N tal que a <= N <= b. (Incluye ambos extremos). | Un dado: randint(1, 6)           |
| random.random()      | Devuelve un decimal (float) entre 0.0 y 1.0 (excluyendo el 1.0).              | Probabilidad: 0.753...           |
| random.uniform(a, b) | Devuelve un decimal aleatorio entre a y b.                                    | Temperatura: uniform(20.5, 30.0) |

##### Operaciones con Listas (Secuencias)

|                            |                                                                 |
| -------------------------- | --------------------------------------------------------------- |
| Función                    | Descripción                                                     |
| random.choice(lista)       | Elige **un** elemento aleatorio de la lista.                    |
| random.choices(lista, k=N) | Elige **N** elementos, permitiendo repetidos (con reemplazo).   |
| random.sample(lista, k=N)  | Elige **N** elementos únicos, **sin repetir** (sin reemplazo).  |
| random.shuffle(lista)      | Mezcla/Baraja la lista original (cambia el orden **in-place**). |

### Ejemplos de Código

##### Ejemplo 1: El Dado y la Moneda (randint y choice)

Lo más básico para juegos.

```python
import random

# 1. Simular un dado (Entero entre 1 y 6)
dado = random.randint(1, 6)
print(f"Has sacado un: {dado}")

# 2. Simular una moneda (Elegir de una lista)
opciones = ["Cara", "Cruz"]
resultado = random.choice(opciones)
print(f"La moneda cayó en: {resultado}")
```

##### Ejemplo 2: Probabilidad (random)

El random.random() devuelve un número entre 0 y 1 (como un porcentaje del 0% al 100%).

```python
import random

# Simulamos una probabilidad de éxito del 20%
probabilidad = random.random() # Genera ej: 0.154 o 0.899

print(f"Valor generado: {probabilidad}")

if probabilidad < 0.20:
    print("¡Éxito crítico! (Ocurrió el evento raro)")
else:
    print("Evento normal.")
```

##### Ejemplo 3: Sorteos y Muestras (sample vs choices)

Esta diferencia es vital. Imagina una rifa.

```python
import random

participantes = ["Ana", "Beto", "Carla", "Daniel", "Elena"]

# A. SAMPLE: Ganadores únicos (No puedes ganar dos veces)
ganadores = random.sample(participantes, k=2)
print(f"Ganadores del premio (sin repetir): {ganadores}")

# B. CHOICES: Selección con repetición (Ej: Tirar 3 veces una moneda)
lados = ["Cara", "Cruz"]
tiradas = random.choices(lados, k=3)
print(f"Resultados de 3 tiradas: {tiradas}")
```

##### Barajar cartas (shuffle)

Nota que shuffle no devuelve nada, modifica la lista original.

```python
import random

playlist = ["Canción A", "Canción B", "Canción C", "Canción D"]

print(f"Orden original: {playlist}")

# Barajamos
random.shuffle(playlist)

print(f"Orden aleatorio: {playlist}")
```

##### Ejemplo 5: La Semilla (seed)

Muy útil para "Testing". Si fijas la semilla, los números aleatorios serán siempre los mismos. Esto permite reproducir errores.

```python
import random

print("--- Ejecución 1 ---")
random.seed(42) # Fijamos la semilla
print(random.randint(1, 100))
print(random.randint(1, 100))

print("\n--- Ejecución 2 (Misma semilla) ---")
random.seed(42) # Reiniciamos la semilla al mismo valor
print(random.randint(1, 100)) # ¡Saldrá el mismo número que arriba!
print(random.randint(1, 100))
```

### Concepto Avanzado: Seguridad

Una advertencia profesional que debes incluir en tu documentación.

**El problema:** random es predecible si se tienen suficientes datos.  
**La solución:** Si necesitas generar contraseñas, tokens de API o claves criptográficas, usa la librería **secrets** (incluida en Python 3.6+).

```python
import secrets

# Generar un token seguro para una URL (reset password)
token = secrets.token_urlsafe(16)
print(f"Token seguro: {token}")
```
