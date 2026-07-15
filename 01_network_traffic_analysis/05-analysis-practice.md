# Módulo 16 — Intro to Network Traffic Analysis

## Sección 5/15: Análisis en la práctica

## 📌 Los 4 tipos de análisis (framework completo)

> [!NOTE]
> **Idea central**
> El flujo de análisis no es un bucle lineal exacto — es dinámico según el objetivo (errores de red vs. malicioso) y la visibilidad disponible. Pero se puede resumir en 4 fases secuenciales.

### 1️⃣ Análisis Descriptivo
> [!TIP]
> **Define el problema**
> Describe el dataset, detecta errores/outliers en la recolección. Responde:
> - ¿Sospecha de brecha o problema de red?
> - **Qué buscamos** y **en qué periodo**
> - **Alcance**: red / host(s) / protocolo

**Ejemplo del módulo:**
```
Objetivo: múltiples hosts descargando archivo malicioso desde bad.example.com
Cuándo: últimas 48h + 2h desde ahora
Archivos: 'superbad.exe', 'new-crypto-miner.exe'
Alcance: red 192.168.100.0/24, protocolos HTTP y FTP
```

### 2️⃣ Análisis de Diagnóstico
> [!TIP]
> **Encuentra las causas**
> Visión retrospectiva — busca **razones** de eventos y desarrollos (correlación + interpretación).

**Pasos:**
1. **Capturar tráfico** → conectarse al segmento en cuestión (live capture) o extraer PCAP/NetFlow histórico del SIEM
2. **Filtrar componentes necesarios** → eliminar tráfico que coincide con el baseline, quedarse solo con lo relevante al alcance
3. **Comprender el tráfico capturado** → usar filtros específicos:

```
# Reconstruir archivos transferidos por FTP
ftp-data

# Filtrar solicitudes GET de HTTP (Wireshark display filter)
http.request.method == "GET"
```

### 3️⃣ Análisis Predictivo
> [!TIP]
> **Modela probabilidades futuras**
> Basado en lo descriptivo + diagnóstico. Identifica tendencias, detecta desviaciones tempranas, compara contra baseline y firmas maliciosas conocidas.

**Componentes:**
- [ ] Tomar notas y mapas mentales de hallazgos:
  - Periodos de captura
  - Hosts sospechosos
  - Conversaciones con los archivos en cuestión (timestamps + número de paquete)
- [ ] Resumen del análisis (qué se encontró) → claro y conciso para que superiores decidan cuarentena/IR

### 4️⃣ Análisis Prescriptivo
> [!TIP]
> **La solución**
> Acota **acciones a tomar** para eliminar/prevenir el problema. Culminación del flujo.

> [!WARNING]
> **No olvidar las lecciones aprendidas**
> Documentar: qué se hizo bien, qué no ayudó, qué mejorar. Esto fortalece procesos futuros.

> [!NOTE]
> **Es cíclico, no lineal**
> Si se necesita IR a gran escala, puede requerir **volver a analizar el PCAP original** examinando otras conversaciones de los hosts afectados (ej: verificar propagación por otra ruta).

## 🎯 Componentes clave de un análisis eficaz

| Componente | Por qué importa |
|---|---|
| **1. Conocer el entorno** | Sin inventario de activos/mapas de red, no se puede distinguir un host legítimo de un intruso |
| **2. Ubicación** | Capturar lo más cerca posible del origen del problema (enlaces de entrada si viene de internet; mismo segmento si es interno) |
| **3. Persistencia** | Un C2 puede comunicarse solo 1 vez cada varias horas/día — no perder el impulso de investigar |

## 🔍 Enfoque práctico — "victorias fáciles"

> [!NOTE]
> **Orden de revisión sugerido**
> 1. **Protocolos estándar primero**: HTTP/S, FTP, email, TCP/UDP básico (la mayoría de ataques entran desde internet)
> 2. **Protocolos de acceso remoto**: SSH, RDP, Telnet — comparar contra la política de seguridad (¿se permite RDP desde fuera?)
> 3. **Buscar patrones**: ¿mismo host hablando con internet a la misma hora todos los días? → perfil típico de **C2 beaconing**
> 4. **Tráfico host-a-host interno**: sospechoso por definición — normalmente los hosts de usuario hablan con infraestructura (DNS, DHCP, servicios), no entre sí
> 5. **Eventos únicos**: cambios de patrón (host que visita un sitio 10x/día y de repente 1x), User-Agent inusual, puertos aleatorios que se abren una sola vez → posibles callbacks de C2

> [!TIP]
> **Pedir ayuda**
> Después de mirar capturas por mucho tiempo, es fácil perder la perspectiva. Un segundo par de ojos ayuda a detectar lo que se pasa por alto.

> [!NOTE]
> **No depender solo de tcpdump/Wireshark**
> Complementar con **Snort, Security Onion, Firewalls, SIEMs** para enriquecer el análisis.

## 🔗 Relacionado
- [04-analysis-process](04-analysis-process.md)
- [06-tcpdump-fundamentals](06-tcpdump-fundamentals.md)
- *Wireshark Filtros de Visualizacion*

#cjca #modulo16 #analisis-descriptivo #analisis-diagnostico #analisis-predictivo #analisis-prescriptivo #c2-beaconing
