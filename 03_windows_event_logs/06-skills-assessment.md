# Módulo 18 — Windows Event Logs & Finding Evil

## Sección 6/6 (Final): Skills Assessment

## 🎯 Escenario

> [!NOTE]
> **Contexto**
> El SOC manager asigna la tarea de analizar logs de ataques antiguos ubicados en `C:\Logs\*` y responder preguntas específicas — evaluación práctica integrando todo lo visto en el módulo (Event Viewer, Sysmon, ETW, Get-WinEvent).

## 🔌 Conexión al lab

```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:[Target IP] /dynamic-resolution
```

> [!WARNING]
> No se incluyen las respuestas exactas (nombres de proceso específicos, Yes/No) — son propias de cada spawn. A continuación, la **metodología de investigación** para cada directorio de logs, aplicando directamente lo cubierto en las secciones 1-5.

## 📁 Metodología por escenario

### `C:\Logs\DLLHijack` — DLL Hijacking

> [!TIP]
> **Enfoque (Sección 2)**
> Buscar **Sysmon Event ID 7** (Image Loaded) y aplicar los 3 IOCs vistos:
> 1. Proceso ejecutándose fuera de su ubicación esperada (System32/Syswow64)
> 2. DLL con nombre "fijo" cargada desde ubicación anómala
> 3. Estado de firma (`Signed: false` en la DLL sospechosa)

```powershell
Get-WinEvent -Path 'C:\Logs\DLLHijack\<archivo>.evtx' -FilterXPath "*[System[(EventID=7)]]" |
  Select-Object TimeCreated, Message | Format-List
```

O directamente en Event Viewer: **Open Saved Log** → filtrar por Event ID 7 → buscar el proceso con `Signed=false` cargando una DLL fuera de su carpeta habitual.

### `C:\Logs\PowershellExec` — PowerShell/C# no administrado

> [!TIP]
> **Enfoque (Sección 2 + 4)**
> Buscar carga anómala de `clr.dll` / `clrjit.dll` (Sysmon Event ID 7) en un proceso que normalmente **no** debería ejecutar código .NET.

**Dos preguntas relacionadas:**
1. **Qué proceso ejecutó** el código no administrado → el proceso donde aparecen cargadas `clr.dll`/`clrjit.dll` de forma anómala
2. **Qué proceso inyectó** en ese proceso → correlacionar con **Sysmon Event ID 8** (CreateRemoteThread) o Event ID 10 (ProcessAccess), buscando el `SourceImage` que interactuó con el proceso identificado en la pregunta anterior

```powershell
# Buscar carga de clr.dll/clrjit.dll
Get-WinEvent -Path 'C:\Logs\PowershellExec\<archivo>.evtx' -FilterXPath "*[System[(EventID=7)]] and *[EventData[Data='clr.dll']]"

# Buscar quién inyectó (CreateRemoteThread = Event ID 8, o ProcessAccess = Event ID 10)
Get-WinEvent -Path 'C:\Logs\PowershellExec\<archivo>.evtx' -FilterXPath "*[System[(EventID=8 or EventID=10)]]"
```

### `C:\Logs\Dump` — LSASS Dumping

> [!TIP]
> **Enfoque (Sección 2)**
> Buscar **Sysmon Event ID 10** (ProcessAccess) con `TargetImage: lsass.exe`.

```powershell
Get-WinEvent -Path 'C:\Logs\Dump\<archivo>.evtx' -FilterXPath "*[EventData[Data[@Name='TargetImage']='C:\Windows\system32\lsass.exe']]"
```

> [!WARNING]
> **IOCs clave a buscar (repaso de Sección 2)**
> - `SourceImage` inusual (ruta/carpeta atípica)
> - `SourceUser` ≠ `TargetUser`
> - Solicitud de `SeDebugPrivilege`

**Segunda pregunta: ¿login malicioso tras el dump?**

> [!TIP]
> **Correlacionar con Event ID 4624 (Security log)**
> Tras identificar el timestamp del dump, buscar eventos de **logon exitoso (4624)** posteriores a esa marca de tiempo — especialmente con `Logon Type` inusual (ej: Network=3, RemoteInteractive=10) o desde una cuenta que no debería estar logueándose en ese momento (posible uso de credenciales robadas del dump).

```powershell
Get-WinEvent -Path 'C:\Logs\Dump\<archivo_security>.evtx' -FilterHashtable @{ID=4624} |
  Where-Object {$_.TimeCreated -gt "<timestamp del dump>"} |
  Select-Object TimeCreated, Message
```

### `C:\Logs\StrangePPID` — Parent PID Spoofing

> [!TIP]
> **Enfoque (Sección 4)**
> Buscar una relación padre-hijo que **no tiene sentido** en el comportamiento normal de Windows (ej: `spoolsv.exe` creando `cmd.exe` sin argumentos, tal como se vio en el ejemplo del módulo).

```powershell
Get-WinEvent -Path 'C:\Logs\StrangePPID\<archivo>.evtx' -FilterHashtable @{ID=1} |
  Select-Object TimeCreated, Message | Format-List
```

> [!WARNING]
> **Recordar la limitación de Sysmon aquí**
> Si el log es **solo de Sysmon**, el `ParentImage` mostrado puede estar **falsificado** (por la técnica de Parent PID Spoofing) — el proceso "temporal" que ejecutó el código es el que aparece como hijo, aunque su padre reportado no sea el real. Si hay logs de ETW/Kernel-Process disponibles en el mismo directorio, cruzarlos para confirmar el padre verdadero.

## 🧠 Checklist general de la evaluación (resumen de todo el módulo)

> [!NOTE]
> **Flujo de investigación aplicado**
- [ ] Identificar el Event ID relevante para la técnica sospechada (7, 8, 10, 1, 4624, etc.)
- [ ] Usar `Get-WinEvent -Path` o Event Viewer (Open Saved Log) para leer el `.evtx` específico
- [ ] Aplicar `-FilterXPath` o `-FilterHashtable` para reducir ruido
- [ ] Correlacionar entre distintos Event IDs (proceso → acceso → logon) para reconstruir la cadena completa
- [ ] Verificar IOCs específicos de cada técnica (firma, ubicación, privilegios, usuario origen/destino)

## 🏁 Cierre del módulo

Con esta sección se completa **Windows Event Logs & Finding Evil** (6/6). Recorrido completo:
1. Fundamentos de Event Logs y Event Viewer (anatomía de un evento, XML queries)
2. Sysmon: detección de DLL hijacking, PowerShell no administrado, credential dumping
3. ETW: arquitectura, proveedores, Logman
4. Aprovechando ETW: Parent PID Spoofing, detección de ensamblados .NET (BYOL) con SilkETW
5. Get-WinEvent: análisis masivo, FilterHashtable, FilterXml, FilterXPath
6. Skills Assessment: aplicación integrada de todo lo anterior

## 🔗 Relacionado
- [Get-WinEvent](05-get-winevent.md)
- [Analyzing Evil With Sysmon & Event Logs](02-analyzing-evil-sysmon-event-logs.md)
- [Tapping Into ETW](04-tapping-into-etw.md)
- *Cheatsheet Event IDs Sysmon*

#cjca #modulo18 #skills-assessment #sysmon #get-winevent #dll-hijacking #lsass #parent-pid-spoofing #modulo-completo
