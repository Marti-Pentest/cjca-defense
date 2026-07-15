# Módulo 16 — Intro to Network Traffic Analysis

## Sección 14/15: Guided Lab — Traffic Analysis Workflow

> [!warning] Nota sobre este resumen
> Este lab requiere spawnear un target y usar `guided-analysis.zip` con su walkthrough incluido. No se incluyen las respuestas concretas (nombre de usuario creado, número de paquetes, puerto sospechoso) — se documenta la **metodología completa del workflow** aplicado a un caso real de intrusión.

## 🎯 Escenario

> [!note] Caso
> Un admin detectó una conexión rara desde el host de **Bob** al revisar las capturas de baseline. Se pide investigar qué está pasando.

## 🔌 Conexión al entorno

```bash
xfreerdp /v:<target IP> /u:htb-student /p:HTB_@cademy_stdnt!
```
- Usuario: `htb-student`
- Contraseña: `HTB_@cademy_stdnt!`
- Interfaz de captura en Wireshark: **ENS224**

## 🧠 Plantilla de Workflow de Análisis (aplicación práctica del framework de la Sección 5)

Este lab es la **aplicación directa** del flujo de 4 fases visto en la Sección 5 (Descriptivo → Diagnóstico → Predictivo → Prescriptivo):

### 1️⃣ ¿Cuál es el problema? (Descriptivo)
```
- Resumen breve del problema: conexión sospechosa desde el host de Bob
- Alcance y objetivo: ¿qué buscamos? ¿en qué periodo?
- Cuándo empezó el problema
- Info de apoyo: archivos, fuentes de datos disponibles
```

### 2️⃣ Definir objetivo(s) — red/host(s)/protocolo
```
- Hosts objetivo: host de Bob + cualquier host con el que se comunique
```

### 3️⃣ Capturar tráfico / leer PCAP previo
- Capturar en vivo desde ENS224, o
- Analizar `guided-analysis.pcap` ya provisto

### 4️⃣ Identificar componentes necesarios (filtrado)
```
Filtrar todo lo que coincida con el baseline común
Conservar solo lo relevante al alcance de la investigación
```

### 5️⃣ Comprensión del tráfico capturado
> [!tip] Estrategia: empezar amplio, cerrar el círculo
> Similar a la Sección 5: primero protocolos estándar → luego host-a-host inusual → luego patrones/puertos raros

### 6️⃣ Toma de notas / mapas mentales
```
- Periodos de captura
- Hosts/puertos sospechosos
- Conversaciones sospechosas (timestamps, número de paquete, archivos)
```

### 7️⃣ Resumen del análisis
Explicar hallazgos con claridad para que los superiores decidan cuarentena o IR más profundo.

## 🚨 Resumen del incidente (revelado al final del lab)

> [!warning] Qué se descubrió
> El actor de amenaza:
> 1. Usó una **shell de Netcat** para interactuar directamente con el host de Bob mientras recolectaba información
> 2. Desde el host de Bob, usó **RDP** para moverse lateralmente hacia otro escritorio Windows del entorno, intentando establecer otro punto de apoyo (foothold)
> 3. El equipo de IR capturó tráfico RDP de ese movimiento lateral
> 4. **Acción tomada**: el host de Bob fue puesto en **cuarentena** e iniciada una respuesta a incidentes formal

> [!tip] Patrón de ataque a reconocer
> Esto ilustra el ciclo típico: **acceso inicial → shell C2 (Netcat) → recolección de info → movimiento lateral (RDP) → nuevo foothold**. Es un ejemplo perfecto de por qué "host de usuario hablando con otro host de usuario" (visto en la Sección 4) es tan sospechoso — el movimiento lateral vía RDP entre estaciones de trabajo no es tráfico normal.

## 🔍 Pistas de qué buscar en Wireshark (basado en el resumen)

| Indicador | Cómo detectarlo |
|---|---|
| **Shell Netcat** | Buscar conexiones TCP en puertos no estándar/altos con tráfico de texto plano tipo comandos de shell (`follow tcp stream`) |
| **Movimiento lateral RDP** | Filtrar `tcp.port == 3389` o `rdp` — ver conexión saliente desde el host de Bob hacia otro host interno |
| **Nuevo usuario creado** | Si el RDP no está cifrado o se puede decodificar, buscar comandos tipo `net user` en el flujo reconstruido |

## 🧠 Preguntas del lab (requieren el PCAP real — checklist para resolverlas)

> [!faq]- ¿Cómo determinarías el nombre del nuevo usuario creado en el host afectado?
> Seguir el flujo TCP de la sesión Netcat/RDP y buscar comandos tipo `net user <nombre> <pass> /add` en el contenido reconstruido.

> [!faq]- ¿Cómo determinarías el total de paquetes del PCAP?
> Wireshark: barra de estado inferior, o `Statistics → Capture File Properties`

> [!faq]- ¿Cómo identificarías el puerto sospechoso usado?
> Buscar conexiones en puertos altos/no estándar con actividad interactiva (patrón de shell) — comparar contra el baseline esperado del segmento.

## 🔗 Relacionado
- [[13-wireshark-packet-inception-lab]]
- [[15-decrypting-rdp-connections]]
- [[04-analysis-process]]
- [[Netcat Shells - Deteccion]]
- [[Movimiento Lateral RDP - Notas]]

#cjca #modulo16 #wireshark #laboratorio #workflow-analisis #netcat #rdp #movimiento-lateral #incident-response
