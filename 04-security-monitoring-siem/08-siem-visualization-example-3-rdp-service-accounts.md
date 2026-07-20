# Módulo 19 — Security Monitoring & SIEM Fundamentals

## Sección 8/11: SIEM Visualization Example 3 — Successful RDP Logon Related To Service Accounts

## 📌 Contexto y motivación

> [!WARNING]
> **Regla de negocio clave**
> Las credenciales de **service accounts nunca deberían usarse para RDP** en entornos corporativos reales. IT Operations informó que todas las service accounts del entorno empiezan con **`svc-`**.

> [!TIP]
> **Por qué importa monitorear esto**
> Las service accounts suelen tener **privilegios excepcionalmente altos** — su uso indebido (ej: RDP interactivo) es una señal fuerte de compromiso o mal uso.

> [!NOTE]
> **Evento base**
> **Event ID 4624** — "An account was successfully logged on"

## 🛠️ Construcción de la visualización

### Filtros clave (2 condiciones)

```
event.code: 4624                            (operador "is")
winlog.logon.type: RemoteInteractive         (operador "is")
```

> [!TIP]
> **RemoteInteractive = RDP**
> El `Logon Type` **RemoteInteractive** corresponde específicamente a sesiones RDP — necesario para distinguir de otros tipos de logon exitoso (interactivo local, red, servicio, etc.)

### Configuración

```
Index pattern: windows*
Tipo de visualizacion: Table
```

**Rows (3 campos):**

| Campo | Representa |
|---|---|
| `user.name.keyword` | Cuenta de servicio que generó el logon |
| `host.hostname.keyword` | Máquina donde ocurrió el RDP exitoso |
| `related.ip.keyword` | IP de la máquina que **inició** la conexión RDP |

**Metrics:**
```
Count
```

### Filtro final vía KQL — service accounts

```
user.name: svc-*
```

> [!WARNING]
> **Diferencia importante de sintaxis**
> En queries **KQL** (barra de búsqueda), **no se usa** el sufijo `.keyword` — a diferencia de los campos configurados en Rows/Metrics del editor visual, donde sí se usa para agregaciones.

## 📊 Resultado: 4 columnas

> [!NOTE]
> **Información mostrada**
> 1. Service account cuyas credenciales generaron el RDP exitoso
> 2. Máquina donde ocurrió el logon
> 3. IP de la máquina que **inició** la conexión
> 4. Número de ocurrencias

```
Save and return -> se agrega al dashboard
```

## 🖥️ Ejercicio práctico

> [!WARNING]
> **Tiempo de espera**
> Esperar 3-5 minutos tras spawnear el target para que Kibana esté disponible.

> [!NOTE]
> **Tarea del lab**
> Navegar al dashboard → revisar "RDP logon for service account" → identificar la **IP de la máquina que inició** el RDP exitoso usando credenciales de service account.

> [!WARNING]
> No se incluye la IP específica — depende del dataset del spawn. El campo a revisar es `related.ip.keyword` en la fila correspondiente de la tabla.

## 🧠 Resumen del flujo (cheatsheet)

```
1. Filtro: event.code: 4624 (logon exitoso)
2. Filtro: winlog.logon.type: RemoteInteractive (especifico de RDP)
3. Table -> Rows: user.name.keyword + host.hostname.keyword + related.ip.keyword
4. Metrics: Count
5. Filtro KQL final: user.name: svc-* (sin .keyword)
```

> [!TIP]
> **Patrón reutilizable**
> Esta metodología (Event ID base + Logon Type específico + filtro de naming convention) es aplicable a cualquier regla de negocio similar: ej. detectar cuentas de admin haciendo login interactivo remoto, o cuentas de servicio conectándose fuera de horario laboral.

## 🔗 Relacionado
- [SIEM Visualization Example 2: Disabled Users](07-siem-visualization-example-2-disabled-users.md)
- [SIEM Visualization Example 4: Group Membership Changes](09-siem-visualization-example-4-group-membership-changes.md)

#cjca #modulo19 #siem-visualization #kibana #rdp #service-accounts #event-4624 #logon-type
