### Scapy

`scapy` es una potente herramienta interactiva de manipulación de paquetes.
A diferencia de otras librerías que ocultan la complejidad, `scapy` te obliga a entender cómo funcionan los protocolos (Ethernet, IP, TCP, ICMP).

**¿Qué puedes hacer?**

*   **Forjar paquetes:** Crear un paquete desde cero bit a bit (ej: un paquete que parezca venir de Google, pero viene de tu PC).
*   **Sniffing:** Interceptar y leer tráfico de la red (como Wireshark).
*   **Escaneo:** Descubrir qué IPs están vivas y qué puertos tienen abiertos.
*   **Ataques/Pruebas:** Inundación de red, ARP Spoofing (solo en entornos controlados/propios).

### La Sintaxis Única: El operador `/`

Lo más distintivo de `scapy` es cómo construyes un paquete. Usas el operador de división `/` para **apilar capas**.

Imagina un paquete como una muñeca rusa (o una cebolla):
1.  **Capa Ethernet** (Cable)
2.  **Capa IP** (Dirección)
3.  **Capa TCP/UDP** (Puerto)
4.  **Payload** (Datos: "Hola")

```python
# Sintaxis Scapy
paquete = Ether() / IP(dst="8.8.8.8") / TCP(dport=80) / "GET / HTTP/1.0"
```

### Funciones Principales

Aquí hay un vocabulario nuevo. Scapy distingue entre trabajar en **Capa 3** (IP, lo normal) y **Capa 2** (Ethernet/MAC, bajo nivel).

##### Enviar Paquetes (Send)

| Función      | Descripción                                                                         |
| :----------- | :---------------------------------------------------------------------------------- |
| `send(pkt)`  | Envía paquetes en **Capa 3** (IP). Scapy se encarga de la MAC y el enrutamiento.    |
| `sendp(pkt)` | Envía paquetes en **Capa 2** (Ethernet). Tú debes especificar la interfaz y la MAC. |

##### Enviar y Recibir (Send and Receive - SR)

Estas son las más útiles. Envías una pregunta y esperas respuesta.

| Función    | Descripción                                                                                                 |
| :--------- | :---------------------------------------------------------------------------------------------------------- |
| `sr(pkt)`  | Envía y recibe respuestas (devuelve dos listas: contestados y no contestados).                              |
| `sr1(pkt)` | **S**end and **R**eceive **1**. Envía el paquete y devuelve **solo la primera respuesta**. Ideal para Ping. |
| `srp(pkt)` | Igual que `sr`, pero para **Capa 2** (ej: Escaneos ARP en red local).                                       |

##### Espiar (Sniffing)

| Función                      | Descripción                                                                                |
| :--------------------------- | :----------------------------------------------------------------------------------------- |
| `sniff(count=N, filter=...)` | Captura paquetes de la red. Puedes filtrar (ej: "tcp port 80") o definir cuántos capturar. |

### Ejemplos de Código

##### Ejemplo 1: Crear y Ver un Paquete

Antes de enviar nada, aprendamos a armarlo.

```python
from scapy.all import *

# Construimos un paquete IP hacia Google, con protocolo ICMP (Ping)
# IP() crea la cabecera IP por defecto
# ICMP() crea la cabecera ICMP por defecto
mi_paquete = IP(dst="8.8.8.8") / ICMP()

# .show() nos muestra la estructura interna detallada
mi_paquete.show()

# .summary() muestra una línea resumen
print(f"Resumen: {mi_paquete.summary()}")
```

##### Ejemplo 2: Hacer un PING manual (`sr1`)

Vamos a replicar el comando `ping` del sistema, pero controlando nosotros el paquete.

```python
from scapy.all import *

ip_destino = "8.8.8.8" # Google DNS

print(f"Haciendo ping a {ip_destino}...")

# Creamos el paquete
paquete = IP(dst=ip_destino) / ICMP()

# Enviamos y esperamos 1 respuesta (timeout de 2 segs para no bloquear)
respuesta = sr1(paquete, timeout=2, verbose=0) # verbose=0 quita mensajes basura

if respuesta:
    print(f"¡Respuesta recibida desde {respuesta[IP].src}!")
    # Podemos ver el tipo de respuesta (0 = Echo Reply)
    print(f"Tipo ICMP: {respuesta[ICMP].type}")
else:
    print("No hubo respuesta (Time out).")
```

##### Ejemplo 3: Escáner de Puertos TCP (SYN Scan)

Así es como funcionan herramientas como *Nmap*.
Enviamos un paquete con la bandera **SYN** (quiero conectar).
*   Si responde **SYN-ACK**: Puerto Abierto.
*   Si responde **RST** (Reset): Puerto Cerrado.

```python
from scapy.all import *

target = "scanme.nmap.org" # Servidor legal para pruebas
puerto = 80

# Flags="S" significa SYN (Synchronize)
paquete_syn = IP(dst=target) / TCP(dport=puerto, flags="S")

# Enviamos y esperamos respuesta
resp = sr1(paquete_syn, timeout=2, verbose=0)

if resp:
    # Verificamos las banderas de la respuesta
    # 'SA' significa SYN-ACK (Abierto)
    if resp.haslayer(TCP) and resp[TCP].flags == "SA":
        print(f"El puerto {puerto} está ABIERTO [+]")
        
        # Buena educación: Enviamos un RST para cerrar la conexión que dejamos a medias
        send(IP(dst=target)/TCP(dport=puerto, flags="R"), verbose=0)
    elif resp.haslayer(TCP) and resp[TCP].flags == "RA":
        print(f"El puerto {puerto} está CERRADO (Reset recibido) [x]")
else:
    print(f"El puerto {puerto} está FILTRADO (Silencio total) [?]")
```

##### Ejemplo 4: Sniffer de Red (Wireshark en Python)

Vamos a leer el tráfico que pasa por tu tarjeta de red en tiempo real.

```python
from scapy.all import *

# Esta función se ejecutará por cada paquete capturado
def procesar_paquete(pkt):
    if pkt.haslayer(IP):
        src = pkt[IP].src
        dst = pkt[IP].dst
        print(f"Paquete detectado: {src} --> {dst}")

print("Escuchando tráfico (Ctrl+C para parar)...")

# sniff bloquea el programa.
# prn: función a llamar por cada paquete
# count: parar después de 10 paquetes (quita esto para infinito)
sniff(prn=procesar_paquete, count=10) 
```

### Diferencia Clave: `socket` vs `scapy`

Esta distinción es vital para entender dónde estás parado.

1.  **`socket` (Nivel Transporte/Aplicación):**
    *   Le dices al sistema operativo: "Abre una conexión a Google".
    *   El sistema operativo construye el paquete Ethernet, IP y TCP por ti. Tú solo mandas datos.
    *   *No puedes* cambiar tu IP de origen (falsificar) fácilmente.

2.  **`scapy` (Nivel Enlace/Red):**
    *   Tú eres el sistema operativo.
    *   Tú construyes el paquete a mano. Tienes que definir qué IP va, qué puerto, qué banderas TCP.
    *   Puedes poner cualquier IP de origen (Spoofing).
    *   Puedes enviar paquetes "rotos" o malformados para ver cómo reacciona un servidor.
