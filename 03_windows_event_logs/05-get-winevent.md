# Módulo 18 — Windows Event Logs & Finding Evil

## Sección 5/6: Get-WinEvent

## 📌 Por qué importa el análisis masivo

> [!NOTE]
> **Contexto**
> En organizaciones grandes se generan **millones de eventos por día**. Para extraer información útil de este volumen, se necesitan herramientas eficientes de consulta masiva. **`Get-WinEvent`** (PowerShell) es la herramienta central para esto.

> [!TIP]
> **Capacidades de Get-WinEvent**
> Recupera logs clásicos (System, Application), logs de Windows Event Logging Technology, y logs de **ETW** — todo desde el mismo cmdlet.

## 🔍 Explorar logs y proveedores disponibles

```powershell
# Listar TODOS los logs disponibles con propiedades clave
Get-WinEvent -ListLog * | Select-Object LogName, RecordCount, IsClassicLog, IsEnabled, LogMode, LogType | Format-Table -AutoSize
```

> [!NOTE]
> **Propiedades clave devueltas**
> - **IsClassicLog**: formato `.evt` (clásico) vs `.evtx` (nuevo)
> - **LogMode**: Circular, Retain, o AutoBackup
> - **LogType**: Administrative, Analytic, Debug, u Operational

```powershell
# Listar proveedores y sus logs asociados
Get-WinEvent -ListProvider * | Format-Table -AutoSize
```

> [!TIP]
> **Utilidad de -ListProvider**
> Permite identificar qué proveedor está vinculado a qué log — útil para saber dónde filtrar antes de armar una query específica.

## 🛠️ Recuperar eventos específicos

```powershell
# Últimos 50 eventos del log System
Get-WinEvent -LogName 'System' -MaxEvents 50 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize

# Eventos de un log operacional específico
Get-WinEvent -LogName 'Microsoft-Windows-WinRM/Operational' -MaxEvents 30 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

> [!TIP]
> **Parámetro `-Oldest`**
> Recupera eventos en **orden cronológico ascendente** (los más antiguos primero) en vez de tener que ordenar manualmente.

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-WinRM/Operational' -Oldest -MaxEvents 30 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

## 📁 Leer archivos .evtx exportados

> [!NOTE]
> **Caso de uso**
> Auditoría o análisis forense de logs exportados/respaldados de otra máquina.

```powershell
Get-WinEvent -Path 'C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Execution\exec_sysmon_1_lolbin_pcalua.evtx' -MaxEvents 5 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

## 🎯 Filtrado con `-FilterHashtable`

```powershell
# Filtrar por LogName + IDs específicos
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1,3} | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

> [!WARNING]
> **IOC potencial: Event ID 1 + 3 en corto período**
> Si un proceso "peligroso"/poco común (ID 1 — Process Create) genera una conexión de red (ID 3 — Network Connection) en poco tiempo, puede indicar comunicación con un **servidor C2**.

```powershell
# Mismo filtro pero sobre un archivo .evtx exportado
Get-WinEvent -FilterHashtable @{Path='C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Execution\sysmon_mshta_sharpshooter_stageless_meterpreter.evtx'; ID=1,3} | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

### Filtrado por rango de fechas

```powershell
$startDate = (Get-Date -Year 2023 -Month 5 -Day 28).Date
$endDate   = (Get-Date -Year 2023 -Month 6 -Day 3).Date
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1,3; StartTime=$startDate; EndTime=$endDate} | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

> [!WARNING]
> **StartTime es inclusivo, EndTime es exclusivo**
> Por eso se especifica el día **siguiente** al último día deseado en el rango (ej: día 3 de junio para incluir hasta el 2 de junio completo).

## 🧬 Filtrado con Hashtable + parseo XML

> [!NOTE]
> **Escenario: investigar conexión sospechosa a una IP**
> Combinar `-FilterHashtable` con parseo manual del XML del evento para extraer campos específicos no expuestos directamente por el objeto.

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=3} |
ForEach-Object {
    $xml = [xml]$_.ToXml()
    $eventData = $xml.Event.EventData.Data
    New-Object PSObject -Property @{
        SourceIP = $eventData | Where-Object {$_.Name -eq "SourceIp"} | Select-Object -ExpandProperty '#text'
        DestinationIP = $eventData | Where-Object {$_.Name -eq "DestinationIp"} | Select-Object -ExpandProperty '#text'
        ProcessGuid = $eventData | Where-Object {$_.Name -eq "ProcessGuid"} | Select-Object -ExpandProperty '#text'
        ProcessId = $eventData | Where-Object {$_.Name -eq "ProcessId"} | Select-Object -ExpandProperty '#text'
    }
}  | Where-Object {$_.DestinationIP -eq "52.113.194.132"}
```

> [!TIP]
> **Uso del ProcessGuid**
> Permite rastrear el **proceso original** que hizo la conexión — clave para reconstruir el árbol de procesos e identificar el ejecutable/script malicioso responsable.

