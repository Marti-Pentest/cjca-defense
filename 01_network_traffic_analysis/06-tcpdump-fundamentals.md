# Módulo 16 — Intro to Network Traffic Analysis

## Sección 6/15: Fundamentos de Tcpdump

## 📌 Conceptos clave

> [!NOTE]
> **¿Qué es tcpdump?**
> Sniffer de paquetes de línea de comandos que captura/interpreta tramas de datos desde un archivo o interfaz de red. Usa librerías **pcap/libpcap** + interfaz en **modo promiscuo** para ver todo el tráfico del segmento, no solo el destinado a nosotros.

> [!WARNING]
> **Requisitos**
> - Requiere privilegios de **root/administrador** (`sudo`) por acceso directo al hardware
> - Windows: WinDump está **descontinuado** → usar WSL con Parrot/Ubuntu como alternativa

## 🛠️ Instalación y verificación

```bash
# Verificar si tcpdump existe en el sistema
which tcpdump

# Instalar si no existe (normalmente ya viene preinstalado en Linux)
sudo apt install tcpdump

# Verificar versión
sudo tcpdump --version
```

```
tcpdump version 4.9.3
libpcap version 1.9.1 (with TPACKET_V3)
OpenSSL 1.1.1f  31 Mar 2020
```

## 🛠️ Modificadores básicos (switches)

| Switch | Resultado |
|---|---|
| `-D` | Muestra interfaces disponibles para capturar |
| `-i` | Selecciona interfaz de captura. Ej: `-i eth0` |
| `-n` | No resuelve direcciones a nombres |
| `-nn` | No resuelve direcciones **ni** puertos a nombres |
| `-e` | Captura cabecera Ethernet junto con datos de capa superior |
| `-X` | Muestra contenido en hexadecimal + ASCII |
| `-XX` | Igual que `-X` pero incluye cabeceras Ethernet (equivale a `-Xe`) |
| `-v`, `-vv`, `-vvv` | Aumenta verbosidad |
| `-c` | Captura N paquetes y cierra. Ej: `-c 100` |
| `-s` | Define qué parte del paquete capturar (snaplen) |
| `-S` | Muestra números de secuencia **absolutos** en vez de relativos |
| `-q` | Menos info de protocolo (quiet) |
| `-r file.pcap` | Lee desde archivo |
| `-w file.pcap` | Escribe a archivo |

> [!TIP]
> **Página de manual**
> ```bash
> man tcpdump
> ```

## 🛠️ Comandos de ejemplo

```bash
# Listar interfaces disponibles
sudo tcpdump -D
```
```
1.eth0 [Up, Running, Connected]
2.any (Pseudo-device that captures on all interfaces) [Up, Running]
3.lo [Up, Running, Loopback]
```

```bash
# Elegir interfaz para capturar
sudo tcpdump -i eth0
```

```bash
# Desactivar resolución de nombres (IP y puertos)
sudo tcpdump -i eth0 -nn
```

```bash
# Mostrar cabecera Ethernet (MAC origen/destino)
sudo tcpdump -i eth0 -e
```

```bash
# Mostrar contenido en hex + ASCII
sudo tcpdump -i eth0 -X
```

```bash
# Combinación de modificadores (buena práctica: encadenarlos)
sudo tcpdump -i eth0 -nnvXX
```

```bash
# Guardar captura en archivo PCAP
sudo tcpdump -i eth0 -w ~/output.pcap
```
```
tcpdump: listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
10 packets captured
131 packets received by filter
0 packets dropped by kernel
```

```bash
# Leer desde archivo PCAP
sudo tcpdump -r ~/output.pcap

# Leer archivo mostrando hex + ASCII (buena práctica combinar switches)
sudo tcpdump -r /tmp/capture.pcap -X
```

> [!WARNING]
> **Cuidado con el almacenamiento**
> Capturar tráfico directo del cable con `-w` puede agotar rápido el espacio en disco, especialmente en segmentos grandes. Usar modificadores (`-c`, `-s`) para limitar el volumen.

## 📖 Desglose de la salida de tcpdump

| Campo | Color (en la doc original) | Descripción |
|---|---|---|
| Timestamp | Amarillo | Hora/fecha del paquete |
| Protocolo | Naranja | Cabecera de capa superior (ej: IP) |
| IP y puerto origen/destino | Naranja | Formato `IP.puerto` ej: `172.16.146.2.21` |
| Flags | Verde | Banderas usadas (SYN, ACK, etc.) |
| Números de secuencia/ACK | Rojo | Rastreo del segmento TCP (relativos por defecto, absolutos con `-S`) |
| Opciones de protocolo | Azul | Valores TCP negociados (window size, SACK, scale factor) |
| Notas / siguiente cabecera | Blanco | Info del disector, ej: reconocimiento de tráfico FTP encapsulado |

> [!TIP]
> **Idea avanzada**
> Se puede usar tcpdump como base de un **IDS/IPS casero**: un script bash que analice paquetes interceptados según un patrón (ej: banear IP tras demasiados ICMP echo requests en cierto tiempo).

## 🧠 Quiz de repaso (Q&A del módulo)

<details>
<summary>¿Qué switch inicia captura sin resolución de nombres, verbose, en ASCII+hex, y captura los primeros 100 paquetes?</summary>

```bash
sudo tcpdump -nn -v -X -c 100
```

</details>


<details>
<summary>Comando para leer `/tmp/capture.pcap` mostrando hex y ASCII (buenas prácticas)</summary>

```bash
sudo tcpdump -r /tmp/capture.pcap -X
```

</details>


<details>
<summary>¿Qué switch aumenta la verbosidad?</summary>

`-v` (o `-vv`, `-vvv` para más detalle)

</details>


<details>
<summary>¿Qué referencia de ayuda del terminal da más info sobre tcpdump?</summary>

`man tcpdump`

</details>


<details>
<summary>¿Qué switch permite escribir la salida a un archivo?</summary>

`-w`

</details>


## 🔗 Relacionado
- [05-analysis-practice](05-analysis-practice.md)
- [07-tcpdump-lab-fundamentals](07-tcpdump-lab-fundamentals.md)
- *Tcpdump Cheatsheet - Switches*

#cjca #modulo16 #tcpdump #packet-capture #pcap #cli
