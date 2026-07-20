# Módulo 18 — Windows Event Logs & Finding Evil

## Sección 4/6: Aprovechando ETW

## 🎯 Detección 1: Relaciones Padre-Hijo Extrañas

> [!NOTE]
> **Concepto base**
> En entornos Windows estándar, ciertos procesos **nunca** deberían crear a otros. Ej: es altamente improbable ver `calc.exe` creando `cmd.exe`. Conocer las relaciones padre-hijo normales permite detectar desviaciones.

> [!TIP]
> **Recurso de referencia**
> Samir Bousseaden compartió un mapa mental de relaciones padre-hijo comunes en Windows — útil como baseline de referencia.

### Herramienta: Process Hacker
Permite explorar jerárquicamente las relaciones padre-hijo entre procesos en ejecución.

> [!WARNING]
> **Ejemplo de anomalía**
> `spoolsv.exe` normalmente crea `conhost.exe`. Si en cambio crea `whoami.exe`, eso es sospechoso — desviación del comportamiento esperado.

### Técnica de ataque: Parent PID Spoofing

> [!WARNING]
> **¿Qué hace?**
> Permite que un proceso aparezca en los logs como si hubiera sido creado por un padre distinto al real — técnica de evasión de detección.

```powershell
# Ejecutar vía proyecto psgetsystem
PS C:\Tools\psgetsystem> powershell -ep bypass
PS C:\Tools\psgetsystem> Import-Module .\psgetsys.ps1
PS C:\Tools\psgetsystem> [MyProcess]::CreateProcessFromParent([Process ID de spoolsv.exe],"C:\Windows\System32\cmd.exe","")
```

> [!WARNING]
> **Resultado en Sysmon (engañoso)**
> El **Event 1 de Sysmon muestra incorrectamente** `spoolsv.exe` como padre de `cmd.exe`. En realidad, fue `powershell.exe` quien lo creó — Sysmon fue **evadido** por el spoofing.

### Solución: ETW con SilkETW

> [!TIP]
> **Identificar el proveedor correcto**
> ```cmd
> logman.exe query providers | findstr "Process"
> ```
> Proveedor relevante: **Microsoft-Windows-Kernel-Process**

```cmd
:: Capturar datos del proveedor con SilkETW
c:\Tools\SilkETW_SilkService_v8\v8\SilkETW>SilkETW.exe -t user -pn Microsoft-Windows-Kernel-Process -ot file -p C:\windows\temp\etw.json
```

> [!NOTE]
> **Resultado — ETW revela la verdad**
> El archivo `etw.json` muestra correctamente que fue **`powershell.exe`** quien creó `cmd.exe` — a diferencia de Sysmon, ETW **no fue engañado** por el Parent PID Spoofing.

> [!TIP]
> **Integración con Event Viewer**
> Los logs de SilkETW pueden ingerirse y visualizarse en el Event Viewer vía **SilkService**, dando visibilidad más profunda y extensa.

## 🎯 Detección 2: Carga de Ensamblados .NET Maliciosos

### Contexto: de "Living off the Land" (LotL) a "Bring Your Own Land" (BYOL)

> [!NOTE]
> **LotL (técnica tradicional)**
> Explotar herramientas legítimas ya presentes en el sistema (ej: PowerShell) para reducir el riesgo de detección.

> [!WARNING]
> **BYOL (evolución, término de Mandiant)**
> En vez de depender de herramientas del sistema, los atacantes ahora usan **ensamblados .NET ejecutados completamente en memoria** — herramientas personalizadas en C#, independientes del sistema objetivo.

**Por qué BYOL es efectivo:**
1. .NET viene **preinstalado** en todo sistema Windows
2. Es un lenguaje **gestionado** (CLR maneja garbage collection, memoria, etc.)
3. Los ensamblados .NET se pueden cargar **directamente en memoria** — sin escribir el ejecutable al disco → minimiza artefactos forenses
4. El framework .NET trae librerías robustas (HTTP, criptografía, IPC/named pipes) → herramientas potentes ya construidas para el atacante

> [!WARNING]
> **Ejemplo real: `execute-assembly` de Cobalt Strike**
> Comando que ejecuta ensamblados .NET directamente desde memoria — herramienta ideal para implementar BYOL en operaciones de Red Team/adversario real.

### Detección vía Sysmon (limitada)

