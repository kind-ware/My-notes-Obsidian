### BeatifulSoup (bs4)
`BeautifulSoup` sirve para **extraer información** de archivos HTML y XML.

**¿Cómo funciona?**
1.  Descargas el HTML de una web (usando `requests`).
2.  Le pasas ese HTML a BeautifulSoup.
3.  BeautifulSoup crea un objeto "Sopa".
4.  Tú le pides cosas a la sopa: *"Dame todos los enlaces"*, *"Dame el texto del título"*, *"Dame la tabla de precios"*.

**Nota:** Librería externa.
Instalación: `pip install beautifulsoup4 
*(También se recomienda instalar `lxml` para mayor velocidad: `pip install lxml`)*.

### Conceptos Clave

Para usarla, necesitas entender un poco de HTML:
*   **Tag (Etiqueta):** Elementos como `<h1>`, `<p>`, `<a>`, `<div>`.
*   **Attribute (Atributo):** Propiedades dentro de la etiqueta, como `href="..."` (enlaces) o `class="..."` (estilos).
*   **Text:** Lo que lee el usuario humano dentro de la etiqueta.

### Funciones Principales

El objeto principal es `soup`.

| Método                          | Descripción                                                              |
| :------------------------------ | :----------------------------------------------------------------------- |
| `BeautifulSoup(html, parser)`   | Crea la sopa. El parser suele ser `'html.parser'` o `'lxml'`.            |
| `soup.find(tag, attrs)`         | Busca la **primera** etiqueta que coincida. Devuelve un objeto o `None`. |
| `soup.find_all(tag, attrs)`     | Busca **todas** las etiquetas que coincidan. Devuelve una lista `[]`.    |
| `soup.select(css_selector)`     | Busca usando selectores CSS (como JQuery/CSS). Muy potente.              |
| `elemento.text` o `.get_text()` | Extrae el texto limpio de la etiqueta (sin el HTML).                     |
| `elemento['atributo']`          | Obtiene el valor de un atributo (ej: el enlace de un `<a>`).             |

### Ejemplos de Código

##### Ejemplo 1: Tu Primera "Sopa" (HTML Local)

Para entender la lógica sin conectarnos a internet todavía.

```python
from bs4 import BeautifulSoup

# HTML simulado (como si viniera de una web)
html_doc = """
<html>
    <head><title>Mi Tienda</title></head>
    <body>
        <h1>Productos en Oferta</h1>
        <div class="producto">
            <p class="nombre">Laptop Gamer</p>
            <p class="precio">$1500</p>
            <a href="http://tienda.com/laptop" id="link1">Ver más</a>
        </div>
        <div class="producto">
            <p class="nombre">Mouse Óptico</p>
            <p class="precio">$20</p>
        </div>
    </body>
</html>
"""

# 1. Crear la sopa
soup = BeautifulSoup(html_doc, 'html.parser')

# 2. Extraer datos simples
titulo = soup.title.text
h1 = soup.find('h1').text

print(f"Título de la página: {titulo}")
print(f"Encabezado: {h1}")

# 3. Extraer un atributo (el enlace)
link = soup.find('a') # Busca la primera etiqueta <a>
url = link['href']    # Accedemos como si fuera un diccionario
print(f"Enlace encontrado: {url}")
```

##### Ejemplo 2: Scraping Real (`requests` + `bs4`)

Vamos a buscar títulos en una web real. Usaremos una web diseñada para practicar scraping: `books.toscrape.com`.

```python
import requests
from bs4 import BeautifulSoup

url = "http://books.toscrape.com/"

# 1. Descargar el HTML (Requests)
response = requests.get(url)

if response.status_code == 200:
    # 2. Parsear el HTML (BeautifulSoup)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    # 3. Buscar elementos
    # En esa web, los libros están en etiquetas <article class="product_pod">
    # Dentro tienen un <h3> y dentro un <a> con el título en el atributo 'title'
    
    articulos = soup.find_all('article', class_='product_pod')
    
    print(f"Se encontraron {len(articulos)} libros en esta página:\n")
    
    for libro in articulos[:5]: # Solo los primeros 5
        # Buscamos la etiqueta <h3>
        tag_h3 = libro.find('h3')
        # Buscamos la etiqueta <a> dentro del h3
        tag_a = tag_h3.find('a')
        
        # Obtenemos el título (está en el atributo 'title', no en el texto)
        titulo_libro = tag_a['title']
        
        # Buscamos el precio (está en un <p class="price_color">)
        precio = libro.find('p', class_='price_color').text
        
        print(f"- {titulo_libro} ({precio})")