> [!NOTE]
> **Referencia técnica**
> El formato del EVTX (de dónde sale `Event.EventData.Data`) está documentado en el [esquema oficial de Windows Event Log XML](https://docs.microsoft.com/en-us/windows/win32/wes/eventschema-schema).

### Ejemplo: detectar carga de clr.dll/mscoree.dll con Query XML manual

```powershell
$Query = @"
<QueryList>
    <Query Id="0">
        <Select Path="Microsoft-Windows-Sysmon/Operational">*[System[(EventID=7)]] and *[EventData[Data='mscoree.dll']] or *[EventData[Data='clr.dll']]
        </Select>
    </Query>
</QueryList>
"@
Get-WinEvent -FilterXml $Query | ForEach-Object {Write-Host $_.Message `n}
```

> [!TIP]
> **Conexión con la sección anterior (ETW)**
> Este query complementa lo visto con SilkETW/DotNETRuntime — detecta el mismo patrón (carga de DLLs de .NET) pero directamente desde Sysmon Event ID 7 vía PowerShell, sin herramientas adicionales.

## 🎯 Filtrado con `-FilterXPath`

```powershell
# Detectar instalación de herramientas de Sysinternals (aceptación de EULA)
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath "*[EventData[Data[@Name='Image']='C:\Windows\System32\reg.exe']] and *[EventData[Data[@Name='CommandLine']='`"C:\Windows\system32\reg.exe`" ADD HKCU\Software\Sysinternals /v EulaAccepted /t REG_DWORD /d 1 /f']]" | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

```powershell
# Investigar conexiones de red a una IP sospechosa específica
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath "*[System[EventID=3] and EventData[Data[@Name='DestinationIp']='52.113.194.132']]"
```

## 🔎 Filtrado por valores de propiedades (`-Property *` + `Where-Object`)

```powershell
# Ver TODAS las propiedades de un evento (para descubrir índices de array)
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1} -MaxEvents 1 | Select-Object -Property *
```

> [!NOTE]
> **Detectar PowerShell codificado (`-enc`) vía índice de propiedad**
> El array `.Properties[]` de un evento "Process Create" contiene los campos en orden fijo. El índice **21** corresponde a `ParentCommandLine`.

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1} | Where-Object {$_.Properties[21].Value -like "*-enc*"} | Format-List
```

> [!WARNING]
> **Por qué esto es un IOC**
> `-enc` es el parámetro de PowerShell para pasar un **comando codificado en Base64** — técnica común para ofuscar scripts maliciosos. Ver esta cadena en la línea de comandos padre es señal de alerta.

### Desglose del comando

| Componente | Función |
|---|---|
| `$_` | Objeto actual en el pipeline (cada evento individual) |
| `.Properties[21].Value` | Array de propiedades del evento Sysmon — índice 21 = `ParentCommandLine` |
| `-like "*-enc*"` | Coincidencia de string con wildcards — busca `-enc` en cualquier posición |
| `Format-List` | Muestra propiedades como lista (más legible que tabla para output largo) |

## 🖥️ Lab práctico

```bash
xfreerdp /u:Administrator /p:'HTB_@cad3my_lab_W1n10_r00t!@0' /v:[Target IP] /dynamic-resolution
```

> [!NOTE]
> **Tarea del lab**
> Usar `Get-WinEvent` para recorrer todos los event logs dentro de `C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Lateral Movement` y determinar cuándo se agregó el share `\\*\PRINT`. Responder con el timestamp exacto (`HH:MM:SS`).

> [!TIP]
> **Enfoque sugerido para resolverlo**
> Combinar `-Path` (para leer todos los `.evtx` del directorio, posiblemente iterando con `Get-ChildItem`) con `-FilterHashtable` o `-FilterXPath` buscando eventos relacionados a creación de shares de red (Event ID 5140/5142 en Security, o el equivalente en los evtx de muestra).

> [!WARNING]
> No se incluye el timestamp exacto — requiere ejecutar la consulta contra los archivos reales del lab.

## 🧠 Cheatsheet de parámetros de Get-WinEvent

| Parámetro | Uso |
|---|---|
| `-ListLog *` | Lista todos los logs disponibles |
| `-ListProvider *` | Lista todos los proveedores y sus logs vinculados |
| `-LogName` | Especifica el log a consultar |
| `-Path` | Lee desde un archivo `.evtx` exportado |
| `-MaxEvents` | Limita cantidad de eventos devueltos |
| `-Oldest` | Devuelve eventos en orden cronológico ascendente |
| `-FilterHashtable` | Filtro estructurado (LogName, ID, StartTime, EndTime, Path) |
| `-FilterXml` | Query XML completo (igual sintaxis que Event Viewer) |
| `-FilterXPath` | Query en sintaxis XPath |

## 🔗 Relacionado
- [Tapping Into ETW](04-tapping-into-etw.md)
- [Skills Assessment](06-skills-assessment.md)
- [Windows Event Logs](01-windows-event-logs.md)
- *EVTX Schema Reference*
- *PowerShell Get-WinEvent Cheatsheet*

#cjca #modulo18 #get-winevent #powershell #sysmon #xpath #filterhashtable #threat-hunting
