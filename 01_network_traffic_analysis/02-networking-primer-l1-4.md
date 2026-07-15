# Módulo 16 — Intro to Network Traffic Analysis

## Sección 2/15: Manual básico de redes: Capas 1-4

## 📌 Modelos OSI vs TCP/IP

| Característica | OSI | TCP/IP |
|---|---|---|
| Capas | 7 | 4 |
| Flexibilidad | Estricto | Flexible |
| Dependencia | Independiente del protocolo, genérico | Basado en protocolos comunes |

> [!NOTE]
> **Mapeo de capas**
> - OSI 7,6,5 (Aplicación, Presentación, Sesión) → TCP/IP 4 (Aplicación)
> - OSI 4 (Transporte) → TCP/IP 3 (Transporte)
> - OSI 3 (Red) → TCP/IP 2 (Internet)
> - OSI 2,1 (Enlace de Datos, Física) → TCP/IP 1 (Enlace)

> [!TIP]
> **Analogía**
> OSI = la teoría de cómo funciona todo. TCP/IP = cómo funciona realmente en la práctica (más combinado, reglas flexibles).

## 📦 PDU (Protocol Data Unit) y encapsulación

| Capa | Tipo de PDU |
|---|---|
| Aplicación/Presentación/Sesión | Datos |
| Transporte | Segmento (TCP) / Datagrama (UDP) |
| Red/Internet | Paquete |
| Enlace de Datos/Enlace | Trama (Frame) |
| Física | Bit |

> [!WARNING]
> **Orden inverso en Wireshark**
> Wireshark muestra la PDU **desencapsulada en orden inverso** (de la capa física hacia la de aplicación) porque así es como se procesa al recibir el paquete.

## 🏷️ Mecanismos de direccionamiento

### MAC (Capa 2 - Enlace de Datos)
- 48 bits, 6 octetos, formato **hexadecimal**
- Comunicación host-a-host dentro de un **dominio de difusión (broadcast domain)**
- Al cruzar una interfaz L3, el router reemplaza el encabezado L2 con la siguiente dirección física de la ruta

### IPv4 (Capa 3 - Red / Capa 2 TCP-IP - Internet)
- 32 bits, 4 octetos, formato **decimal** (0-255 cada octeto)
- Ejemplo: `192.168.86.243`
- Sin conexión, no garantiza entrega (depende de TCP para fiabilidad)

### IPv6
- 128 bits, 16 octetos, formato **hexadecimal**
- Ventajas: espacio de direcciones enorme, mejor multicast, direccionamiento global, IPSec integrado, headers simplificados
- Adopción mundial: ~40% (según Google, al momento de escribir el módulo)

| Tipo | Descripción |
|---|---|
| Unicast | Una interfaz |
| Anycast | Múltiples interfaces, solo una responde (balanceo de carga) |
| Multicast | Múltiples interfaces, todas reciben el paquete |
| Broadcast | No existe en IPv6 (se simula con multicast) |

## 🔀 TCP vs UDP

| Característica | TCP | UDP |
|---|---|---|
| Transmisión | Orientado a conexión | Sin conexión ("fire and forget") |
| Establecimiento | Handshake de 3 vías | No verifica si el destino escucha |
| Entrega | Basada en flujo | Paquete por paquete |
| Recepción | Números de secuencia/ACK | No le importa |
| Velocidad | Más lento (overhead) | Rápido, no fiable |

> [!TIP]
> **Cuándo se usa cada uno**
> - **TCP**: SSH, cambios de contraseña remotos, cualquier cosa donde la integridad > velocidad
> - **UDP**: streaming de video, DNS → donde la velocidad > integridad (perder un paquete no es crítico)

## 🛠️ Handshake de 3 vías TCP (SYN, SYN/ACK, ACK)

```
Cliente → Servidor : SYN                (negocia número de secuencia inicial)
Servidor → Cliente : SYN, ACK           (acusa recibo + negocia su propio seq)
Cliente → Servidor : ACK                (confirma, conexión establecida)
```

> [!NOTE]
> **Ejemplo real (captura)**
> Puertos vistos: `57678` (puerto alto aleatorio del cliente) → `80` (HTTP, servidor)
> 1. `SYN` — cliente inicia
> 2. `SYN, ACK` — servidor responde
> 3. `ACK` — cliente confirma → conexión establecida
> 4. Solicitud HTTP enviada, empieza transferencia con ACKs por cada chunk

## 🛠️ Cierre de sesión TCP (FIN)

```
Cliente → Servidor : FIN, ACK
Servidor → Cliente : FIN, ACK
Cliente → Servidor : ACK
```

> [!NOTE]
> **Patrón esperado de cierre correcto**
> ```
> FIN, ACK
> FIN, ACK
> ACK
> ```
> Esto indica una terminación "elegante" (graceful) de la sesión.

## 🧠 Quiz de repaso (Q&A del módulo)

<details>
<summary>¿Cuántas capas tiene el modelo OSI?</summary>

7

</details>


<details>
<summary>¿Cuántas capas tiene el modelo TCP/IP?</summary>

4

</details>


<details>
<summary>¿Verdadero o Falso: los routers operan en la capa 2 del modelo OSI?</summary>

**Falso** — los routers operan en la **capa 3** (Red)

</details>


<details>
<summary>¿Qué mecanismo de direccionamiento se usa en la capa de Enlace (Link Layer) del modelo TCP/IP?</summary>

Direccionamiento **MAC**

</details>


<details>
<summary>¿En qué capa del modelo OSI se encapsula una PDU como "paquete"?</summary>

Capa **3** (Red)

</details>


<details>
<summary>¿Qué mecanismo de direccionamiento usa una dirección de 32 bits?</summary>

**IPv4**

</details>


<details>
<summary>¿Qué protocolo de la capa de Transporte está orientado a conexión?</summary>

**TCP**

</details>


<details>
<summary>¿Qué protocolo de la capa de Transporte se considera no fiable?</summary>

**UDP**

</details>


<details>
<summary>El handshake de 3 vías de TCP consiste en: 1. SYN, 2. SYN/ACK, 3. ¿?</summary>

**ACK**

</details>


## 🔗 Relacionado
- [01-intro-nta](01-intro-nta.md)
- [03-networking-primer-l5-7](03-networking-primer-l5-7.md)
- *OSI vs TCP-IP Cheatsheet*

#cjca #modulo16 #osi #tcpip #tcp #udp #handshake #ipv4 #ipv6
