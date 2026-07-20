# Módulo 19 — Security Monitoring & SIEM Fundamentals

## Sección 6/11: SIEM Visualization Example 1 — Failed Logon Attempts (All Users)

## 📌 Concepto: Dashboards y Visualizaciones

> [!NOTE]
> **¿Qué es un dashboard?**
> Contenedor de **múltiples visualizaciones**, permitiendo organizar y mostrar datos de forma significativa.

## 🛠️ Creando el primer Dashboard y Visualización

### Pasos de configuración inicial

```
1. http://[Target IP]:5601 -> toggle navegacion -> Dashboard
2. Eliminar dashboard existente "SOC-Alerts" (si aplica)
3. Click "Create new dashboard"
4. Click "Create visualization"
5. Calendario -> especificar "last 15 years" -> Apply
```

### Los 4 elementos clave de la ventana de creación

| Elemento | Función |
|---|---|
| **Filter** | Filtra datos antes de graficar (ej: `event.code: 4625`) |
| **Index pattern** | Dataset a usar (ej: `windows*` para separar de network, Linux, etc.) |
| **Search bar** | Verifica existencia de un campo específico en el dataset |
| **Visualization type** | Tipo de gráfico (Bar vertical stacked, Table, Metric, etc.) |

> [!TIP]
> **Filtro para failed logons**
> ```
> event.code: 4625  (operador "is")
> ```
> Event ID 4625 = failed logon attempt en Windows.

> [!WARNING]
> **`.keyword` para agregaciones**
> Usar `user.name.keyword` en vez de `user.name` cuando se trabaja con **agregaciones** (conteos, top values). El sufijo `.keyword` indica el campo como texto exacto no analizado — necesario para que Elasticsearch agrupe correctamente.

## 📊 Configurando la tabla

### Paso 1: Elegir tipo de visualización → "Table"

### Paso 2: Configurar "Rows"

```
Field: user.name.keyword
Values: top 1000
Rank by: Count of records (descendente)
```

> [!NOTE]
> **Nota sobre el ranking inicial**
> Puede aparecer "Rank by Alphabetical" en vez de "Count of records" al principio — es normal, se habilita automáticamente al configurar Metrics.

### Paso 3: Configurar "Metrics"

```
Metric: Count
```

> [!TIP]
> **Resultado**
> Al seleccionar "Count", la tabla se puebla automáticamente con datos (asumiendo que hay eventos en el dataset).

### Paso 4: Agregar segunda fila — máquina de origen

```
Field: host.hostname.keyword
```

> [!NOTE]
> **Resultado final: 3 columnas**
> 1. Username (login)
> 2. Máquina donde ocurrió el intento
> 3. Número de ocurrencias del evento

> [!WARNING]
> **Problema identificado**
> La tabla actual muestra tanto **usuarios** como **cuentas de computadora** — idealmente se necesita un filtro para excluir cuentas de máquina y mostrar solo usuarios reales.

### Guardar

```
Save and return -> agrega la visualizacion al dashboard
Save (dashboard) -> titulo: "SOC-Alerts"
```

## 🔧 Refinando la visualización — Feedback del SOC Manager

> [!NOTE]
> **Requisitos solicitados**
1. Nombres de columna más claros
2. Incluir el **Logon Type**
3. Resultados ordenados
4. Excluir usernames: `DESKTOP-DPOESND`, `WIN-OK9BH1BCKSD`, `WIN-RMMGJA7T9TC`
5. No monitorear cuentas de computadora (mala práctica dejarlas)

### Aplicando los cambios

```
Dashboard -> editar (icono lapiz) -> gear icon en la visualizacion -> "Edit lens"
```

**Renombrar campos (display name):**

| Campo original | Display name |
|---|---|
| `user.name.keyword` (top values) | **Username** |
| `host.hostname.keyword` (top values) | **Event logged by** |
| `winlog.logon.type.keyword` (nuevo, agregado) | **Logon Type** |
| Count of records (metric) | **# of logins** (alineación: Right) |

> [!TIP]
> **Nuevo campo: Logon Type**
> Se agrega usando `winlog.logon.type.keyword` — permite distinguir el tipo de logon (interactivo, red, servicio, RDP, etc.) asociado a cada intento fallido.

### Ordenar resultados

> [!NOTE]
> **Sorting**
> Configurar orden descendente en la columna **"# of logins"** para ver primero los usuarios con más intentos fallidos.

### Excluir usernames específicos

```
Filter: user.name.keyword IS NOT "DESKTOP-DPOESND"
```
(repetir para los otros 2 hostnames)

### Excluir cuentas de computadora (vía KQL)

```
NOT user.name: *$ AND winlog.channel.keyword: Security
```

> [!TIP]
> **Por qué esta sintaxis**
> - `NOT user.name: *$` → excluye cuentas que terminan en `$` (convención de Windows para **cuentas de máquina**, ej: `DESKTOP-ABC123$`)
> - `AND winlog.channel.keyword: Security` → asegura que solo se consideren logs del canal **Security**, evitando datos no relacionados

### Título final

```
Click en "No Title" -> asignar titulo descriptivo a la visualizacion -> Save
```

## 🖥️ Ejercicio práctico

> [!WARNING]
> **Tiempo de espera**
> Esperar 3-5 minutos tras spawnear el target para que Kibana esté disponible.

> [!NOTE]
> **Tarea del lab**
> Navegar al dashboard → revisar la visualización refinada ("Failed logon attempts [All users]") → identificar el número de logins para la cuenta **sql-svc1**.

> [!WARNING]
> No se incluye el número exacto — depende del dataset específico del spawn. La metodología completa de construcción/refinamiento de la visualización está documentada arriba para reproducirla.

## 🧠 Resumen del flujo completo (cheatsheet)

```
1. Crear dashboard -> crear visualizacion
2. Definir rango de tiempo (calendario)
3. Elegir index pattern (windows*)
4. Elegir tipo de viz (Table)
5. Configurar Rows (username.keyword, hostname.keyword, logon.type.keyword)
6. Configurar Metrics (Count -> renombrar a "# of logins")
7. Aplicar filtros de exclusion (usernames especificos + cuentas de maquina via KQL)
8. Ordenar resultados
9. Renombrar campos con display names claros
10. Titulo + Save
```

## 🔗 Relacionado
- [SIEM Use Case Development](05-siem-use-case-development.md)
- [SIEM Visualization Example 2: Failed Logon Disabled Users](07-siem-visualization-example-2-disabled-users.md)

#cjca #modulo19 #siem-visualization #kibana #dashboard #kql #failed-logon #event-4625
