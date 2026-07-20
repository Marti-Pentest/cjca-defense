# Módulo 19 — Security Monitoring & SIEM Fundamentals

## Sección 9/11: SIEM Visualization Example 4 — Users Added Or Removed From A Local Group

## 📌 Contexto y motivación

> [!NOTE]
> **Objetivo**
> Monitorear adiciones/remociones de usuarios del grupo local **"Administrators"**, desde el **5 de marzo de 2023** hasta la fecha.

> [!NOTE]
> **Eventos base**
> - **Event ID 4732**: A member was added to a security-enabled local group
> - **Event ID 4733**: A member was removed from a security-enabled local group

## 🛠️ Construcción de la visualización

### Filtros clave

```
event.code: 4732 OR 4733
group.name: Administrators
```

> [!TIP]
> **Por qué ambos filtros**
> El filtro de `event.code` captura tanto adiciones como remociones; el filtro de `group.name` acota **específicamente** al grupo Administrators (podría haber otros grupos locales generando estos mismos Event IDs).

### Configuración

```
Index pattern: windows*
Tipo de visualizacion: Table
```

**Rows (5 campos):**

| Campo | Pregunta que responde |
|---|---|
| `user.name.keyword` | ¿Quién realizó la acción? (usuario/proceso responsable) |
| `winlog.event_data.MemberSid.keyword` | ¿Qué usuario fue agregado/removido? (SID del miembro afectado) |
| `group.name.keyword` | ¿A qué grupo? (verificación doble de que es "Administrators") |
| `event.action.keyword` | ¿Fue una adición o una remoción? |
| `host.name.keyword` | ¿En qué máquina ocurrió? |

**Metrics:**
```
Count
```

```
Save and return -> se agrega al dashboard
```

## ⏰ Acotando el rango temporal

> [!WARNING]
> **Detalle técnico crítico: Elasticsearch usa "buckets"**
> Elasticsearch agrega datos (rangos de tiempo, etc.) en **buckets**. Es indispensable usar un **rango de tiempo absoluto** al crear la visualización.
>
> Si no se hace esto, el bucket mostrará un intervalo **semanal** en vez de **diario** — distorsionando el análisis granular por día.

> [!TIP]
> **Ejemplo de rango absoluto correcto**
> `1 de enero – 31 de enero de 2023` (fechas específicas, no relativas tipo "last 7 days")

**Pasos:**
```
1. Editar panel -> Customize time range
2. Establecer fecha de inicio: March 5, 2023 (absoluta)
3. Save (dashboard) para persistir los cambios
```

## 🖥️ Ejercicio práctico

> [!WARNING]
> **Tiempo de espera**
> Esperar 3-5 minutos tras spawnear el target para que Kibana esté disponible.

> [!NOTE]
> **Tarea del lab**
> Extender la visualización "User added or removed from a local group" → identificar la **fecha común** en la que ocurrieron todos los eventos devueltos (formato `20XX-0X-0X`).

> [!WARNING]
> No se incluye la fecha específica — depende del dataset del spawn. El enfoque es revisar el campo `@timestamp` de los documentos devueltos tras aplicar el rango temporal absoluto correcto.

## 🧠 Resumen del flujo (cheatsheet)

```
1. Filtro: event.code: 4732 OR 4733 (member added/removed)
2. Filtro: group.name: Administrators
3. Table -> Rows: user.name + MemberSid + group.name + event.action + host.name (todos .keyword)
4. Metrics: Count
5. Rango de tiempo: ABSOLUTO (no relativo) desde 2023-03-05 hasta hoy
6. Save
```

> [!TIP]
> **Por qué esta visualización es valiosa para el SOC**
> Cambios de membresía en grupos privilegiados (como Administrators) son un **indicador clásico de escalada de privilegios** o persistencia post-compromiso. Monitorear `event.action` (added vs removed) + `host.name` + `MemberSid` permite detectar rápidamente adiciones no autorizadas a grupos de alto privilegio.

## 🔗 Relacionado
- [SIEM Visualization Example 3: RDP Service Accounts](08-siem-visualization-example-3-rdp-service-accounts.md)
- [The Triaging Process](10-the-triaging-process.md)

#cjca #modulo19 #siem-visualization #kibana #privilege-escalation #group-membership #event-4732 #event-4733 #elasticsearch-buckets
