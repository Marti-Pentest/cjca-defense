# Módulo 16 — Intro to Network Traffic Analysis

## Sección 7/15: Laboratorio de Fundamentos (Capturing With Tcpdump)

## 📌 Objetivo del laboratorio
> [!note] Escenario
> Como nuevo administrador de red de "la Corporación", capturar tráfico del broadcast domain local para **establecer una línea base** y validar que las herramientas/dependencias funcionan correctamente.

## 🛠️ Tareas y comandos

### Tarea 1 — Validar instalación de tcpdump
```bash
which tcpdump
```

### Tarea 2 — Listar interfaces disponibles
```bash
sudo tcpdump -D
```

### Tarea 3 — Filtros de captura básicos

```bash
# Verbosidad + salida en ASCII y hexadecimal
sudo tcpdump -i eth0 -v -X

# Sin resolución de nombres + números de secuencia relativos (por defecto)
sudo tcpdump -i eth0 -nn
```

> [!tip] Recordatorio
> Los números de secuencia son **relativos** por defecto en tcpdump. Para verlos en absoluto se usa `-S`.

### Tarea 4 — Guardar captura en archivo PCAP
```bash
sudo tcpdump -i eth0 -w ~/baseline_capture.pcap
```

### Tarea 5 — Leer captura desde archivo PCAP
```bash
# Sin resolución de nombres + números de secuencia/ACK absolutos
sudo tcpdump -r ~/baseline_capture.pcap -nn -S
```

## 🧠 Quiz de repaso (Q&A del módulo)

> [!faq]- ¿Qué switch permite hacer pipe del contenido de un pcap hacia otra función como `grep`?
> ```bash
> sudo tcpdump -r archivo.pcap -l | grep "patron"
> ```
> El switch clave es **`-l`** (line-buffered output), que permite que la salida se procese en tiempo real por otro comando en un pipe.

> [!faq]- ¿Verdadero o Falso: el filtro "port" mira tráfico de origen Y destino?
> **Verdadero** — `port 80` coincide con tráfico donde el puerto 80 es origen o destino.

> [!faq]- ¿Qué filtro usarías para descartar tráfico ICMP de la captura? (palabra, no símbolo)
> ```bash
> sudo tcpdump -i eth0 not icmp
> ```
> Filtro: **`not icmp`**

> [!faq]- ¿Qué comando muestra dónde/si tcpdump está instalado?
> ```bash
> which tcpdump
> ```

> [!faq]- ¿Cómo se inicia una captura en eth0?
> ```bash
> sudo tcpdump -i eth0
> ```

> [!faq]- ¿Qué switch da más verbosidad?
> `-v` (o `-vv` / `-vvv`)

> [!faq]- ¿Qué switch escribe la salida a un archivo .pcap?
> `-w`

> [!faq]- ¿Qué switch lee una captura desde un archivo .pcap?
> `-r`

> [!faq]- ¿Qué switch muestra el contenido en Hex y ASCII?
> `-X`

## 💡 Notas de aplicación práctica
> [!tip] Uso real de esta técnica
> - Examinar hosts/servidores específicos y con quién interactúan
> - Identificar **backdoors** o brechas de seguridad
> - Monitorear/registrar toda la comunicación de un servidor para análisis posterior con filtros y patrones

## 🔗 Relacionado
- [[06-tcpdump-fundamentals]]
- [[08-tcpdump-packet-filtering]]
- [[Tcpdump Cheatsheet - Switches]]

#cjca #modulo16 #tcpdump #laboratorio #pcap #baseline
