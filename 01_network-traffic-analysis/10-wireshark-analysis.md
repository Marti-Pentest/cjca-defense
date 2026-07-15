# Módulo 16 — Intro to Network Traffic Analysis

## Sección 10/15: Análisis con Wireshark

## 📌 ¿Qué es Wireshark?

> [!note] Definición
> Analizador de tráfico de red **gratuito y open-source**, similar a tcpdump pero con GUI. Multiplataforma, captura de muchas interfaces (WiFi, USB, Bluetooth), permite **deep packet inspection** en cientos de protocolos.

### Características clave
- Inspección profunda de paquetes para cientos de protocolos
- GUI y TTY (variantes de línea de comandos)
- Soporta Ethernet, 802.11, PPP/HDLC, ATM, Bluetooth, USB, Token Ring, Frame Relay, FDDI
- **Descifrado** de IPsec, ISAKMP, Kerberos, SNMPv3, SSL/TLS, WEP, WPA/WPA2

## 🛠️ Requisitos de instalación

> [!note] Windows
> - Universal C Runtime (incluido en Win10/Server 2019+)
> - CPU 64-bit AMD64/x86-64 o 32-bit x86
> - 500 MB RAM + 500 MB disco (más para capturas grandes)
> - Descargar desde wireshark.org, validar hash e instalar

```bash
# Linux - verificar instalación
which wireshark

# Instalar si no existe
sudo apt install wireshark
```

## 🔀 TShark vs Wireshark (Terminal vs GUI)

> [!tip] Cuándo usar cada uno
> - **TShark**: ideal para máquinas sin entorno de escritorio, pipe fácil hacia otras herramientas, comparte sintaxis con Wireshark
> - **Wireshark**: experiencia completa con GUI, mejor para análisis visual profundo

### Parámetros básicos de TShark

| Switch | Resultado |
|---|---|
| `-D` | Lista interfaces disponibles y sale |
| `-L` | Lista medios de capa de enlace disponibles |
| `-i` | Elige interfaz de captura. Ej: `-i eth0` |
| `-f` | Filtro de paquetes (sintaxis libpcap), usado **durante** la captura |
| `-c` | Captura N paquetes y sale |
| `-a` | Condición de parada automática (duración, tamaño, N paquetes) |
| `-r` | Lee desde archivo |
| `-w` | Escribe a archivo (formato pcapng) |
| `-P` | Imprime resumen mientras escribe a archivo (`-w`) |
| `-x` | Agrega salida Hex + ASCII |
| `-h` | Menú de ayuda |

```bash
# Ver ayuda completa
tshark -h

# Localizar tshark
which tshark

# Listar interfaces disponibles
tshark -D

# Capturar en interfaz 1 y guardar a archivo
tshark -i 1 -w /tmp/test.pcap

# Seleccionar interfaz y escribir a archivo
sudo tshark -i eth0 -w /tmp/test.pcap

# Aplicar filtro de captura con -f
sudo tshark -i eth0 -f "host 172.16.146.2"
```

```
Capturing on 'eth0'
    1 0.000000000 172.16.146.2 → 172.16.146.1 DNS 70 Standard query 0x0804 A github.com
    2 0.258861645 172.16.146.1 → 172.16.146.2 DNS 86 Standard query response 0x0804 A github.com A 140.82.113.4
    3 0.259866711 172.16.146.2 → 140.82.113.4 TCP 74 48256 → 443 [SYN] Seq=0 Win=64240...
    6 0.306888828 172.16.146.2 → 140.82.113.4 TLSv1 579 Client Hello
    7 0.347570701 140.82.113.4 → 172.16.146.2 TLSv1.3 2785 Server Hello, Change Cipher Spec, Application Data...
```

> [!tip] Formato de salida de TShark (más legible que tcpdump)
> Muestra columnas claras: número, tiempo relativo, origen → destino, protocolo, tamaño, e info descriptiva (ej: `[SYN]`, `Client Hello`, `Application Data`)

## 🖥️ Termshark (TUI)

> [!note] ¿Qué es?
> Interfaz de usuario basada en texto (TUI) que da una experiencia similar a Wireshark **dentro de la terminal**. Repo: github.com/gcla/termshark

> [!warning] Comportamiento
> La ventana de Termshark **no se abre hasta detectar tráfico** que coincida con el filtro de captura — hay que darle tiempo si parece que no pasa nada.