> [!TIP]
> **Principio similar al de PowerShell no administrado**
> Vigilar la carga anómala de DLLs asociadas a .NET: **`clr.dll`** y **`mscoree.dll`** en procesos que normalmente no las requieren — vía **Sysmon Event ID 7** (Image Loaded).

### Simulación del ataque: Seatbelt

> [!NOTE]
> **¿Qué es Seatbelt?**
> Ensamblado .NET conocido, usado por adversarios para obtener **situational awareness** en un sistema comprometido (privilegios, configuración, etc.)

```powershell
# Ejecutar Seatbelt precompilado
PS C:\Tools\GhostPack Compiled Binaries>.\Seatbelt.exe TokenPrivileges
```

**Output de ejemplo (TokenPrivileges):**
```
SeDebugPrivilege:                 SE_PRIVILEGE_ENABLED
SeChangeNotifyPrivilege:          SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
SeImpersonatePrivilege:           SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
SeCreateGlobalPrivilege:          SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
```

> [!WARNING]
> **Limitación de Sysmon Event ID 7 en este caso**
> - Genera **gran volumen de eventos** (ruidoso si no está bien configurado)
> - Informa **qué DLLs se cargan**, pero NO da detalles granulares del **contenido real** del ensamblado .NET ejecutado

### Solución: ETW con proveedor Microsoft-Windows-DotNETRuntime

```cmd
:: Capturar con SilkETW, apuntando a keywords específicas (0x2038)
c:\Tools\SilkETW_SilkService_v8\v8\SilkETW>SilkETW.exe -t user -pn Microsoft-Windows-DotNETRuntime -uk 0x2038 -ot file -p C:\windows\temp\etw.json
```

> [!TIP]
> **Resultado**
> El JSON generado contiene información mucho más rica: **incluye nombres de métodos** del ensamblado cargado — visibilidad que Sysmon no puede dar.

### Desglose de las keywords usadas (0x2038)

| Keyword | Qué monitorea |
|---|---|
| **JitKeyword** | Eventos de compilación JIT — métodos compilados en tiempo de ejecución. Útil para entender el **flujo de ejecución** del ensamblado |
| **InteropKeyword** | Interoperabilidad entre código gestionado y no gestionado — posibles interacciones con APIs nativas |
| **LoaderKeyword** | Proceso de carga de ensamblados dentro del runtime .NET — qué se está cargando/ejecutando |
| **NGenKeyword** | Eventos de Native Image Generator — detecta uso de ensamblados **precompilados** para evadir detección basada en JIT |

## 🖥️ Lab práctico

```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:[Target IP] /dynamic-resolution
```

> [!NOTE]
> **Tarea del lab**
> Replicar la ejecución de Seatbelt + SilkETW → identificar el `ManagedInteropMethodName` que empieza con "G" y termina en "ion".
>
> Herramientas provistas: `c:\Tools\SilkETW_SilkService_v8\v8`, `C:\Tools\GhostPack Compiled Binaries`

> [!WARNING]
> No se incluye el nombre del método exacto — requiere reproducir el ataque en el spawn propio y examinar el JSON de salida de SilkETW buscando eventos de `InteropKeyword`.

## 🧠 Resumen: Sysmon vs ETW (idea central de la sección)

| Escenario | Sysmon | ETW (SilkETW) |
|---|---|---|
| Parent PID Spoofing | ❌ Engañado — muestra padre falso | ✅ Revela el padre real (Kernel-Process) |
| Carga de ensamblado .NET | ⚠️ Solo ve qué DLL se cargó (clr.dll, mscoree.dll) | ✅ Ve nombres de métodos y flujo de ejecución (DotNETRuntime) |

> [!TIP]
> **Conclusión clave del módulo**
> Sysmon es valioso, pero **no es infalible** — puede ser evadido (Parent PID Spoofing) o dar visibilidad limitada (ensamblados .NET). ETW ofrece una capa de telemetría más profunda y resistente a ciertas técnicas de evasión.

## 🔗 Relacionado
- [ETW Fundamentals](03-etw-fundamentals.md)
- [Get-WinEvent](05-get-winevent.md)
- [Analyzing Evil With Sysmon & Event Logs](02-analyzing-evil-sysmon-event-logs.md)
- *SilkETW Cheatsheet*
- *BYOL vs LotL - Notas*

#cjca #modulo18 #etw #silketw #parent-pid-spoofing #dotnet #byol #seatbelt #cobalt-strike
