# Módulo 16 — Intro to Network Traffic Analysis

## Sección 8/15: Filtrado de Paquetes con Tcpdump

## 📌 Tabla de filtros útiles

| Filtro | Resultado |
|---|---|
| `host` | Filtra tráfico bidireccional hacia/desde el host especificado |
| `src` / `dst` | Designa host/puerto de **origen** o **destino** específicamente |
| `net` | Tráfico hacia/desde una red (notación `/`) |
| `proto` | Filtra por protocolo (ether, tcp, udp, icmp) — nombre o número |
| `port` | Bidireccional, muestra tráfico con ese puerto como origen o destino |
| `portrange` | Rango de puertos, ej: `0-1024` |
| `less` / `greater` (`<` `>`) | Filtra por tamaño de paquete |
| `and` / `&&` | Concatena dos filtros — **ambos** deben cumplirse |
| `or` / `\|\|` | Coincide con **cualquiera** de dos condiciones |
| `not` / `!` | Excluye una condición ("todo excepto x") |

## 🛠️ Ejemplos de comandos

```bash
# Filtro host (bidireccional)
sudo tcpdump -i eth0 host 172.16.146.2

# Filtro src (solo origen)
sudo tcpdump -i eth0 src host 172.16.146.2

# Filtro src + puerto (solo un lado de la conversación)
sudo tcpdump -i eth0 tcp src port 80

# Filtro dest + net (notación CIDR)
sudo tcpdump -i eth0 dest net 172.16.146.0/24

# Filtro de protocolo — nombre común
sudo tcpdump -i eth0 udp

# Filtro de protocolo — número (17 = UDP)
sudo tcpdump -i eth0 proto 17

# Filtro de puerto (asegura solo tráfico HTTP real, no cualquier cosa en puerto 80)
sudo tcpdump -i eth0 tcp port 443

# Filtro portrange
sudo tcpdump -i eth0 portrange 0-1024

# Filtro less (paquetes SYN/FIN/KeepAlive suelen ser pequeños)
sudo tcpdump -i eth0 less 64

# Filtro greater (útil para detectar transferencias de archivos grandes)
sudo tcpdump -i eth0 greater 500

# Filtro AND (ambas condiciones deben cumplirse)
sudo tcpdump -i eth0 host 192.168.0.1 and port 23

# Filtro OR (cualquiera de las dos condiciones)
sudo tcpdump -r sus.pcap icmp or host 172.16.146.1

# Filtro NOT (excluir ICMP)
sudo tcpdump -r sus.pcap not icmp
```

> [!TIP]
> **TCP vs puerto genérico**
> `port 80` muestra **cualquier** tráfico en el puerto 80 (aunque no sea HTTP). `tcp port 80` asegura que sea específicamente tráfico TCP en ese puerto — más preciso para aislar el protocolo real.

> [!TIP]
> **DNS usa TCP y UDP**
> Para protocolos híbridos como DNS (puerto 53): usar `tcp/udp port 53` para especificar transporte, o solo `port 53` para ver ambos.

## ⚠️ Filtros pre-captura vs. post-captura

> [!WARNING]
> **Diferencia importante**
> - **Pre-captura** (al capturar en vivo): descarta permanentemente el tráfico que no coincide → riesgo de perder datos útiles. Usar solo cuando se busca algo específico.
> - **Post-captura** (al leer un `.pcap` con `-r`): el filtro solo afecta la **vista en terminal**, el archivo original queda intacto. Se puede re-ejecutar con otro filtro sin perder datos.

## 🎯 Tips de interpretación avanzada

| Modificador | Efecto |
|---|---|
| `-S` | Números de secuencia **absolutos** (largos) — útil para buscar en otras herramientas/logs |
| `-v`, `-X`, `-e` | **Aumentan** cantidad de datos mostrados |
| `-c`, `-n`, `-s`, `-S`, `-q` | **Reducen/modifican** cantidad de datos vistos |
| `-A` | Solo texto ASCII (sin hex) — útil para buscar contenido legible rápido |
| `-l` | Line-buffered output — permite pipe hacia otras herramientas (ej: `grep`) |

```bash
# Ver solo ASCII de una captura (rápido para leer contenido humano)
sudo tcpdump -Ar telnet.pcap

# Pipe hacia grep para buscar patrones específicos
sudo tcpdump -Ar http.cap -l | grep 'mailto:*'
```

## 🚩 Filtrado por flags de protocolo (nivel de bit)

> [!NOTE]
> **Sintaxis avanzada — inspección de bytes específicos**
> Se puede filtrar por bits individuales dentro del header TCP usando notación `protocolo[byte] & valor`.

```bash
# Buscar paquetes con el flag SYN activado
# (byte 13 del header TCP, bit 2 = SYN)
sudo tcpdump -i eth0 'tcp[13] &2 != 0'
```

> [!TIP]
> **Cómo se lee esto**
> Cuenta hasta el byte 13 de la estructura TCP y evalúa el bit 2. Si está en 1 (ON), el flag SYN está activo.

## 📚 RFCs de referencia

| Protocolo | RFC |
|---|---|
| IP | RFC 791 |
| ICMP | RFC 792 |
| TCP | RFC 793 |
| UDP | RFC 768 |

## 🧠 Quiz de repaso (Q&A del módulo)

<details>
<summary>¿Qué filtro permite ver tráfico desde o hacia el host 10.10.20.1?</summary>

```bash
host 10.10.20.1
```

</details>


<details>
<summary>¿Qué filtro permite capturar basado en cualquiera de dos opciones?</summary>

`or` (o `||`)

</details>


<details>
<summary>¿Verdadero o Falso: tcpdump resuelve IPs a hostnames por defecto?</summary>

**Verdadero** (por eso se usa `-n`/`-nn` para desactivarlo)

</details>


## 🔗 Relacionado
- [06-tcpdump-fundamentals](06-tcpdump-fundamentals.md)
- [07-tcpdump-lab-fundamentals](07-tcpdump-lab-fundamentals.md)
- [09-tcpdump-lab-interrogating-traffic](09-tcpdump-lab-interrogating-traffic.md)
- *Tcpdump Cheatsheet - Switches*

#cjca #modulo16 #tcpdump #filtros #bpf #packet-filtering
