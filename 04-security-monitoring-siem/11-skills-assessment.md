# Módulo 19 — Security Monitoring & SIEM Fundamentals

## Sección 11/11 (Final): Skills Assessment

## 🎯 Escenario: Onboarding en Eagle como SOC Tier 1

> [!NOTE]
> **Contexto del rol**
> Primer día completo como **SOC Tier 1 analyst** en "Eagle". Reunión con un senior analyst para entender el entorno, luego monitoreo del dashboard "SOC-Alerts".

## 📋 Notas del entorno (contexto crítico para el triaging)

> [!WARNING]
> **Información clave dada por el senior analyst**
> 1. **Toda la infraestructura está en cloud** — la DMZ vieja está cerrada, no hay servidores ahí
> 2. **IT Operations Team**: solo 4 personas, únicas con altos privilegios en el entorno
> 3. El equipo de IT Ops **suele usar la cuenta de administrador default** aunque se les indique lo contrario (mala práctica conocida)
> 4. Endpoints hardened según **CIS baselines**; whitelisting limitado
> 5. Existe un **PAW (Privileged Admin Workstation)** — todas las actividades admin **deben** realizarse ahí
> 6. Entorno Linux = servidores "leftover" con poca actividad regular. **root bloqueado para conexión remota** (hallazgo de auditoría) — quien necesite privilegios debe escalar vía `sudo`
> 7. **Naming conventions estrictas**: service accounts contienen `-svc` en el nombre, passwords largas/complejas, tarea muy específica (normalmente servicios locales)

> [!TIP]
> **Por qué este contexto importa tanto**
> Este es el corazón del **triaging real**: la misma alerta técnica puede ser "nada sospechoso" o "escalar inmediatamente" dependiendo de si viola o no las políticas/patrones específicos de **esta** organización. Sin este contexto, cualquier análisis sería genérico e impreciso.

## 📊 Revisión de las 7 visualizaciones del dashboard SOC-Alerts

### Visualización 1: Failed logon attempts (All users)

> [!NOTE]
> **Qué revelaría normalmente**
> Posibles ataques de brute force: un usuario con muchos intentos fallidos, o varios usuarios conectándose desde/hacia el mismo endpoint.

> [!TIP]
> **Pista del módulo**
> Los datos actuales **no apuntan** a ese escenario clásico — pero hay una anomalía relacionada con la cuenta **`sql-svc1`**.

> [!WARNING]
> **Framework de decisión a aplicar**
> Recordar la regla del entorno: las **service accounts** (`-svc`) tienen una tarea muy específica. Si `sql-svc1` (una service account) aparece con **múltiples fallos de login**, esto contradice su patrón esperado de uso (silencioso, automatizado, sin fallos frecuentes) → esto por sí solo ya amerita más que "nothing suspicious".

### Visualización 2: Failed logon attempts (Disabled user)

> [!NOTE]
> **Hallazgo**
> Un incidente donde el usuario **"Anni"** intentó autenticarse pese a tener la cuenta deshabilitada.

> [!TIP]
> **Cómo evaluarlo**
> Un solo intento aislado contra una cuenta deshabilitada puede ser: el propio usuario que no sabe que fue deshabilitado, IT tratando de reactivarla, o un intento malicioso con credenciales filtradas. **No hay contexto en las notas del entorno** que indique que esto es esperado → requiere más información antes de decidir severidad.

### Visualización 3: Failed logon attempts (Admin users only)

> [!TIP]
> **Pista del módulo**
> Verificar si **todos** los eventos ocurrieron en **PAWs** o **Domain Controllers**.

> [!WARNING]
> **Por qué esto es la clave de la decisión**
> Regla del entorno: las actividades admin **deben** ocurrir en el PAW. Si los intentos de login de cuentas admin están confinados a PAWs/DCs (lugares esperados), es coherente con la política — más benigno. Si aparecen en **otras máquinas**, viola la política de PAW-only.

### Visualización 4: RDP logon for service account

> [!WARNING]
> **Regla de negocio (vista en Sección 8)**
> Las service accounts **nunca** deberían usarse para RDP — tienen una función muy específica y automatizada.

> [!TIP]
> **Pregunta guía**
> ¿Se observa algo que amerite sospecha? → Cualquier RDP exitoso de una service account **contradice directamente** su propósito declarado — esto es una violación clara del patrón esperado.

