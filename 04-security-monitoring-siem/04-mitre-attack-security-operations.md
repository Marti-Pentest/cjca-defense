# Módulo 19 — Security Monitoring & SIEM Fundamentals

## Sección 4/11: MITRE ATT&CK & Security Operations

## 📌 ¿Qué es MITRE ATT&CK?

> [!NOTE]
> **Definición**
> **Adversarial Tactics, Techniques, and Common Knowledge** — recurso extenso y actualizado regularmente que documenta las **TTPs** (tactics, techniques, procedures) usadas por actores de amenaza.

> [!TIP]
> **Propósito**
> Ayudar a los profesionales de ciberseguridad a **comprender, identificar y reaccionar** ante amenazas de forma más proactiva e informada.

> [!NOTE]
> **Matrices por contexto**
> El framework incluye matrices adaptadas a distintos contextos de cómputo: **enterprise, mobile, cloud**, etc. Cada matriz vincula:
> - **Tácticas** = objetivos que el atacante busca lograr
> - **Técnicas** = métodos para lograr esos objetivos
>
> Esta vinculación permite a los equipos de seguridad examinar y **predecir metódicamente** las actividades del atacante.

> [!TIP]
> **Recordatorio (ya visto en Módulo 17)**
> Esto conecta directamente con lo cubierto en la sección de **Cyber Kill Chain**: mismo concepto de Tactic → Technique → Sub-technique, aplicado ahora específicamente al contexto de operaciones SOC.

## 🎯 Casos de uso de MITRE ATT&CK en Security Operations

| Caso de uso | Descripción |
|---|---|
| **Detection and Response** | Diseñar planes de detección/respuesta basados en TTPs reconocidas — permite contramedidas proactivas |
| **Security Evaluation & Gap Analysis** | Identificar fortalezas/debilidades de la postura de seguridad → priorizar inversión en controles |
| **SOC Maturity Assessment** | Medir la capacidad del SOC para detectar/responder/mitigar distintas TTPs → identificar áreas de mejora |
| **Threat Intelligence** | Lenguaje y formato unificado para describir acciones adversarias → mejor colaboración interna/externa |
| **Cyber Threat Intelligence Enrichment** | Contexto sobre TTPs del atacante, posibles objetivos e IOCs → decisiones más informadas |
| **Behavioral Analytics Development** | Mapear TTPs a comportamientos específicos de usuario/sistema → modelos de detección de anomalías |
| **Red Teaming and Penetration Testing** | Replicar técnicas reales de atacantes de forma sistemática → evaluar capacidades defensivas |
| **Training and Education** | Estructura organizada y comprehensiva → excelente recurso para entrenar profesionales de seguridad |

> [!NOTE]
> **Idea central**
> MITRE ATT&CK es un **lenguaje y estructura compartida** para describir/entender el comportamiento adversario. Es clave para mejorar múltiples aspectos de las operaciones de seguridad: desde threat intelligence y behavioral analytics, hasta SOC maturity assessment y CTI enrichment.

## 🔗 Relacionado
- [SOC Definition & Fundamentals](03-soc-definition-fundamentals.md)
- [SIEM Use Case Development](05-siem-use-case-development.md)
- [Modulo 17 - Cyber Kill Chain](../02-incident-handling-process/02-cyber-kill-chain.md)

#cjca #modulo19 #mitre-attack #ttps #soc-maturity #threat-intelligence #behavioral-analytics #red-teaming
