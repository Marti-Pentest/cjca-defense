# Módulo 16 — Intro to Network Traffic Analysis

## Sección 12/15: Uso Avanzado de Wireshark

## 📌 Pestañas de plugins: Estadísticas y Analizar

> [!NOTE]
> **Pestaña Estadísticas (Statistics)**
> Ofrece perspectiva general del tráfico capturado: **top talkers**, conversaciones específicas, desglose por IP y protocolo, jerarquía de protocolos.

> [!NOTE]
> **Pestaña Analizar (Analyze)**
> Permite: seguir flujos TCP, filtrar por tipo de conversación, preparar nuevos filtros de paquetes, examinar **información experta** (expert info) que Wireshark genera sobre el tráfico.

## 🔗 Seguir Flujos TCP (Follow TCP Stream)

> [!TIP]
> **¿Qué hace?**
> Une los paquetes TCP para reconstruir el flujo completo en formato legible. Permite extraer datos (imágenes, archivos, etc.) de la captura. Funciona para casi cualquier protocolo sobre TCP.

### Cómo usarlo (GUI)
```
1. Clic derecho en un paquete del flujo deseado
2. Seleccionar: Seguir → TCP
3. Se abre una ventana con el flujo reconstruido completo
```

### Alternativa por filtro
```
tcp.stream eq #
```
> [!NOTE]
> **Comportamiento esperado**
> Al filtrar por un stream específico, se ve primero el **handshake TCP completo** (3 paquetes) y luego la transferencia de datos — todo lo irrelevante queda oculto.

## 📂 Extraer Datos y Archivos de una Captura

> [!WARNING]
> **Requisito crítico**
> Necesitas haber capturado **la conversación completa**. Si falta algún fragmento, la reconstrucción del datagrama fallará (relacionado con fragmentación TCP/IP).

### Pasos (GUI)
```
1. Detener la captura
2. Archivo → Exportar → seleccionar formato de protocolo
   (DICOM, HTTP, SMB, etc.)
```

## 📁 Extracción específica vía FTP

> [!NOTE]
> **Recordatorio de puertos FTP**
> - **Puerto 20 (TCP)**: transferencia de datos
> - **Puerto 21 (TCP)**: canal de control (login, listar archivos, comandos)

### Filtros de visualización FTP clave

| Filtro | Uso |
|---|---|
| `ftp` | Muestra todo el protocolo FTP — identifica qué hosts/servidores transfieren datos |
| `ftp.request.command` | Muestra comandos enviados por el canal de control (puerto 21) — útil para ver **usuarios/contraseñas** y nombres de archivo solicitados |
| `ftp-data` | Muestra los datos transferidos por el canal de datos (puerto 20) — permite reconstruir el archivo transferido |

### Flujo de trabajo completo para extraer un archivo FTP de un PCAP

```
1. Identificar tráfico FTP:
   ftp

2. Revisar comandos control para ver qué se transfirió y quién:
   ftp.request.command

3. Elegir un archivo de interés, filtrar por:
   ftp-data

4. Seleccionar el paquete correspondiente → Seguir flujo TCP

5. Cambiar "Show and save data as" a "Raw"
   → Guardar con el nombre del archivo original

6. Validar la extracción comprobando el tipo de archivo
```

> [!TIP]
> **Ejemplo real del módulo**
> Se mostró extracción de archivos llamados `secrets.txt` y `Shield-prototype-plans` desde una conversación FTP — ejemplo de cómo un análisis de tráfico puede revelar exfiltración de datos sensibles.

## 🧠 Quiz de repaso (Q&A del módulo)

<details>
<summary>¿Qué pestaña de plugin da metadata de conversaciones y desglose de protocolos de todo el PCAP?</summary>

**Estadísticas** (Statistics)

</details>


<details>
<summary>¿Qué pestaña de plugin permite aplicar filtros, seguir flujos y ver info experta?</summary>

**Analizar** (Analyze)

</details>


<details>
<summary>¿Qué protocolo de transporte orientado a flujo permite seguir y reconstruir conversaciones con sus datos?</summary>

**TCP**

</details>


<details>
<summary>¿Verdadero o Falso: Wireshark puede extraer archivos de tráfico HTTP?</summary>

**Verdadero**

</details>


<details>
<summary>¿Verdadero o Falso: el filtro `ftp-data` muestra datos enviados por el puerto TCP 21?</summary>

**Falso** — `ftp-data` corresponde al **puerto 20** (canal de datos). El puerto 21 es el canal de control (`ftp.request.command`)

</details>


## 🔗 Relacionado
- [11-wireshark-familiarity-lab](11-wireshark-familiarity-lab.md)
- [13-wireshark-packet-inception-lab](13-wireshark-packet-inception-lab.md)
- *SMB - Notas*
- *FTP - Notas Seccion 3*

#cjca #modulo16 #wireshark #follow-tcp-stream #ftp #extraccion-archivos #exfiltracion
