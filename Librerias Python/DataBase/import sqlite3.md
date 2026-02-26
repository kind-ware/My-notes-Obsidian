### SQLite3

El módulo `sqlite3` proporciona una interfaz SQL compatible con la especificación DB-API 2.0.
Permite interactuar con bases de datos SQLite usando sentencias SQL puras (`SELECT`, `INSERT`, `CREATE`).

**El Flujo de Trabajo ("La Trinidad"):**
1.  **Conexión (`Connection`):** Abre el archivo de la base de datos.
2.  **Cursor (`Cursor`):** Es el "obrero" que ejecuta las órdenes y recorre los resultados.
3.  **Sentencia (`Execute`):** El código SQL que quieres correr.

### Funciones Principales

| Función / Método                  | Descripción                                                                       |
| :-------------------------------- | :-------------------------------------------------------------------------------- |
| `sqlite3.connect("archivo.db")`   | Conecta a la base de datos. Si el archivo no existe, **lo crea automáticamente**. |
| `connection.cursor()`             | Crea un objeto cursor para ejecutar comandos.                                     |
| `cursor.execute(sql, parametros)` | Ejecuta una sentencia SQL. **Usa `?` para parámetros** (seguridad).               |
| `cursor.executemany(sql, lista)`  | Ejecuta la misma sentencia muchas veces (inserción masiva).                       |
| `cursor.fetchone()`               | Devuelve el **siguiente** resultado de una consulta (uno solo).                   |
| `cursor.fetchall()`               | Devuelve **todos** los resultados restantes como una lista.                       |
| `connection.commit()`             | **Guardar cambios.** Obligatorio tras `INSERT`, `UPDATE` o `DELETE`.              |
| `connection.close()`              | Cierra la conexión. Importante para no corromper el archivo.                      |

### Ejemplos de Código
##### Ejemplo 1: Crear Base de Datos y Tabla

Lo básico: crear el archivo y definir la estructura.

```python
import sqlite3

# 1. Conectar (Crea 'mi_empresa.db' si no existe)
conexion = sqlite3.connect("mi_empresa.db")

# 2. Crear el cursor
cursor = conexion.cursor()

# 3. Ejecutar SQL (Crear tabla)
# Usamos triple comilla para escribir SQL en varias líneas
cursor.execute("""
    CREATE TABLE IF NOT EXISTS empleados (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        nombre TEXT NOT NULL,
        salario REAL,
        departamento TEXT
    )
""")

# 4. Guardar y Cerrar
conexion.commit()
conexion.close()

print("Base de datos creada exitosamente.")
```

##### Ejemplo 2: Insertar Datos (Create)

Aquí insertamos datos. Nota el uso de `?` para evitar problemas de seguridad.

```python
import sqlite3

conexion = sqlite3.connect("mi_empresa.db")
cursor = conexion.cursor()

# A. Inserción simple
cursor.execute("INSERT INTO empleados (nombre, salario, departamento) VALUES (?, ?, ?)", 
               ("Juan Pérez", 2500.50, "Ventas"))

# B. Inserción masiva (executemany) - Muy eficiente
nuevos_empleados = [
    ("Ana Gómez", 3200.00, "IT"),
    ("Carlos Ruiz", 1800.00, "Limpieza"),
    ("Laura Díaz", 4500.00, "Gerencia")
]
cursor.executemany("INSERT INTO empleados (nombre, salario, departamento) VALUES (?, ?, ?)", 
                   nuevos_empleados)

conexion.commit() # ¡IMPORTANTE! Si olvidas esto, no se guarda nada.
conexion.close()

print("Datos insertados.")
```

##### Ejemplo 3: Leer Datos (Read - SELECT)

Recuperar información. Por defecto devuelve tuplas `(1, 'Juan', ...)`.

```python
import sqlite3

conexion = sqlite3.connect("mi_empresa.db")
cursor = conexion.cursor()

print("--- Todos los empleados ---")
cursor.execute("SELECT * FROM empleados")
todos = cursor.fetchall() # Lista de tuplas

for emp in todos:
    print(emp) 
    # Salida: (1, 'Juan Pérez', 2500.5, 'Ventas')

print("\n--- Solo los de IT ---")
# Usamos una tupla (it,) para el parámetro, incluso si es uno solo
cursor.execute("SELECT nombre, salario FROM empleados WHERE departamento = ?", ("IT",))
empleado_it = cursor.fetchone() # Solo el primero que encuentre

print(f"Encontrado: {empleado_it}")

conexion.close()
```

##### Ejemplo 4: Actualizar y Borrar (Update / Delete)

```python
import sqlite3

with sqlite3.connect("mi_empresa.db") as conexion:
    cursor = conexion.cursor()
    
    # UPDATE: Subir sueldo a Ventas
    cursor.execute("UPDATE empleados SET salario = salario + 100 WHERE departamento = ?", ("Ventas",))
    
    # DELETE: Despedir a Carlos
    cursor.execute("DELETE FROM empleados WHERE nombre = ?", ("Carlos Ruiz",))
    
    # Al usar 'with', el commit se hace automático si no hay errores.
    # Si hay error, hace rollback (deshace cambios).
    
print("Actualización completada.")
```

### Concepto Avanzado: Seguridad (SQL Injection)

Esta es la lección más importante de `sqlite3`.

**MAL (Inseguro):**
Usar f-strings o concatenación de texto permite que un hacker borre tu base de datos.
```python
usuario = "admin'; DROP TABLE empleados; --"
# Esto ejecuta: SELECT * FROM users WHERE name = 'admin'; DROP TABLE empleados; --'
cursor.execute(f"SELECT * FROM empleados WHERE nombre = '{usuario}'") 
```

**BIEN (Seguro - Parameterized Query):**
Usar `?` hace que la base de datos trate la entrada como texto literal, no como código ejecutable.
```python
usuario = "admin'; DROP TABLE empleados; --"
# Esto busca literalmente a alguien llamado "admin'; DROP..." (que no existe)
cursor.execute("SELECT * FROM empleados WHERE nombre = ?", (usuario,))
```

### Sugerencia: `Row Factory` (Diccionarios)

Por defecto, `sqlite3` devuelve tuplas `(1, 'Ana')`. Es difícil recordar qué es el índice 0 o 1.
Puedes configurarlo para que devuelva algo parecido a un diccionario.

```python
import sqlite3

con = sqlite3.connect("mi_empresa.db")

# Cambiamos la fábrica de filas
con.row_factory = sqlite3.Row 

cur = con.cursor()
cur.execute("SELECT * FROM empleados WHERE id = 1")
fila = cur.fetchone()

# Ahora puedes acceder por nombre de columna
print(f"Nombre: {fila['nombre']}")
print(f"Salario: {fila['salario']}")

con.close()
```

### Diferencia: `sqlite3` vs `SQLModel`

Conociendo ambas, verás la diferencia clara:

| Característica       | `sqlite3` (Nativo)                                   | `SQLModel` (ORM)                                 |
| :------------------- | :--------------------------------------------------- | :----------------------------------------------- |
| **Lenguaje**         | Escribes **SQL** (`SELECT * FROM...`)                | Escribes **Python** (`select(Heroe)`)            |
| **Validación**       | Ninguna (SQLite es flexible)                         | Estricta (Pydantic)                              |
| **Velocidad de Dev** | Lenta (escribir strings a mano)                      | Rápida (autocompletado en IDE)                   |
| **Uso ideal**        | Scripts pequeños, aprender SQL, rendimiento extremo. | APIs (FastAPI), Proyectos grandes y mantenibles. |

