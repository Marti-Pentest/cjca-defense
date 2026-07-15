# Módulo 16 — Intro to Network Traffic Analysis

## Sección 9/15: Interrogar el tráfico de red con filtros de captura y visualización (Lab)

> [!WARNING]
> **Nota sobre este resumen**
> Este laboratorio usa un archivo `TCPDump-lab-2.zip` específico. No se incluyen las respuestas exactas del lab (IPs, timestamps concretos) por política de no compartir soluciones — se documenta la **metodología y comandos** para replicar el proceso con el pcap real.

## 🎯 Escenario
> [!NOTE]
> **Objetivo del lab**
> Determinar qué servidores están respondiendo a solicitudes **DNS** y **HTTP/S** en la red local, a partir de un pcap ya capturado.

## 🛠️ Flujo de trabajo del laboratorio (comandos por tarea)

### Tarea 1 — Leer captura sin filtros
```bash
sudo tcpdump -r TCPDump-lab-2.pcap
```

### Tarea 2 — Identificar tipo de tráfico
```bash
# Ver con más detalle para identificar protocolos y puertos
sudo tcpdump -r TCPDump-lab-2.pcap -nn
```
> [!TIP]
> **Qué observar**
> - Protocolos comunes (HTTP, DNS, etc.)
> - Puertos usados (53, 80, 443, etc.)
> - Volumen y variedad de hosts

### Tarea 3 — Identificar conversaciones (patrón cliente-servidor)
```bash
# Ver el handshake completo con puertos
sudo tcpdump -r TCPDump-lab-2.pcap -nn tcp
```
> [!TIP]
> **Cómo identificar servidor vs cliente**
> - El **servidor** normalmente responde desde un **puerto bien conocido** (53=DNS, 80=HTTP, 443=HTTPS)
> - El **cliente** usa un puerto alto aleatorio (ej: 49xxx, 55xxx)
> - Buscar el primer `SYN` → `SYN,ACK` → `ACK` para identificar quién inicia y quién responde

### Tarea 4 — Interpretar la captura en profundidad
```bash
# Ver timestamps y protocolo de la primera conversación
sudo tcpdump -r TCPDump-lab-2.pcap -nn -c 5

# Buscar respuestas DNS para un dominio específico (ej: apache.org)
sudo tcpdump -r TCPDump-lab-2.pcap -nn port 53
```

### Tarea 5 — Filtrar solo tráfico DNS
```bash
sudo tcpdump -r TCPDump-lab-2.pcap -nn port 53
# o específicamente
sudo tcpdump -r TCPDump-lab-2.pcap -nn udp port 53
```
> [!TIP]
> **Qué buscar**
> - **Servidor DNS del segmento**: la IP que responde consistentemente en el puerto 53
> - **Dominios solicitados**: nombres en las queries (`A?`, `PTR?`, etc.)
> - **Tipos de registro**: A (IPv4), PTR (reverse lookup), AAAA (IPv6), CNAME (alias)

### Tarea 6 — Filtrar tráfico TCP (HTTP/HTTPS)
```bash
sudo tcpdump -r TCPDump-lab-2.pcap -nn tcp port 80 or tcp port 443
```
> [!TIP]
> **Qué buscar**
> - Páginas solicitadas → visible en el contenido HTTP (con `-A` para ver texto)
> - Métodos HTTP más comunes → buscar cadenas `GET`, `POST`, etc.
> - Código de respuesta más común → buscar `200 OK`, `404`, etc.

### Tarea 7 — Determinar la aplicación del servidor web
```bash
# Ver en hex + ASCII para inspeccionar headers HTTP (Server:, banner, etc.)
sudo tcpdump -r TCPDump-lab-2.pcap -nn -X tcp port 80
```
> [!TIP]
> **Qué buscar**
> El header HTTP `Server:` en la respuesta suele revelar la aplicación/versión (ej: `Apache/2.4.x`, `nginx`, `Microsoft-IIS`). Esto también puede ser una pista de **fingerprinting** — relevante tanto en ofensiva (recon) como en defensiva (inventario de activos).

## 🔍 Ejemplo real de interpretación (práctica propia con este pcap)

Al analizar el tráfico capturado, se identificó el **three-way handshake TCP completo**:

```bash
sudo tcpdump -r TCPDump-lab-2.pcap -nn
```

Patrón observado en varios intentos de conexión hacia dos IPs candidatas de `apache.org` (resuelto vía DNS con dos registros A):

```
# Intento fallido (termina en RESET, no ACK final):
...43804 > 95.216.26.30.80: Flags [S]           # SYN
...95.216.26.30.80 > ...43804: Flags [S.]       # SYN-ACK
...43804 > 95.216.26.30.80: Flags [R]           # ❌ RESET

# Intento que SÍ completa el handshake:
...43806 > 95.216.26.30.80: Flags [S]           # SYN
...95.216.26.30.80 > ...43806: Flags [S.]       # SYN-ACK
...43806 > 95.216.26.30.80: Flags [.], ack 1     # ✅ ACK final
```

> [!NOTE]
> **Por qué aparecen conexiones con RESET**
> Cuando DNS devuelve **múltiples IPs** para un mismo dominio, el cliente (navegador) puede probar varias conexiones en paralelo, quedándose con la más rápida y **reseteando (RST)** las demás. Este es un comportamiento normal ("happy eyeballs" / conexiones especulativas), **no necesariamente malicioso**.

Para identificar el servidor DNS del segmento:
```bash
sudo tcpdump -r TCPDump-lab-2.pcap -nn port 53
```
Se busca la IP que responde consistentemente en el puerto 53 a las consultas del cliente — típicamente la puerta de enlace local (ej: `172.16.146.1` en el laboratorio de práctica).

## 🧠 Preguntas guía para cualquier análisis de PCAP (checklist reutilizable)

> [!NOTE]
> **Plantilla de análisis (aplica a cualquier pcap futuro)**
- [ ] ¿Qué tipo de tráfico veo? (protocolo, puerto)
- [ ] ¿Hay más de una conversación? ¿Cuántas?
- [ ] ¿Cuántos hosts únicos hay?
- [ ] ¿Cuál es el timestamp de la primera conversación TCP?
- [ ] ¿Qué tráfico puedo filtrar para limpiar la vista?
- [ ] ¿Quiénes son los servidores? (responden en puertos conocidos: 53, 80, 443...)
- [ ] ¿Qué registros/métodos se solicitaron? (GET, POST, registros A de DNS...)

## 🔗 Relacionado
- [08-tcpdump-packet-filtering](08-tcpdump-packet-filtering.md)
- [10-wireshark-analysis](10-wireshark-analysis.md)
- *Checklist Analisis PCAP*

#cjca #modulo16 #tcpdump #laboratorio #dns #http #pcap-analysis
