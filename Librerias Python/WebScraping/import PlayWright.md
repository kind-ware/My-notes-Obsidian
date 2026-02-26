### playwright
Playwright permite automatizar Chromium, Firefox y WebKit con una sola API.
Su arquitectura es diferente a Selenium: se comunica directamente con el navegador a través de protocolos de depuración, lo que lo hace mucho más estable.

**Jerarquía:**
1.  **Browser:** El navegador en sí (Chrome, Firefox).
2.  **Context:** Una sesión aislada (como una ventana de Incógnito). Puedes tener muchos contextos en un solo navegador.
3.  **Page:** Una pestaña dentro del contexto.

**¿Por qué Playwright enamora?**
1.  **Velocidad:** Es ridículamente rápido comparado con Selenium.
2.  **Auto-Wait (Esperas Automáticas):** Olvídate de `time.sleep` o `WebDriverWait`. Playwright espera automáticamente a que los elementos estén listos antes de hacer clic.
3.  **Headless por defecto:** Corre sin abrir ventana gráfica (para servidores), pero puedes verla si quieres.
4.  **Generador de Código:** Puedes navegar manualmente y Playwright escribe el código por ti.
5.  **Nativo Asíncrono:** Se integra perfectamente con `asyncio` y `FastAPI`.

**Nota:** Librería externa.
Instalación (tiene dos pasos):
1.  `pip install playwright`
2.  `playwright install` (Esto descarga los binarios de Chromium, Firefox y WebKit).

### Modos de Uso (Síncrono vs Asíncrono)

Playwright ofrece dos APIs.
*   **`sync_api`**: Para scripts sencillos (estilo Selenium).
*   **`async_api`**: Para aplicaciones modernas (`asyncio`, `FastAPI`). **Esta es la recomendada.**

### Ejemplos de Código

##### Ejemplo 1: Tu Primer Script (Modo Síncrono)
Para entender la estructura básica.

```python
from playwright.sync_api import sync_playwright

def run():
    # El manager 'sync_playwright' maneja los drivers
    with sync_playwright() as p:
        # 1. Lanzar navegador
        # headless=False para VER la ventana (por defecto es True/Invisible)
        browser = p.chromium.launch(headless=False)
        
        # 2. Abrir una página
        page = browser.new_page()
        
        # 3. Navegar
        page.goto("https://www.google.com")
        
        # 4. Interactuar (Auto-wait en acción)
        # Playwright espera a que el título exista
        print(f"Título: {page.title()}")
        
        # Tomar captura
        page.screenshot(path="google.png")
        
        # 5. Cerrar
        browser.close()

if __name__ == "__main__":
    run()
```

##### Ejemplo 2: Modo Asíncrono (`asyncio`) - Nivel Pro

Aquí aprovechamos tu conocimiento de `asyncio`. Esto es ideal si quieres scrapear 10 páginas a la vez.

```python
import asyncio
from playwright.async_api import async_playwright

async def main():
    async with async_playwright() as p:
        # Lanzamos el navegador
        browser = await p.chromium.launch(headless=False)
        
        # Creamos una página
        page = await browser.new_page()
        
        print("Navegando a Wikipedia...")
        await page.goto("https://es.wikipedia.org")
        
        # Selector inteligente: busca por placeholder
        await page.fill('input[name="search"]', "Python")
        
        # Presionar Enter
        await page.keyboard.press("Enter")
        
        # Esperar a que cargue el selector del título
        # (Playwright espera solo, pero a veces queremos asegurar un elemento específico)
        await page.wait_for_selector("#firstHeading")
        
        titulo = await page.title()
        print(f"Llegamos a: {titulo}")
        
        await browser.close()

if __name__ == "__main__":
    asyncio.run(main())
```

##### Ejemplo 3: Selectores Poderosos y Extracción

Playwright tiene un motor de selectores increíble.
*   `text="Log in"`: Busca por texto.
*   `css=.clase`: Busca por CSS.
*   `xpath=//div`: Busca por XPath.
*   `n-th=0`: El primer elemento.

