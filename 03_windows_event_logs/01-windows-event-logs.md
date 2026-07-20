# Módulo 18 — Windows Event Logs & Finding Evil

## Sección 1/6: Windows Event Logs

## 📌 Fundamentos de Windows Event Logging

> [!NOTE]
> **¿Qué son los Windows Event Logs?**
> Parte intrínseca del sistema operativo Windows. Almacenan logs de distintos componentes: el sistema mismo, aplicaciones, proveedores ETW, servicios, etc. Se acceden vía **Event Viewer** o programáticamente con la **Windows Event Log API**.

### Logs por defecto de Windows

| Log | Contenido |
|---|---|
| **Application** | Errores de aplicaciones |
| **Security** | Eventos de seguridad (logons, cambios de auditoría, etc.) |
| **Setup** | Actividades de instalación/configuración del sistema |
| **System** | Información general del sistema |
| **Forwarded Events** | Logs reenviados desde **otras máquinas** — útil para vista centralizada |

> [!TIP]
> **Archivos .evtx**
> Event Viewer puede abrir archivos `.evtx` previamente guardados en la sección **"Saved Logs"** — clave para análisis forense offline de capturas de logs.

## 🧬 Anatomía de un Event Log

> [!NOTE]
> **Niveles de severidad en Application logs**
> - **Information**: eventos generales de uso (inicio/parada de la app)
> - **Error**: errores específicos con detalles del problema

### Componentes principales de un Event

| Campo | Descripción |
|---|---|
| **Log Name** | Nombre del log (Application, System, Security, etc.) |
| **Source** | Software que generó el evento |
| **Event ID** | Identificador único del evento |
| **Task Category** | Ayuda a entender el propósito/uso del evento |
| **Level** | Severidad (Information, Warning, Error, Critical, Verbose) |
| **Keywords** | Flags para categorizar más allá de otras clasificaciones (ej: "Audit Success"/"Audit Failure") |
| **User** | Cuenta de usuario logueada cuando ocurrió el evento |
| **OpCode** | Identifica la operación específica reportada |
| **Logged** | Fecha/hora del registro |
| **Computer** | Nombre del equipo donde ocurrió |
| **XML Data** | Toda la info anterior + datos adicionales en formato XML |

> [!TIP]
> **Campo Keywords es clave para filtrado**
> Permite precisión al buscar tipos específicos de eventos — mejora mucho la eficiencia del log management.

> [!NOTE]
> **Investigación de un Event ID real: caso SideBySide (Event 35)**
> Ejemplo de error por mismatch de arquitectura de procesador al generar un contexto de activación para Visual Studio. Se puede extraer el **Process ID** del evento para análisis más preciso.

## 🔐 Análisis de Security Logs — Caso práctico: Event ID 4624

> [!NOTE]
> **Event ID 4624 — Successful Logon**
> Según la [documentación de Microsoft](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624): indica la creación de una sesión de logon en la máquina destino.

**Campos clave:**
- **Logon ID**: permite correlacionar este logon con otros eventos que compartan el mismo ID
- **Logon Type**: tipo de logon (ej: Type 5 = **Service logon**, sugiere que SYSTEM inició un nuevo servicio)

> [!WARNING]
> **Requiere correlación adicional**
> Un Logon Type 5 por sí solo no dice *qué* servicio se inició — hay que correlacionar con otros datos usando el **Logon ID**.

## 🛠️ XML Queries personalizadas

> [!TIP]
> **Cómo crear un query XML manual**
> ```
> Filter Current Log → XML → Edit Query Manually
> ```
> Permite lenguaje de consulta granular. Ejemplo del módulo: filtrar por `SubjectLogonId = 0x3E7` para correlacionar todos los eventos asociados a ese Logon ID específico.

> [!NOTE]
> **Beneficio práctico**
> Permite enfocarse en la cuenta responsable de iniciar un servicio y eliminar ruido — aunque incluso refinado, el volumen de datos puede seguir siendo alto.

## 🕵️ Caso de investigación completo — Narrativa del módulo

> [!NOTE]
> **Secuencia de eventos reconstruida**
> 1. **Event ID 4907** (Audit Policy Change) — el SACL de un objeto (registro/archivo) fue modificado
> 2. Proceso responsable: `SetupHost.exe` (⚠️ nombre legítimo — el malware puede enmascararse con nombres así)
> 3. Objeto afectado: el **bootmanager**
> 4. Se pueden comparar `NewSd` y `OldSd` (Security Descriptors) para ver los cambios exactos
> 5. Luego: **Event 4624** (Logon) seguido de **Event 4672** (Special Logon)
> 6. El Special Logon revela privilegios de token otorgados (ej: `SeAssignPrimaryTokenPrivilege`, `SeDebugPrivilege`)

