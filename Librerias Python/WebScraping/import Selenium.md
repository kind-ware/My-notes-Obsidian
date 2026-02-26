### Documentación: Librería `selenium`

## 1. Descripción General
Selenium automatiza navegadores web.
Lo que hace es abrir una ventana de Chrome (o el que elijas), cargar la URL, ejecutar el JavaScript y permitirte interactuar con el resultado final (el DOM).

**Pros:**
*   Ve lo mismo que el usuario (incluyendo lo generado por JS).
*   Puede hacer clic, escribir, arrastrar y soltar.

**Contras:**
*   Es **lento** y pesado (consume mucha RAM y CPU) comparado con `requests`.
*   Es más fácil de detectar por sistemas anti-bot.

**Nota:** Es una librería externa.
Instalación: `pip install selenium`
*(En versiones antiguas necesitabas descargar un archivo `chromedriver.exe` manualmente. En Selenium 4+, la librería suele descargarlo sola, ¡magia!).*

### Conceptos Clave

| Componente      | Descripción                                                                                                |
| :-------------- | :--------------------------------------------------------------------------------------------------------- |
| `webdriver`     | El conductor. Es el objeto que representa al navegador.                                                    |
| `By`            | La forma de encontrar elementos (`By.ID`, `By.CSS_SELECTOR`, `By.XPATH`).                                  |
| `Keys`          | Teclas especiales (Enter, Tab, Esc, Flechas).                                                              |
| `WebDriverWait` | **Vital.** Hace que el script espere inteligentemente a que aparezca un elemento antes de intentar usarlo. |
| `Options`       | Configuraciones del navegador (Modo incógnito, Tamaño de ventana, Headless).                               |

### La Estrategia de Búsqueda (`By`)

En Selenium 4, se usa una sola función `find_element` y se le pasa la estrategia:

*   `By.ID`: La más rápida y segura.
*   `By.CSS_SELECTOR`: La más versátil (estilo jQuery).
*   `By.XPATH`: La más potente (permite buscar por texto: *"El botón que dice 'Login'"*).
*   `By.NAME`: Para formularios.

### Ejemplos de Código

##### Ejemplo 1: Hola Mundo (Abrir Google)

Este script abrirá una ventana real de Chrome, irá a Google y se cerrará a los 5 segundos.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
import time

# 1. Iniciar el navegador (Chrome)
# Selenium Manager descargará el driver necesario automáticamente si no lo tienes
driver = webdriver.Chrome()

# 2. Navegar a una URL
driver.get("https://www.google.com")

# 3. Obtener datos
titulo = driver.title
print(f"El título de la página es: {titulo}")

# 4. Esperar un poco para que veas que funciona
time.sleep(3)

# 5. Cerrar el navegador (Importante para no dejar procesos zombies)
driver.quit()
```

##### Ejemplo 2: Interactuar (Buscar en Wikipedia)

Aquí aprenderás a escribir (`send_keys`) y presionar Enter (`Keys.RETURN`).

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
import time

driver = webdriver.Chrome()
driver.get("https://es.wikipedia.org")

# 1. Buscar la barra de búsqueda
# En Wikipedia, el input tiene name="search"
caja_busqueda = driver.find_element(By.NAME, "search")

# 2. Escribir
caja_busqueda.send_keys("Python (lenguaje de programación)")

# 3. Presionar Enter
caja_busqueda.send_keys(Keys.RETURN)

# 4. Esperar a que cargue la nueva página (forma "mala" con sleep)
time.sleep(2)

# 5. Verificar que llegamos
assert "Python" in driver.title
print("¡Búsqueda exitosa!")

driver.quit()
```

##### Ejemplo 3: Esperas Explícitas (La forma PROFESIONAL)

El error n.º 1 en Selenium es usar `time.sleep(5)`. ¿Y si el internet es lento y tarda 6? El script falla. ¿Y si carga en 1? Perdiste 4 segundos.

Usamos `WebDriverWait` para decir: *"Espera HASTA 10 segundos, pero si aparece antes, continúa ya"*.

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("https://www.google.com")

try:
    # Definimos una espera máxima de 10 segundos
    wait = WebDriverWait(driver, 10)

    # Esperamos a que la caja de búsqueda sea "visible"
    # name='q' es la caja de búsqueda de Google
    caja = wait.until(EC.visibility_of_element_located((By.NAME, "q")))
    
    caja.send_keys("Selenium Python")
    caja.submit() # submit() es equivalente a dar Enter en un formulario

    # Esperamos a que aparezcan los resultados (id='search')
    resultados = wait.until(EC.presence_of_element_located((By.ID, "search")))
    
    print("Resultados cargados correctamente.")

except Exception as e:
    print(f"Error: No cargó a tiempo. {e}")

finally:
    driver.quit()
```

##### Ejemplo 4: Modo Headless (Sin Ventana) y Screenshots

Para servidores o scripts automáticos, no quieres que se abra una ventana gráfica.
Configuramos Chrome en modo `headless`.

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

# 1. Configurar opciones
chrome_options = Options()
chrome_options.add_argument("--headless") # No mostrar ventana UI
chrome_options.add_argument("--window-size=1920,1080") # Definir tamaño virtual

driver = webdriver.Chrome(options=chrome_options)

print("Navegando en modo invisible...")
driver.get("https://www.python.org")

# 2. Tomar captura de pantalla
driver.save_screenshot("captura_python.png")
print("Captura guardada como 'captura_python.png'")

driver.quit()
```

### La "Pareja Perfecta": Selenium + BeautifulSoup

Selenium es lento para leer datos, pero bueno para renderizar.
BeautifulSoup es rápido para leer, pero malo para renderizar.
**Puedes usarlos juntos**

1.  Usas **Selenium** para cargar la página, hacer scroll y loguearte.
2.  Una vez cargada, le pides el HTML final (`driver.page_source`).
3.  Le pasas ese HTML a **BeautifulSoup** para extraer los datos rapidísimo.

```python
# ... (código selenium previo) ...
driver.get("https://sitio-dinamico.com")

# Una vez que cargó todo el JS:
html_renderizado = driver.page_source

# Pasamos la posta a BeautifulSoup
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_renderizado, "html.parser")

# Y ahora extraemos datos a la velocidad de la luz
precios = soup.find_all("span", class_="precio")
```

### Integración con Stack

1.  **Con `sqlite3` / `SQLModel`**:
    *   Haces un bot que entra a Amazon, busca "Laptop", extrae precios y los guarda en tu base de datos para monitorear ofertas históricas.
2.  **Con `schedule` / `time`**:
    *   Programas el script para que corra cada mañana a las 9 AM.
3.  **Con `logging`**:
    *   Si Selenium no encuentra un botón (porque la web cambió de diseño), guardas el error en un log y te envías una alerta.
4.  **Con `pandas`**:
    *   Si extraes tablas de datos, las pasas a un DataFrame y luego a Excel.

### Resumen

Selenium es poder bruto.

*   ¿La web tiene login? Selenium puede escribir usuario/pass.
*   ¿La web tiene scroll infinito? Selenium puede simular la tecla `Fin`.
*   ¿La web te bloquea `requests`? Selenium parece un navegador real (aunque hay formas de detectarlo).

**Nota final:**
Existe una alternativa moderna llamada **`Playwright`**. Es más rápida, maneja mejor las esperas automáticas y permite grabar videos de la sesión.
Sin embargo, Selenium sigue siendo el rey en cuanto a cantidad de tutoriales y soporte en internet.