else:
    print("Error al cargar la página")
```

##### Ejemplo 3: Selectores CSS (`.select`)

Si sabes CSS, esto es mucho más rápido que usar `find`.
*   `.clase` busca por clase.
*   `#id` busca por ID.
*   `div > p` busca un p directo dentro de un div.

```python
from bs4 import BeautifulSoup

html = """
<div id="contenedor-principal">
    <ul class="menu">
        <li class="item">Inicio</li>
        <li class="item destacado">Ofertas</li>
        <li class="item">Contacto</li>
    </ul>
</div>
"""
soup = BeautifulSoup(html, 'html.parser')

# 1. Buscar por ID
div = soup.select_one('#contenedor-principal')

# 2. Buscar elementos anidados por clase
# "Dame todos los .item que estén dentro de .menu"
items = soup.select('.menu .item')

print("Menú:")
for i in items:
    print(f"Opción: {i.text}")

# 3. Buscar clase específica
oferta = soup.select_one('.destacado')
print(f"\nDestacado: {oferta.text}")
```

##### Ejemplo 4: Descargar Imágenes

Combinando `bs4` para encontrar la URL y `open` (file I/O) para guardar.

```python
import requests
from bs4 import BeautifulSoup
import os

url = "http://books.toscrape.com/"
soup = BeautifulSoup(requests.get(url).text, 'html.parser')

# Buscamos la primera imagen
img_tag = soup.find('img')
img_url_relativa = img_tag['src'] # ej: media/cache/2c/da/2cd...jpg

# Construimos la URL completa
img_url_absoluta = url + img_url_relativa

print(f"Descargando: {img_url_absoluta}")

# Descargamos los bytes
img_data = requests.get(img_url_absoluta).content

with open("portada_libro.jpg", "wb") as f:
    f.write(img_data)

print("Imagen guardada.")
```

### Ética y Buenas Prácticas 🤖

El Web Scraping es una zona gris. Sigue estas reglas:

1.  **Revisa el `robots.txt`:** Ve a `sitio.com/robots.txt` para ver qué permiten scrapear.
2.  **No seas agresivo:** No hagas 100 peticiones por segundo o tumbarás el servidor (y te bloquearán la IP). Usa `time.sleep()`.
3.  **User-Agent:** Identifícate. Algunas webs bloquean scripts de Python por defecto.

```python
headers = {
    'User-Agent': 'MiScraperBot/1.0 (educativo)'
}
requests.get(url, headers=headers)
```

### Integración con Stack

Mira lo que puedes construir ahora:

1.  **`requests`**: Bajas el HTML de una web de noticias.
2.  **`BeautifulSoup`**: Extraes los titulares y los enlaces.
3.  **`pydantic`**: Validas que los datos tengan formato correcto (Título no vacío, URL válida).
4.  **`sqlite3` / `SQLModel`**: Guardas las noticias en tu base de datos para no perderlas.
5.  **`Rich`**: Muestras en la terminal una tabla bonita con las noticias encontradas.
6.  **`schedule`** (o un bucle con `time.sleep`): Haces que esto corra automáticamente cada hora.

### Resumen

`BeautifulSoup` es ideal para sitios **estáticos** (donde el contenido viene en el HTML).

**¿El problema?**
Hoy en día, muchas webs (YouTube, Twitter, Instagram) usan JavaScript para cargar el contenido (React, Vue, Angular).
Si usas `requests` + `bs4` en esas webs, verás un HTML vacío o de carga, porque `requests` no ejecuta JavaScript.
	
Para esos casos (sitios dinámicos), necesitas un **Navegador Automatizado**.
El rey histórico es **`Selenium`**, pero el príncipe moderno y rápido es **`Playwright`**.

¿Cuál prefieres ver para scrapear sitios complejos?
1.  **`Selenium`**: El estándar clásico, muy documentado.
2.  **`Playwright`**: La versión moderna de Microsoft, más rápida y capaz.