```python
# Supongamos que estamos dentro de main() async

# Click en un botón que contiene el texto "Comenzar"
await page.click('text=Comenzar')

# Click en un botón CSS específico
await page.click('button.primary-btn')

# Extraer texto de una lista de elementos
# $$ evalúa todos los elementos que coinciden
temas = await page.eval_on_selector_all('.topic-list li', 'elements => elements.map(e => e.innerText)')

print("Temas encontrados:", temas)
```

##### Ejemplo 4: Interceptar Red (Network Interception)

Esto es algo que Selenium sufre para hacer. Playwright puede **bloquear** imágenes o CSS para que el scraping sea más rápido.

```python
async def main():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page()

        # Bloquear imágenes y fuentes para ahorrar datos/tiempo
        await page.route("**/*", lambda route: 
            route.abort() 
            if route.request.resource_type in ["image", "font"] 
            else route.continue_()
        )

        await page.goto("https://amazon.com")
        # Cargará rapidísimo porque no descarga las fotos de productos
        
        await browser.close()
```

##### Ejemplo 5: Grabación de Video y Tracing

Playwright puede grabar un video de toda la sesión. Ideal para depurar errores ("¿Por qué falló el bot anoche?").

```python
browser = await p.chromium.launch()
# Al crear el contexto, activamos la grabación
context = await browser.new_context(record_video_dir="videos/")
page = await context.new_page()

await page.goto("https://youtube.com")
# ... hacemos cosas ...

await context.close() # Al cerrar el contexto, se guarda el video.
```

### La "Killer Feature": Codegen

Esta herramienta hace que Playwright sea superior para principiantes y expertos.
No necesitas escribir el código de los selectores a mano.

Abre tu terminal y escribe:
`playwright codegen wikipedia.org`

**¿Qué pasa?**
1.  Se abre un navegador.
2.  Se abre una ventana con código Python.
3.  **Todo lo que hagas en el navegador (clics, escribir text) se escribe automáticamente en código Python en tiempo real.**
4.  Copias el código, lo pegas en tu editor y listo. ¡Es magia! ✨

### Selenium vs Playwright

| Característica  | Selenium                            | Playwright                        |
| :-------------- | :---------------------------------- | :-------------------------------- |
| **Velocidad**   | Lento (Protocolo HTTP antiguo)      | Muy Rápido (WebSockets)           |
| **Esperas**     | Manuales (`WebDriverWait`, `sleep`) | **Automáticas** (Auto-wait)       |
| **Instalación** | Drivers manuales (antes)            | `playwright install` (automático) |
| **Selectores**  | Estándar (CSS/XPath)                | Avanzados (Text, Layout, React)   |
| **Async**       | No nativo                           | Nativo (`async`/`await`)          |
| **Comunidad**   | Gigante (20 años de historia)       | Creciendo rápido (Microsoft)      |

### Integración con Stack

1.  **`pytest`**: Existe un plugin llamado `pytest-playwright`.
    *   Te permite escribir tests de integración para tu web.
    *   Ejemplo: "Entrar a la web, loguearse y verificar que aparece el dashboard".

2.  **`BeautifulSoup`**:
    *   Usas Playwright para cargar la página dinámica (JS).
    *   Obtienes el HTML con `content = await page.content()`.
    *   Se lo pasas a `BeautifulSoup(content)` para parsear rápido.

3.  **`FastAPI`**:
    *   Puedes crear un endpoint que dispare un scraper de Playwright en segundo plano y devuelva los datos en JSON.

### Resumen

Playwright es el futuro de la automatización web.
*   Si necesitas interactuar con webs modernas (React, Vue, Angular).
*   Si necesitas velocidad.
*   Si quieres evitar los dolores de cabeza de los tiempos de espera.

Ahora tienes:
*   **Web Scraping Estático:** `requests` + `BeautifulSoup`.
*   **Web Scraping Dinámico / Tests E2E:** `Playwright`.

