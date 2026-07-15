# Módulo 16 — Intro to Network Traffic Analysis

## Sección 4/15: El Proceso de Análisis

## 📌 Conceptos clave

> [!NOTE]
> **¿Qué es el Análisis de Tráfico?**
> Examen detallado de un evento/proceso para determinar **origen e impacto**, dividiendo los datos en fragmentos comprensibles y buscando desviaciones del tráfico normal: comunicaciones remotas no autorizadas (RDP, SSH, Telnet), o instancias que preceden problemas de red.

> [!TIP]
> **El valor central: visibilidad**
> Sin capacidad de monitorear tráfico, falta una **pieza enorme del rompecabezas**. Permite:
> - Establecer baseline del entorno
> - Detectar cambios rápidamente
> - Alimentar herramientas avanzadas (IDS/IPS, firewalls, Splunk, ELK)

> [!WARNING]
> **No confiar solo en herramientas automatizadas**
> Las herramientas pueden ser burladas por atacantes. Combinar automatización + **revisión manual** = check & balance. El ojo humano sigue siendo el mejor recurso para detectar comportamiento malicioso.

## 🔀 Captura Activa vs Pasiva

> [!NOTE]
> **Diferencia fundamental**
> - **Pasiva**: copiar datos sin interactuar con los paquetes
> - **Activa** (in-line): enfoque práctico, el tráfico pasa literalmente por nuestro punto de captura

### Tabla de dependencias

| Dependencia | Pasiva | Activa | Descripción |
|---|:---:|:---:|---|
| **Permiso** | ✅ | ✅ | Siempre obtener autorización por escrito — puede ser ilegal sin ella (sanidad, banca, etc.) |
| **Puerto Espejo (Mirror/SPAN)** | ✅ | ❌ | Copia tráfico de otras fuentes a nuestra interfaz + NIC en modo promiscuo. Necesario porque VLANs/switch ports no reenvían fuera de su broadcast domain |
| **Herramienta de Captura** | ✅ | ✅ | tcpdump, Wireshark, Netminer, etc. Cuidado: archivos PCAP grandes + cada filtro en Wireshark re-analiza todo → requiere recursos |
| **Ubicación en línea** | ❌ | ✅ | Requiere cambiar la topología de red — el tap se vuelve un "next hop" invisible |
| **Tap de Red / Host multi-NIC** | ❌ | ✅ | Necesario para que el tráfico siga fluyendo mientras se inspecciona. Mejor ubicación: enlace de capa 3 entre segmentos conmutados |
| **Almacenamiento/Procesamiento** | ✅ | ✅ | Activa (enlace L3) requiere mucho más que pasiva (LAN) |

> [!TIP]
> **Analogía de la sección**
> Captura pasiva en LAN = **llenar un vaso desde una fuente** (flujo manejable). Captura activa en un enlace enrutado (L3) = **llenar una taza de té con una manguera** (mucha más presión/volumen).

## 🎯 Importancia de la línea base (Baseline)

> [!WARNING]
> **Sin baseline, el análisis es abrumador**
> Escenario: segmento con alta latencia + archivos nuevos sospechosos en escritorios.
> - **Sin baseline**: hay que examinar cada conversación manualmente para determinar qué es legítimo vs. rogue asset → proceso lento y agotador
> - **Con baseline**: se descarta rápido el tráfico legítimo conocido → foco inmediato en anomalías

### Ejemplo práctico de detección
> [!NOTE]
> **Caso: dos PCs de usuario hablando entre sí en puertos 8080 y 445**
> - Los puertos **no son raros en sí mismos** (8080 = web alt, 445 = SMB)
> - Lo sospechoso es el **patrón**: tráfico SMB/web normalmente va de host → servidor, no entre dos PCs de usuario
> - Herramienta útil: **"Top Talkers"** en Wireshark → identifica hosts que envían grandes volúmenes de datos, comparándolos contra el baseline

> [!TIP]
> **Regla de oro**
> Cuanto más rápido se obtiene visibilidad ante una intrusión, **menor el daño potencial**. Entender el flujo normal de tráfico y el comportamiento típico de los protocolos es la base de una detección rápida.

## 🔗 Relacionado
- [03-networking-primer-l5-7](03-networking-primer-l5-7.md)
- [05-analysis-practice](05-analysis-practice.md)
- *Baseline de Red - Notas*

#cjca #modulo16 #nta #baseline #captura-activa #captura-pasiva #top-talkers
