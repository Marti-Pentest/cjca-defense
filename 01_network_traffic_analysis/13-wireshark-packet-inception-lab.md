# Módulo 16 — Intro to Network Traffic Analysis

## Sección 13/15: Incepción de Paquetes — Diseccionando Tráfico con Wireshark (Lab)

> [!WARNING]
> **Nota sobre este resumen**
> Este lab requiere spawnear un target específico y usar `Wireshark-lab-2.pcap`. No se incluyen respuestas exactas del lab (nombre de archivo, empleado sospechoso) — se documenta toda la **metodología** para replicarla.

## 🎯 Escenario — Parte 1: Análisis forense de PCAP pre-capturado

> [!NOTE]
> **Caso**
> Un pcap contiene una sesión web **no cifrada** con una imagen incrustada. Sospecha: el usuario esconde mensajes ocultos detrás de la imagen (posible **esteganografía** o exfiltración de datos).

### Flujo de trabajo — Extracción HTTP

```
1. File → Open → Wireshark-lab-2.pcap
   (~1171 paquetes totales, <20 son HTTP)

2. Filtrar solo tráfico HTTP:
   http

3. Buscar líneas con "200 OK" (respuesta a un GET)
   → Clic derecho → Follow → TCP Stream

4. Verificar presencia de imagen JPEG buscando formato JFIF:
   Puedes filtrar por:
   http contains "JFIF"

5. Extraer los objetos:
   File → Export Objects → HTTP...
   → Seleccionar el archivo de imagen → Save
```

> [!TIP]
> **¿Por qué buscar JFIF?**
> **JFIF** (JPEG File Interchange Format) es el estándar de facto para archivos JPEG. Buscarlo en el tráfico HTTP confirma que se transfirió una imagen, antes de gastar tiempo en exportar objetos.

> [!WARNING]
> **Sobre la "esteganografía" sospechada**
> Una vez extraída la imagen, el análisis de datos ocultos dentro de ella (esteganografía) queda **fuera del alcance de Wireshark** — se necesitarían herramientas como `steghide`, `binwalk`, `exiftool`, o análisis manual de los bytes del archivo.

## 🎯 Escenario — Parte 2: Captura en vivo (RDP a VM de laboratorio)

### Conexión al lab
```bash
xfreerdp /v:<target IP> /u:htb-student /p:HTB_@cademy_stdnt!
```
- Usuario: `htb-student`
- Contraseña: `HTB_@cademy_stdnt!`
- IP: se obtiene al spawnear el target en la plataforma

### Captura
```
Interfaz a usar: ENS224
Dejar correr la captura unos minutos antes de detener
Hosts de interés: 172.16.10.2 y 172.16.10.20
```

## 🧠 Checklist de autoanálisis (aplicable a cualquier captura en vivo)

> [!NOTE]
> **Preguntas guía antes de seguir pasos dirigidos**
- [ ] ¿Cuántas conversaciones se pueden ver?
- [ ] ¿Quiénes son clientes y servidores?
- [ ] ¿Qué protocolos se usan?
- [ ] ¿Algo digno de mención? (puertos mal usados, texto plano, credenciales)

## 📁 Análisis FTP — Metodología

```
1. Filtrar tráfico FTP:
   ftp

2. Identificar servidor (quien responde en puerto 21) y tipo de autenticación:
   ftp.request.command
   → Buscar comandos USER/PASS para ver si es anónimo o autenticado
   → "USER anonymous" indica sesión anónima

3. Examinar qué archivo(s) se transfirieron (RETR, STOR, etc.)

4. Extraer y reensamblar vía ftp-data:
   ftp-data
   → Seleccionar paquete de interés → Follow TCP Stream
   → "Show and save data as" → Raw → Guardar con nombre/extensión original
```

## 🌐 Análisis HTTP — Metodología

```
1. Filtrar tráfico HTTP:
   http

2. Identificar servidor web (IP que responde 200 OK)

3. Determinar aplicación del servidor:
   → Revisar header "Server:" en la respuesta (ver en hex/ASCII o en el panel de detalles)

4. Ver métodos más comunes:
   → Observar columna Info: GET, POST, etc.

5. Seguir flujo y extraer:
   Follow → TCP Stream
   File → Export Objects → HTTP
```

## 🧠 Preguntas de autoevaluación (para tu propia práctica)

<details>
<summary>¿Qué filtros usaste y fueron efectivos?</summary>

Reflexión personal — documentar en tu vault qué combinaciones (`http`, `ftp`, `tcp.stream eq #`, `ip.addr ==`) te dieron mejores resultados.

</details>


<details>
<summary>¿Qué filtro verías para tráfico TCP solo del lado del cliente?</summary>

```
ip.src == <IP_del_cliente> and tcp
```

</details>


## 🔗 Relacionado
- [12-wireshark-advanced-usage](12-wireshark-advanced-usage.md)
- [14-guided-lab-traffic-analysis-workflow](14-guided-lab-traffic-analysis-workflow.md)
- *FTP - Notas Seccion 3*
- *Esteganografia - Herramientas Externas*

#cjca #modulo16 #wireshark #laboratorio #ftp #http #export-objects #forense #rdp