### Visualización 5: User added or removed from a local group

> [!NOTE]
> **Hallazgo**
> Un administrador agregó un individuo (representado solo por su **SID**) al grupo **"Administrators"**.

> [!TIP]
> **Pregunta clave del módulo**
> ¿Escalar a Tier 2/3, o consultar primero con IT Operations?

> [!WARNING]
> **Framework de decisión**
> Recordar: solo **4 personas** de IT Ops tienen altos privilegios. Un SID desconocido (no resuelto a un nombre familiar) agregado a Administrators es exactamente el tipo de evento que requiere **verificar con IT Ops primero** — podría ser un cambio legítimo no documentado, o podría ser una escalada de privilegios no autorizada. La ambigüedad del SID (no mapeado a un nombre conocido del equipo de 4 personas) es la señal a investigar.

### Visualización 6: Admin logon not from PAW

> [!WARNING]
> **Violación directa de política**
> Recordar: "todas las actividades admin deben realizarse en el PAW". Un logon admin que **no** proviene del PAW contradice esta política explícita.

> [!TIP]
> **Pregunta clave**
> ¿Escalar directamente, o consultar con IT Ops primero?

> [!NOTE]
> **Contexto relevante**
> El módulo menciona que IT Ops **suele usar la cuenta default de administrador** incluso cuando se les indica lo contrario — esto sugiere que violaciones de política por parte del propio equipo de IT Ops **no son inusuales por mal hábito**, no necesariamente por compromiso. Esto empuja la decisión hacia **consultar con IT Ops primero** en vez de escalar directo, dado el patrón de comportamiento ya conocido del equipo.

### Visualización 7: SSH Logins

> [!WARNING]
> **Regla crítica del entorno Linux**
> El usuario **root está bloqueado** para conexión remota (por hallazgo de auditoría) — cualquiera que necesite privilegios debe usar `sudo`.

> [!TIP]
> **Qué buscar**
> Cualquier login SSH exitoso **como root** contradice directamente esta política de seguridad ya implementada — sería una señal fuerte de bypass de control o compromiso.

## 🧠 Framework general aplicado en este assessment

> [!NOTE]
> **Patrón de decisión (aplicando el proceso de Triaging de la Sección 10)**
> Para cada visualización, el criterio se reduce a:
> 1. **¿Existe una política/patrón conocido del entorno que esta actividad viola?** (Contextual Analysis)
> 2. **¿La violación es ambigua o tiene explicación plausible conocida?** → Consultar con IT Operations primero
> 3. **¿La violación es clara, sin explicación plausible, o involucra activos/cuentas de alto privilegio de forma inequívoca?** → Escalar a Tier 2/3
> 4. **¿El patrón es exactamente el esperado/documentado como normal?** → Nothing suspicious

> [!WARNING]
> No se incluyen las respuestas exactas (Nothing suspicious / Consult with IT Operations / Escalate) para cada visualización — dependen de los datos reales devueltos en cada dashboard del spawn específico. El objetivo de este resumen es documentar el **razonamiento** aplicable a cada caso, no anticipar la clasificación final del examen.

## 🏁 Cierre del módulo

Con esta sección se completa **Security Monitoring & SIEM Fundamentals** (11/11). Recorrido completo:
1. SIEM: definición, evolución, casos de uso de negocio, flujo de datos
2. Elastic Stack: Elasticsearch, Logstash, Kibana, Beats, KQL, ECS
3. SOC: definición, roles, tiers, evolución (1.0 → 2.0 → Cognitive)
4. MITRE ATT&CK en operaciones de seguridad
5. SIEM Use Case Development (ciclo de vida completo + ejemplos MSBuild)
6-9. Construcción práctica de 4 visualizaciones SIEM en Kibana
10. Proceso de Alert Triaging (11 etapas)
11. Skills Assessment: aplicación integrada de triaging contextual

## 🔗 Relacionado
- [The Triaging Process](10-the-triaging-process.md)
- [SIEM Use Case Development](05-siem-use-case-development.md)

#cjca #modulo19 #skills-assessment #alert-triaging #soc-analyst #kibana #contextual-analysis #modulo-completo
