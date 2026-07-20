# Módulo 18 — Windows Event Logs & Finding Evil

## Sección 2/6: Análisis de Actividades Maliciosas con Sysmon y Registros de Eventos

## 📌 Fundamentos de Sysmon

> [!NOTE]
> **¿Qué es Sysmon?**
> **System Monitor** — servicio de Windows + controlador de dispositivo que **permanece residente a través de reinicios**, monitoreando y registrando actividad del sistema en el Event Log de Windows.

**Componentes principales:**
- Servicio de Windows para monitorear actividad
- Controlador de dispositivo que captura los datos
- Registro de eventos para mostrar la actividad capturada

> [!TIP]
> **Ventaja única de Sysmon**
> Registra información que **normalmente NO aparece** en los logs de Security estándar — herramienta clave para monitoreo profundo y análisis forense.

### Event IDs clave de Sysmon (relevantes para esta sección)

| Event ID | Nombre | Uso |
|---|---|---|
| **1** | Process Creation | Creación de procesos |
| **3** | Network Connection | Conexiones de red |
| **7** | Image Loaded | Carga de módulos/DLLs |
| **10** | ProcessAccess | Acceso entre procesos (ej: acceso a LSASS) |

> [!NOTE]
> **IDs estándar de Windows Security también relevantes**
> - **Event ID 4624**: nuevo logon exitoso
> - **Event ID 4688**: proceso recién creado

## 🛠️ Instalación y configuración de Sysmon

```cmd
:: Instalación básica con hashes MD5, SHA256, imphash + logging de red
C:\Tools\Sysmon> sysmon.exe -i -accepteula -h md5,sha256,imphash -l -n

:: Aplicar configuración XML personalizada
C:\Tools\Sysmon> sysmon.exe -c filename.xml
```

