# Módulo 16 — Intro to Network Traffic Analysis

## Sección 11/15: Familiarización con Wireshark (Lab)

## 🎯 Escenario del laboratorio

> [!NOTE]
> **Situación**
> Un usuario reporta **tiempos de carga inusualmente largos** al navegar la web, "como si la red estuviera saturada y el ordenador hiciera muchas conexiones diferentes". Tarea: capturar tráfico del portátil mientras navega y determinar si algo anda mal.

## 🛠️ Flujo de trabajo del laboratorio

### Tarea 1 — Validar instalación y explorar la GUI
```bash
which wireshark
```
> [!TIP]
> **Foco especial**
> Prestar atención a la pestaña **Capture** y qué opciones contiene (interfaces, opciones de captura, filtros de captura).

### Tarea 2 — Seleccionar interfaz activa
Elegir la interfaz activa (`eth0`, tarjeta WiFi, etc.) desde la pantalla de inicio de Wireshark.

### Tarea 3 — Crear filtro de captura para la IP del host
```bash
# Filtro de captura (sintaxis BPF) — solo tráfico desde/hacia la IP del host
host <IP_DEL_PORTATIL>
```
> [!NOTE]
> **Por qué filtrar así**
> El escenario pide enfocarse **solo en el portátil en cuestión** — descartar todo lo demás en la red reduce el ruido desde el inicio.

### Tarea 4 — Generar tráfico de prueba
1. Navegar a `pepsi.com`
2. Navegar a `http://apache.org`
3. Volver a Wireshark → debería verse tráfico fluyendo
4. Una vez cargada la página, **Stop** (cuadrado rojo en la barra de acciones)

### Tarea 5 — Analizar resultados

> [!NOTE]
> **Preguntas guía (para responder con tu propia captura)**
> - ¿Se establecen **múltiples sesiones** entre el host y el servidor web? → Buscar varios handshakes TCP (`SYN`→`SYN,ACK`→`ACK`) simultáneos hacia la misma IP/dominio o hacia distintos recursos de la misma página
> - ¿Qué **protocolos de aplicación** aparecen? → Típicamente DNS, HTTP, HTTPS/TLS (columna Protocolo en Wireshark)
> - ¿Hay algo visible en **texto plano**? → HTTP (no HTTPS) expone contenido legible: headers, HTML, formularios, etc.

## 💡 Contexto clave: por qué "múltiples conexiones" no es necesariamente un problema

> [!TIP]
> **Interpretación relevante para el escenario**
> Una página web moderna (como `pepsi.com`) carga **decenas de recursos en paralelo**: imágenes, scripts, fuentes, trackers de terceros, anuncios — cada uno puede requerir su propia conexión TCP/TLS a distintos dominios (CDNs, Google Fonts, analytics, etc.).
>
> Esto **explica el síntoma reportado** ("muchas conexiones diferentes") sin que sea necesariamente un problema de seguridad o de red — es el comportamiento normal de sitios con muchos recursos externos. Esto conecta directamente con el concepto de **baseline** visto en secciones anteriores: sin conocer qué es "normal" para la navegación web moderna, este patrón podría parecer sospechoso cuando en realidad no lo es.

## 📚 Recurso adicional para seguir practicando

> [!TIP]
> **PCAPs de muestra**
> [Wireshark Sample Captures](https://wiki.wireshark.org/SampleCaptures) — incluye tráfico de ICS/SCADA, malware, y muchos otros protocolos para practicar análisis.

## 🔗 Relacionado
- [10-wireshark-analysis](10-wireshark-analysis.md)
- [12-wireshark-advanced-usage](12-wireshark-advanced-usage.md)
- *Baseline de Red - Notas*

#cjca #modulo16 #wireshark #laboratorio #filtros-captura #baseline
