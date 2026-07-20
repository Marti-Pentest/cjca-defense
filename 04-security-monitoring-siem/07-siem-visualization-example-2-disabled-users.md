# Módulo 19 — Security Monitoring & SIEM Fundamentals

## Sección 7/11: SIEM Visualization Example 2 — Failed Logon Attempts (Disabled Users)

## 📌 Concepto: por qué "failed" para usuarios deshabilitados

> [!NOTE]
> **Lógica clave**
> No es posible loguearse exitosamente con un usuario **deshabilitado** — nunca será "successful" aunque se provean las credenciales correctas. Cuando esto ocurre, el log de Windows incluye el campo adicional **`SubStatus: 0xC0000072`**, indicando la razón específica del fallo.

## 🛠️ Construcción de la visualización

### Filtro clave

```
event.code: 4625
winlog.event_data.SubStatus: 0xc0000072   (operador "is")
```

> [!TIP]
> **Diferencia con la visualización anterior**
> Se agrega el filtro de `SubStatus` para acotar **específicamente** a fallos causados por cuentas deshabilitadas, no cualquier fallo de logon.

### Configuración

```
Index pattern: windows*
Tipo de visualizacion: Table
```

**Rows:**
```
Field: user.name.keyword
Values: top 1000
Rank by: Count of records (descendente)
```

**Metrics:**
```
Count
```

**Segunda fila (máquina de origen):**
```
Field: host.hostname.keyword
```

> [!NOTE]
> **Resultado: 3 columnas**
> 1. Usuario deshabilitado cuyas credenciales generaron el intento fallido
> 2. Máquina donde ocurrió el intento
> 3. Número de ocurrencias

```
Save and return -> se agrega al dashboard
```

## 🖥️ Ejercicios prácticos

> [!WARNING]
> **Tiempo de espera**
> Esperar 3-5 minutos tras spawnear el target para que Kibana esté disponible.

### Tarea 1: Agregar Logon Type a la visualización

> [!NOTE]
> **Objetivo**
> Editar/crear la visualización "Failed logon attempts [Disabled user]" agregando el **logon type** — mismo campo usado en la sección anterior: `winlog.logon.type.keyword`.

> [!WARNING]
> No se incluye el logon type específico devuelto — depende del documento real en el dataset del spawn. El campo a inspeccionar es `winlog.logon.type.keyword` en el documento resultante.

### Tarea 2: Filtrar usuarios "admin" (wildcard)

> [!NOTE]
> **Objetivo**
> Crear/editar "Failed logon attempts [Admin users only]" para incluir cualquier username que contenga "admin" **en cualquier posición** del string (no solo al inicio).

> [!TIP]
> **Diferencia entre wildcard "empieza con" vs "contiene"**
> Recordar de la Sección 2:
> ```
> user.name: admin*      ->  empieza con "admin" (admin, administrator, admin123)
> user.name: *admin*     ->  contiene "admin" en cualquier posicion (ej: "sql-admin-2", "localadmin")
> ```

> [!WARNING]
> **Respuesta esperada para la Tarea 2**
> Lo que va después de `user.name:` para capturar "admin" en cualquier parte del string es:
> ```
> *admin*
> ```

## 🧠 Resumen del flujo (cheatsheet)

```
1. Filtro base: event.code: 4625
2. Filtro especifico: winlog.event_data.SubStatus: 0xc0000072 (disabled user)
3. Table -> Rows: user.name.keyword + host.hostname.keyword
4. Metrics: Count
5. (Extension) Agregar Rows: winlog.logon.type.keyword
6. (Variante) Filtro KQL wildcard: user.name: *admin* para buscar "admin" en cualquier posicion
```

## 🔗 Relacionado
- [SIEM Visualization Example 1: All Users](06-siem-visualization-example-1-all-users.md)
- [SIEM Visualization Example 3: RDP Service Accounts](08-siem-visualization-example-3-rdp-service-accounts.md)
- [Introduction To The Elastic Stack](02-introduccion-elastic-stack.md)

#cjca #modulo19 #siem-visualization #kibana #disabled-users #substatus #kql-wildcards #event-4625