## 🖼️ Los 3 paneles de la GUI de Wireshark

| Panel | Color | Función |
|---|---|---|
| **Lista de paquetes** | Naranja | Resumen por paquete: Número, Tiempo, Fuente, Destino, Protocolo, Info |
| **Detalles del paquete** | Azul | Desglose por capas (modelo OSI) — **en orden inverso** (capa inferior arriba) |
| **Bytes del paquete** | Verde | Contenido en ASCII/Hex; resalta el campo seleccionado arriba |

> [!tip] Formato del panel de bytes
> Cada línea muestra: offset de datos + 16 bytes hex + 16 bytes ASCII. Bytes no imprimibles se muestran como `.`

> [!note] Guardar una captura
> `Archivo → Guardar` o desde la barra de herramientas. Wireshark soporta múltiples formatos (usar `.pcap` como estándar).

## 🔀 Filtros de Captura vs Filtros de Visualización

> [!warning] Diferencia crítica
> - **Filtros de captura**: se aplican **antes** de iniciar la captura. Sintaxis BPF (igual que tcpdump). Descartan permanentemente lo que no coincide → menos opciones pero reducen datos en disco.
> - **Filtros de visualización**: se aplican **durante o después** de la captura. Sintaxis propia de Wireshark, mucho más rica. No modifican el archivo original, solo la vista — pero Wireshark debe **reprocesar** los datos cada vez.

> [!warning] Rendimiento con capturas grandes
> Cuantos más paquetes, más tarda Wireshark en aplicar filtros (segundos a minutos). Si el pcap es grande, considerar dividirlo en trozos más pequeños primero.

### Tabla de filtros de captura (sintaxis BPF)

| Filtro | Resultado |
|---|---|
| `host x.x.x.x` | Tráfico de un host específico |
| `net x.x.x.x/24` | Tráfico hacia/desde una red |
| `src/dst net x.x.x.x/24` | Solo origen o solo destino de la red |
| `port #` | Filtra todo excepto el puerto especificado |
| `not port #` | Todo excepto ese puerto |
| `port # and #` | Concatena puertos con AND |
| `portrange x-x` | Tráfico dentro del rango |
| `ip / ether / tcp` | Solo tráfico de esa cabecera de protocolo |
| `broadcast / multicast / unicast` | Tipo específico de tráfico |

**Ejemplo de captura filtrada:** `host 214.15.2.30`

### Tabla de filtros de visualización (sintaxis Wireshark)

| Filtro | Resultado |
|---|---|
| `ip.addr == x.x.x.x` | Tráfico de un host (declaración OR) |
| `ip.addr == x.x.x.x/24` | Tráfico de una red (declaración OR) |
| `ip.src/dst == x.x.x.x` | Tráfico hacia/desde un host específico |
| `dns / tcp / ftp / arp / ip` | Filtra por protocolo específico |
| `tcp.port == x` | Puerto TCP específico |
| `tcp.port / udp.port != x` | Todo excepto ese puerto |
| `and / or / not` | Concatenar / cualquiera / excluir |

> [!warning] Puerto ≠ protocolo
> Filtrar por `port 80` muestra **cualquier tráfico** en ese puerto (sea o no HTTP real). Filtrar por `http` busca marcadores específicos del protocolo (GET/POST, etc.), sin importar el puerto usado. Son conceptos distintos — no confundirlos.

## 🧠 Quiz de repaso (Q&A del módulo)

> [!faq]- ¿Verdadero o Falso: Wireshark puede correr en Windows y Linux?
> **Verdadero**

> [!faq]- ¿Qué panel permite ver un resumen de cada paquete capturado?
> **Lista de paquetes** (Packet List)

> [!faq]- ¿Qué panel muestra el tráfico capturado en ASCII y Hex?
> **Bytes del paquete** (Packet Bytes)

> [!faq]- ¿Qué switch de TShark lista las interfaces posibles para capturar?
> `-D`

> [!faq]- ¿Qué switch permite aplicar filtros en TShark?
> `-f`

> [!faq]- ¿Un filtro de captura se aplica antes o después de iniciar la captura?
> **Antes** (before)

## 🔗 Relacionado
- [[09-tcpdump-lab-interrogating-traffic]]
- [[11-wireshark-familiarity-lab]]
- [[Wireshark Filtros de Captura vs Visualizacion]]

#cjca #modulo16 #wireshark #tshark #termshark #filtros-captura #filtros-visualizacion
