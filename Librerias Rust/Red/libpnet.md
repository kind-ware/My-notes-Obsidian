## Descripción General

`pnet` es una librería multiplataforma para el manejo de redes a bajo nivel. A diferencia de `std::net`, que trabaja con sockets de alto nivel (TCP/UDP), `pnet` permite acceder a la **capa de enlace de datos** (Data Link Layer).

Proporciona herramientas para:
1.  **Captura de paquetes:** Leer frames directamente de la interfaz de red.
2.  **Manipulación/Parsing:** Estructuras para "entender" los headers de los protocolos.
3.  **Inyección:** Enviar paquetes personalizados (spoofing).

### Casos de Uso

*   **Sniffers de red:** Monitorización de tráfico y diagnóstico.
*   **Sistemas de Detección de Intrusos (IDS):** Analizar patrones de tráfico sospechosos.
*   **Implementación de Protocolos:** Crear protocolos propios o no soportados por el SO (ej. ICMP personalizado).
*   **Simulación de Redes:** Herramientas de pentesting o escaneo de puertos (estilo Nmap).

## Módulos y Funciones Principales

*   **`datalink`**: Permite listar las interfaces de red y crear un canal de comunicación con el hardware.
*   **`packet`**: Contiene submódulos para cada protocolo (`ethernet`, `ipv4`, `tcp`, `udp`, `icmp`). Cada uno tiene:
    *   Un `Packet` (para leer).
    *   Un `MutablePacket` (para construir/modificar).
*   **`datalink::channel(...)`**: Crea un par emisor/receptor para la interfaz seleccionada.

## Ejemplos de Código

### Preparación (Cargo.toml)
```toml
[dependencies]
pnet = "0.35"
```

### Ejemplo 1: Listar Interfaces de Red
Antes de capturar, necesitas saber qué interfaces (Wi-Fi, Ethernet, Loopback) tienes disponibles.

```rust
use pnet::datalink;

fn main() {
    let interfaces = datalink::interfaces();
    
    println!("Interfaces detectadas:");
    for interface in interfaces {
        println!("Nombre: {} | IP: {:?}", interface.name, interface.ips);
    }
}
```

### Ejemplo 2: Sniffer Simple (Captura de Headers Ethernet)
Este código escucha en una interfaz y nos dice quién envía paquetes a quién.

```rust
use pnet::datalink::{self, Channel::Ethernet};
use pnet::packet::ethernet::EthernetPacket;

fn main() {
    // 1. Seleccionar la primera interfaz disponible (ejemplo)
    let interface = datalink::interfaces().into_iter()
        .find(|iface| !iface.is_loopback() && iface.is_up())
        .expect("No se encontró una interfaz activa");

    // 2. Crear el canal para capturar paquetes
    let (_, mut rx) = match datalink::channel(&interface, Default::default()) {
        Ok(Ethernet(tx, rx)) => (tx, rx),
        Ok(_) => panic!("Tipo de canal no soportado"),
        Err(e) => panic!("Error al crear el canal: {}", e),
    };

    println!("Iniciando sniffer en {}...", interface.name);

    loop {
        match rx.next() {
            Ok(packet) => {
                let eth_packet = EthernetPacket::new(packet).unwrap();
                println!(
                    "Frame: [Origen: {} -> Destino: {}] Tipo: {:?}",
                    eth_packet.get_source(),
                    eth_packet.get_destination(),
                    eth_packet.get_ethertype()
                );
            },
            Err(e) => panic!("Error al recibir paquete: {}", e),
        }
    }
}
```

### Ejemplo 3: Deep Packet Inspection (Parsing IPv4)
Cómo "entrar" dentro del paquete Ethernet para ver la información de IP.

```rust
use pnet::packet::ethernet::{EthernetPacket, EtherTypes};
use pnet::packet::ipv4::Ipv4Packet;
use pnet::packet::Packet;

fn handle_packet(ethernet: &EthernetPacket) {
    match ethernet.get_ethertype() {
        EtherTypes::Ipv4 => {
            if let Some(header) = Ipv4Packet::new(ethernet.payload()) {
                println!(
                    "  IP: {} -> {} | Protocolo: {:?}",
                    header.get_source(),
                    header.get_destination(),
                    header.get_next_level_protocol()
                );
            }
        },
        _ => {} // Ignorar otros (IPv6, ARP, etc.)
    }
}
// (Este handler se llamaría dentro del loop del ejemplo anterior)
```

## Buenas Prácticas y Consideraciones

1.  **Privilegios Elevados:** Capturar paquetes requiere acceder directamente al hardware. **Debes ejecutar tu programa como Root (Linux/macOS) o Administrador (Windows)**.
2.  **Requisitos en Windows:** Necesitas tener instalado **Npcap** (sucesor de WinPcap). Además, debes configurar el modo de compatibilidad de Npcap si usas Windows moderno.
3.  **Rendimiento:** Capturar cada paquete en una red de alta velocidad (10Gbps+) puede saturar un solo hilo de Rust. Considera usar buffers grandes y procesar los paquetes en un pool de hilos o de forma muy ligera.
4.  **Seguridad:** Ten mucho cuidado al parsear paquetes. Un paquete malformado diseñado maliciosamente podría causar un pánico en tu programa si no manejas bien los `unwrap()`. `pnet` devuelve `Option` en la mayoría de sus constructores de paquetes por esta razón.
5.  **Modo Promiscuo:** Por defecto, solo verás paquetes dirigidos a tu tarjeta de red. Si quieres ver TODO el tráfico del segmento de red (Switch/Hub), debes activar el modo promiscuo en la configuración del canal:

```rust
let mut config = datalink::Config::default();
config.promiscuous = true;
```