> [!WARNING]
> **SACL — System Access Control List**
> La "S" de SACL = System. Permite a administradores **loguear intentos de acceso** a objetos seguros. Cada ACE (Access Control Entry) dentro de un SACL especifica qué tipos de intentos de acceso (fallidos, exitosos, o ambos) generan un registro de auditoría.
>
> Referencia: [Access Control Lists (Microsoft)](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-control-lists)

> [!WARNING]
> **SeDebugPrivilege**
> Indica que el usuario puede **manipular memoria que no le pertenece** — privilegio de alto riesgo si es abusado.

## 📚 Lista de Event IDs útiles para threat hunting

### Windows System Logs

| Event ID | Nombre | Por qué importa |
|---|---|---|
| **1074** | System Shutdown/Restart | Apagones/reinicios inesperados → posible malware o acceso no autorizado |
| **6005** | Event log service started | Marca boot-up del sistema — punto de partida para investigar |
| **6006** | Event log service stopped | Normal en shutdown; anómalo si es inesperado → posible ocultamiento de actividad |
| **6013** | Windows uptime | Uptime menor al esperado → posible reinicio por intrusión |
| **7040** | Service status change | Cambio de tipo de inicio (manual↔automático) → posible tampering |

### Windows Security Logs

| Event ID | Nombre | Por qué importa |
|---|---|---|
| **1102** | Audit log cleared | ⚠️ Señal fuerte de intento de **borrar evidencia** |
| **1116** | Antivirus malware detection | Aumento repentino → posible ataque dirigido o infección masiva |
| **1118/1119/1120** | AV remediation started/succeeded/failed | Monitorear especialmente los **1120 (fallidos)** — requieren atención inmediata |
| **4624** | Successful Logon | Establece comportamiento normal — logons a horas raras o ubicaciones distintas = alerta |
| **4625** | Failed Logon | Múltiples fallos = posible **brute-force** |
| **4648** | Logon con credenciales explícitas | Anomalías aquí → posible **movimiento lateral** |
| **4656** | Handle a un objeto solicitado | Detecta intentos de acceso a recursos sensibles |
| **4672** | Special Privileges Assigned | Rastrea uso de privilegios de superusuario |
| **4698** | Scheduled task creado | Mecanismo común de **persistencia** |
| **4700/4701** | Scheduled task enabled/disabled | Manipulación de tareas para persistencia |
| **4702** | Scheduled task actualizado | Cambios que podrían indicar intención maliciosa |
| **4719** | System audit policy changed | Posible intento de **cubrir huellas** desactivando auditoría |
| **4738** | User account changed | Cambios inesperados → posible account takeover o insider threat |
| **4771** | Kerberos pre-auth failed | Equivalente a 4625 pero para Kerberos → posible brute-force |
| **4776** | DC validó credenciales | Múltiples fallos → posible brute-force |
| **5001** | AV real-time protection config changed | Cambios no autorizados → intento de deshabilitar Defender |
| **5140** | Network share accedido | Acceso no autorizado a shares de red |
| **5142** | Network share creado | Shares no autorizados → posible exfiltración o propagación de malware |
| **5145** | Chequeo de acceso a network share | Chequeos frecuentes → posible mapeo de red (recon) |
| **5157** | WFP bloqueó una conexión | Útil para identificar tráfico malicioso |
| **7045** | Servicio instalado en el sistema | Aparición repentina de servicios desconocidos → posible instalación de malware |

> [!WARNING]
> **Clave del threat detection: conocer el "normal" del entorno**
> Lo que es anómalo en un entorno puede ser normal en otro. Es fundamental:
> - Ajustar (tune) monitoreo/alertas al entorno específico para minimizar falsos positivos
> - Tener **log management centralizado** que recolecte, parsee y alerte en tiempo real
> - **Correlacionar** estos logs con otros logs de sistema/seguridad para una vista holística

## 🖥️ Lab práctico

```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:[Target IP] /dynamic-resolution
```

> [!NOTE]
> **Tareas del lab**
> 1. Analizar el evento ID 4624 del 8/3/2022 10:23:25 → identificar el ejecutable responsable de modificar la configuración de auditoría (formato `T_W_____.exe`)
> 2. Construir un XML query para determinar si ese ejecutable modificó la auditoría de `wpfgfx_v0400.dll` → obtener el timestamp exacto del evento

> [!WARNING]
> **Nota sobre las respuestas**
> No se incluyen las respuestas exactas del lab (esto requiere investigación propia en el entorno spawneado) — pero la **metodología completa está arriba**: seguir la cadena 4907 → identificar proceso → correlacionar con 4624/4672 → construir el XML query filtrando por `SubjectLogonId` u `ObjectName` según corresponda.

## 🔗 Relacionado
- [Analyzing Evil With Sysmon & Event Logs](02-analyzing-evil-sysmon-event-logs.md)
- *SACL y ACL - Notas*
- *Event ID Cheatsheet*

#cjca #modulo18 #windows-event-logs #event-viewer #sacl #threat-hunting #xml-queries