> [!TIP]
> **Configuraciones populares (community configs)**
> - [SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config) — configuración completa (usada en este módulo)
> - [olafhartong/sysmon-modular](https://github.com/olafhartong/sysmon-modular) — enfoque modular

> [!NOTE]
> Sysmon también existe para Linux.

## 🎯 Detección 1: DLL Hijacking

### Preparar Sysmon para capturar carga de módulos

> [!WARNING]
> **Ajuste clave en el XML: include → exclude**
> Por defecto la config viene con `include` en el RuleGroup de `ImageLoad` (sin reglas = no se registra nada). Cambiar a **`exclude`** (sin reglas dentro) asegura que **todo** se capture, sin excluir nada.

```cmd
C:\Tools\Sysmon> sysmon.exe -c sysmonconfig-export.xml
```

**Ruta en Event Viewer:**
```
Applications and Services Logs → Microsoft → Windows → Sysmon → Operational
```

> [!NOTE]
> **Estructura de un Event ID 7 normal**
> ```
> ProcessID: 8060
> Image: mmc.exe
> ImageLoaded: psapi.dll
> Signed: true
> User: DESKTOP-N33HELB\Waldo
> ```
> Contiene: estado de firma de la DLL, proceso que la cargó, y la DLL específica cargada.

### Reproducción del ataque

> [!NOTE]
> **Caso: calc.exe + WININET.dll**
> DLLs hijackeables conocidas para `calc.exe`: `CRYPTBASE.DLL`, `edputil.dll`, `MLANG.dll`, `PROPSYS.dll`, `Secur32.dll`, `SSPICLI.DLL`, `WININET.dll`

**Pasos del ataque:**
1. Renombrar `reflective_dll.x64.dll` (Stephen Fewer's reflective DLL) → `WININET.dll`
2. Copiar `calc.exe` (de `C:\Windows\System32`) + `WININET.dll` maliciosa a un directorio con permisos de escritura (ej: Desktop)
3. Ejecutar `calc.exe` → en vez de la calculadora, aparece un MessageBox ("Hello from DllMain!")

### Comparación: DLL maliciosa vs. legítima

| Campo | WININET.dll maliciosa | WININET.dll legítima |
|---|---|---|
| ProcessID | 6212 | 5464 |
| Image | calc.exe (fuera de System32) | calc.exe (System32) |
| ImageLoaded | WININET.dll | wininet.dll |
| **Signed** | **false** ⚠️ | **true** |
| User | DESKTOP-N33HELB\Waldo | DESKTOP-N33HELB\Waldo |

### 🚩 IOCs identificados para esta detección

> [!WARNING]
> **Los 3 indicadores clave**
> 1. **Ubicación de calc.exe**: nunca debería estar fuera de `System32`/`Syswow64` — una copia en otro directorio con permisos de escritura ya es sospechosa
> 2. **WININET.dll cargada fuera de System32 con calc.exe como padre**: indicador fuerte de hijacking — el nombre de la DLL no puede cambiarse (el atacante no puede evadir esto)
> 3. **Estado de firma**: la DLL original está firmada por Microsoft; la inyectada NO
>
> ⚠️ Precaución: no todas las cargas de WININET.dll fuera de System32 son maliciosas (algunas apps empaquetan DLLs por estabilidad) — pero para `calc.exe` específicamente, es una señal confiable.

## 🎯 Detección 2: Inyección de PowerShell/C# no administrada

> [!NOTE]
> **Contexto: C# es un "managed language"**
> Requiere el **CLR (Common Language Runtime)** como entorno de ejecución backend. El código no se ejecuta directamente como ensamblador — se compila a bytecode que el CLR procesa.

### Herramienta: Process Hacker

> [!TIP]
> **Detección visual**
> Procesos administrados (.NET) aparecen resaltados (verde) en Process Hacker. Pasar el cursor sobre `powershell.exe` muestra: **"Process is managed (.NET)"**

**DLLs clave a vigilar en Modules:**
- `clr.dll`
- `clrjit.dll`

> [!WARNING]
> **Señal de alerta**
> Si estas DLLs aparecen cargadas en procesos que **normalmente NO las requieren** (ej: `spoolsv.exe`), sugiere un ataque de **execute-assembly** o **inyección de PowerShell no administrada**.

### Reproducción del ataque (PSInject)

```powershell
powershell -ep bypass
Import-Module .\Invoke-PSInject.ps1
Invoke-PSInject -ProcId [Process ID de spoolsv.exe] -PoshCode "V3JpdGUtSG9zdCAiSGVsbG8sIEd1cnU5OSEi"
```

> [!NOTE]
> **Resultado observable**
> `spoolsv.exe` (normalmente **no administrado**, servicio de Print Spooler firmado por Microsoft) pasa a estado **"managed by .NET"** tras la inyección.

**Validación cruzada:**
- Process Hacker → pestaña Modules → ver `clr.dll` cargado
- Sysmon Event ID 7 → `ImageLoaded: clr.dll` en proceso `spoolsv.exe`, firmado por Microsoft, usuario `NT AUTHORITY\SYSTEM`

## 🎯 Detección 3: Credential Dumping (Mimikatz)

> [!NOTE]
> **Contexto: LSASS**
> **Local Security Authority Subsystem Service** gestiona credenciales de usuarios — objetivo principal de herramientas como Mimikatz.

### Reproducción del ataque

```cmd
C:\Tools\Mimikatz> mimikatz.exe

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::logonpasswords
```

> [!WARNING]
> **`sekurlsa::logonpasswords`**
> Extrae hashes NTLM/SHA1 y contraseñas en texto plano (si están disponibles) accediendo directamente a la memoria de LSASS.

### Detección vía Sysmon Event ID 10 (ProcessAccess)

> [!NOTE]
> **Estructura del evento**
> ```
> SourceImage: AgentEXE.exe
> TargetImage: lsass.exe
> SourceUser: DESKTOP-R4PEEIF\waldo
> TargetUser: NT AUTHORITY\SYSTEM
> ```

### 🚩 IOCs de credential dumping

> [!WARNING]
> **Señales de alerta**
> 1. **Proceso origen inusual**: un ejecutable random desde carpetas como `Downloads` accediendo a `lsass.exe`
> 2. **SourceUser ≠ TargetUser**: ej. `waldo` (SourceUser) accediendo como `SYSTEM` (TargetUser) — anomalía clara
> 3. **SeDebugPrivilege solicitado**: requerido por el proceso de dumping estilo Mimikatz — otro IOC más

> [!WARNING]
> **Cuidado con falsos positivos**
> Algunos procesos **legítimos** acceden a LSASS normalmente: procesos de autenticación, AV/EDR. No todo acceso a LSASS es malicioso — hay que correlacionar con los otros IOCs.

## 🖥️ Lab práctico

```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:[Target IP] /dynamic-resolution
```

> [!NOTE]
> **Tareas del lab (requieren reproducir los 3 ataques en el entorno propio)**
> 1. Replicar DLL hijacking → obtener SHA256 de la WININET.dll maliciosa
> 2. Replicar Unmanaged PowerShell → obtener SHA256 de clrjit.dll cargado por spoolsv.exe
> 3. Replicar Credential Dumping → obtener hash NTLM del usuario Administrator
>
> Herramientas ya provistas en el target: `C:\Tools\Sysmon`, `C:\Tools\Reflective DLL Injection`, `C:\Tools\PSInject`, `C:\Tools\Mimikatz`

> [!WARNING]
> No se incluyen hashes/respuestas exactas — son específicos de cada spawn del lab. La metodología completa para reproducir y detectar cada ataque está documentada arriba.

## 🧠 Resumen de IOCs por técnica (cheatsheet rápido)

| Técnica | Sysmon Event ID | IOCs clave |
|---|---|---|
| DLL Hijacking | 7 (Image Loaded) | Ubicación fuera de System32, nombre de DLL fijo, firma ausente |
| Unmanaged PowerShell Injection | 7 (Image Loaded) | `clr.dll`/`clrjit.dll` en proceso que normalmente no los usa |
| Credential Dumping (LSASS) | 10 (ProcessAccess) | SourceUser≠TargetUser, proceso origen inusual, SeDebugPrivilege |

## 🔗 Relacionado
- [Windows Event Logs](01-windows-event-logs.md)
- [ETW Fundamentals](03-etw-fundamentals.md)
- *Sysmon Config Cheatsheet*
- *Mimikatz - Notas*

#cjca #modulo18 #sysmon #dll-hijacking #credential-dumping #lsass #unmanaged-powershell #ioc
