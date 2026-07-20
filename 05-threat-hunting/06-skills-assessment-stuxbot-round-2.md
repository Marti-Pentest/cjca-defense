# Módulo 20 — Introduction to Threat Hunting & Hunting With Elastic

## Sección 6/6 (Final): Skills Assessment — Hunting For Stuxbot (Round 2)

## 🎯 Contexto: nueva iteración de Stuxbot

> [!WARNING]
> **Inteligencia actualizada sobre TTPs**
> 1. Usa **`C:\Users\Public`** como directorio de tránsito para desplegar utilidades adicionales
> 2. Usa **registry run keys** como mecanismo de persistencia
> 3. Usa **PowerShell Remoting** para movimiento lateral y acceso a domain controllers

> [!TIP]
> **Por qué esto es un "Round 2"**
> Es la aplicación práctica de todo lo visto en la Sección 5 (metodología de hunting) pero contra **nuevas TTPs** — sin el paso a paso guiado esta vez. Este assessment evalúa si se pueden **construir las queries KQL** a partir de un framework de referencia (MITRE ATT&CK), no solo replicar queries ya dadas.

## 📊 Fuentes de datos disponibles (recordatorio)

| Fuente | Index pattern |
|---|---|
| Windows audit logs | `windows*` |
| Sysmon logs | `windows*` |
| PowerShell logs | `windows*` |
| Zeek logs | `zeek*` |

## 🔌 Configuración inicial

```
1. http://[Target IP]:5601 -> Discover
2. Calendario -> "last 15 years" -> Apply
```

## 🎯 Hunt 1: Lateral Tool Transfer hacia C:\Users\Public

> [!NOTE]
> **Referencia MITRE**
> [T1570 — Lateral Tool Transfer](https://attack.mitre.org/techniques/T1570/)

### Metodología para construir la query

> [!TIP]
> **Enfoque (aplicando lo aprendido en la Sección 5)**
> Un "tool transfer" hacia `C:\Users\Public` se manifiesta típicamente como un evento de **creación de archivo** (Sysmon Event ID 11 — FileCreate) donde el `file.path` apunta a esa carpeta.

```
event.code:11 AND file.path:"C:\Users\Public\*"
```

> [!TIP]
> **Alternativa si se sospecha transferencia vía SMB**
> Si la herramienta llegó vía copia de red (ej: PsExec, SMB admin shares), también vale la pena revisar:
> ```
> event.code:11 AND file.path:C\:\\Users\\Public\\*
> ```
> o cruzar con Sysmon Event ID 15 (FileCreateStreamHash) si el archivo vino de descarga vía navegador/red.

> [!WARNING]
> **Pregunta del lab**
> Identificar el `user.name` asociado al documento donde la herramienta transferida **empieza con "r"** (pista: nombre de herramienta común de post-explotación/recon).

## 🎯 Hunt 2: Registry Run Keys / Startup Folder

> [!NOTE]
> **Referencia MITRE**
> [T1547.001 — Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder](https://attack.mitre.org/techniques/T1547/001/)

### Metodología para construir la query

> [!TIP]
> **Event ID relevante**
> **Sysmon Event ID 13** (RegistryEvent - Value Set) es el más directo para detectar escritura de valores en el registro asociados a persistencia.

```
event.code:13 AND registry.path:*CurrentVersion\\Run*
```

> [!NOTE]
> **Rutas de registro clásicas de persistencia (Run Keys)**
> - `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
> - `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`
> - Variantes `RunOnce`

> [!WARNING]
> **Pregunta del lab**
> Identificar el contenido del campo `registry.value` en el documento relacionado a la **primera acción de persistencia basada en registro**. Esto implica ordenar los resultados por timestamp ascendente para encontrar el evento más temprano.

## 🎯 Hunt 3: PowerShell Remoting para movimiento lateral (hacia DC1)

> [!NOTE]
> **Referencia técnica**
> [T1028 — WinRM for Lateral Movement (ired.team)](https://www.ired.team/offensive-security/lateral-movement/t1028-winrm-for-lateral-movement)

### Metodología para construir la query

> [!TIP]
> **Indicadores de PowerShell Remoting (WinRM)**
> - **Logon Type 3** (Network) o específicamente tipo asociado a WinRM en el log de Security
> - Proceso `wsmprovhost.exe` (WinRM Provider Host) apareciendo como proceso nuevo en el destino — es el proceso que WinRM lanza del lado del servidor cuando recibe una sesión remota
> - Conexión al puerto **5985** (HTTP) o **5986** (HTTPS) — puertos estándar de WinRM

```
event.code:1 AND process.name:"wsmprovhost.exe" AND host.hostname:"DC1"
```

> [!TIP]
> **Complementar con logs de logon**
> ```
> event.code:4624 AND host.hostname:"DC1" AND winlog.event_data.LogonProcessName:"NtLmSsp*"
> ```
> o filtrar por `winlog.channel` relacionado a WinRM operational logs si están disponibles como fuente separada.

> [!WARNING]
> **Pregunta del lab**
> Identificar el `winlog.user.name` asociado al documento de movimiento lateral vía PowerShell Remoting **específicamente dirigido a DC1**.

## 🧠 Framework general para resolver estos 3 hunts (checklist reutilizable)

> [!NOTE]
> **Pasos aplicables a cualquier hunt basado en MITRE ATT&CK**
1. **Leer la técnica en MITRE ATT&CK** — identificar qué artefactos/comportamientos deja (proceso, archivo, registro, red)
2. **Mapear el artefacto a un Sysmon Event ID específico** (Process Creation=1, FileCreate=11, RegistryEvent=13, Network=3, etc.)
3. **Construir la query KQL base** con ese Event ID + el campo/valor característico de la técnica
4. **Acotar por host/usuario/timeframe** según lo que pida la pregunta específica
5. **Ordenar cronológicamente** si se pide "el primero" o "el más temprano"
6. **Expandir el documento** para extraer el campo exacto solicitado (`user.name`, `registry.value`, `winlog.user.name`, etc.)

## 🖥️ Nota sobre las respuestas

> [!WARNING]
> **Sin respuestas exactas**
> No se incluyen los valores específicos de `user.name`, `registry.value`, o `winlog.user.name` — son datos del dataset real del spawn. La metodología de construcción de cada query KQL (arriba) es la parte transferible y reutilizable de este ejercicio; aplicarla contra el entorno real del lab debería llevar directamente a los documentos correctos.

## 🏁 Cierre del módulo

Con esta sección se completa **Introduction to Threat Hunting & Hunting With Elastic** (6/6). Recorrido completo:
1. Fundamentos de Threat Hunting (dwell time, relación con IH, estructura de equipo, cuándo cazar, risk assessment)
2. El proceso de Threat Hunting (8 etapas + caso práctico Emotet)
3. Glosario de Threat Hunting (Adversary, APT, TTPs, Pyramid of Pain, Diamond Model)
4. *(sección pendiente de notas)*
5. Hunting for Stuxbot — caso completo end-to-end con Elastic Stack
6. Skills Assessment — Stuxbot Round 2, construcción autónoma de queries basadas en MITRE ATT&CK

## 🔗 Relacionado
- [Hunting For Stuxbot](05-hunting-for-stuxbot.md)
- [Threat Hunting Glossary](03-threat-hunting-glossary.md)
- [Modulo 19 - SIEM Fundamentals](../04-security-monitoring-siem/01-siem-definition-fundamentals.md)
- *MITRE ATT&CK T1570, T1547.001, T1028*

#cjca #modulo20 #skills-assessment #stuxbot #mitre-attack #lateral-movement #registry-persistence #powershell-remoting #modulo-completo
