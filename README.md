# CJCA Defense — Notas de Estudio

Notas personales de estudio del path **CJCA (Certified Junior Cybersecurity Analyst)** de HTB Academy, enfocado en los módulos de **Blue Team / Defensa**.

Vengo del lado ofensivo (ya tengo la **CPTS**), así que estas notas están escritas desde esa perspectiva: entender cómo piensa un atacante ayuda muchísimo a entender qué hay que detectar.

## 📚 Roadmap del path CJCA (módulos cubiertos aquí)

| # | Módulo | Estado |
|---|--------|--------|
| 16 | Intro to Network Traffic Analysis | ✅ Completo |
| 17 | Incident Handling Process | ⏳ Pendiente (se publicará cuando esté completo) |
| 18 | Windows Event Logs & Finding Evil | ✅ Completo |
| 19 | Security Monitoring & SIEM Fundamentals | ✅ Completo |
| 20 | Introduction to Threat Hunting & Hunting With Elastic | 🟡 Casi completo (falta Sección 4) |

> [!NOTE]
> El Módulo 17 no está publicado todavía (solo tenía 2/11 secciones documentadas) — se subirá cuando esté completo. Por eso la numeración de carpetas salta del 01 al 03.

## 🗂️ Estructura del repo

```
cjca-defense/
├── README.md
├── 01-network-traffic-analysis/       (15 secciones)
├── 03-windows-event-logs/             (6 secciones)
├── 04-security-monitoring-siem/       (11 secciones)
└── 05-threat-hunting/                 (5/6 secciones)
```

## 📝 Formato de las notas

Cada archivo sigue una estructura pensada para repaso rápido, usando **GitHub Flavored Markdown**:

- **Alerts** (`> [!NOTE]`, `> [!TIP]`, `> [!WARNING]`) para conceptos clave, tips y advertencias
- **Bloques de código** con todos los comandos/filtros/sintaxis relevante
- **Tablas** para comparativas (protocolos, switches, filtros, Event IDs)
- **Quiz de repaso** en bloques `<details>` plegables al final de cada sección
- **Tags** al final de cada nota para conectar temas relacionados
- **Diagramas Mermaid** para flujos y arquitecturas (kill chain, ciclo de vida IR, ETW, SIEM data flow, cadenas de ataque)

## 📖 Índice — Módulo 16: Intro to Network Traffic Analysis

1. [Análisis de Tráfico de Red](01-network-traffic-analysis/01-intro-nta.md)
2. [Networking Primer — Capas 1-4](01-network-traffic-analysis/02-networking-primer-l1-4.md)
3. [Networking Primer — Capas 5-7](01-network-traffic-analysis/03-networking-primer-l5-7.md)
4. [El Proceso de Análisis](01-network-traffic-analysis/04-analysis-process.md)
5. [Análisis en la Práctica](01-network-traffic-analysis/05-analysis-practice.md)
6. [Fundamentos de Tcpdump](01-network-traffic-analysis/06-tcpdump-fundamentals.md)
7. [Lab: Capturing With Tcpdump](01-network-traffic-analysis/07-tcpdump-lab-fundamentals.md)
8. [Filtrado de Paquetes con Tcpdump](01-network-traffic-analysis/08-tcpdump-packet-filtering.md)
9. [Lab: Interrogating Network Traffic](01-network-traffic-analysis/09-tcpdump-lab-interrogating-traffic.md)
10. [Análisis con Wireshark](01-network-traffic-analysis/10-wireshark-analysis.md)
11. [Lab: Familiarity With Wireshark](01-network-traffic-analysis/11-wireshark-familiarity-lab.md)
12. [Uso Avanzado de Wireshark](01-network-traffic-analysis/12-wireshark-advanced-usage.md)
13. [Lab: Packet Inception](01-network-traffic-analysis/13-wireshark-packet-inception-lab.md)
14. [Guided Lab: Traffic Analysis Workflow](01-network-traffic-analysis/14-guided-lab-traffic-analysis-workflow.md)
15. [Decrypting RDP Connections](01-network-traffic-analysis/15-decrypting-rdp-connections.md)

## 📖 Índice — Módulo 18: Windows Event Logs & Finding Evil

1. [Windows Event Logs](03-windows-event-logs/01-windows-event-logs.md)
2. [Analyzing Evil With Sysmon & Event Logs](03-windows-event-logs/02-analyzing-evil-sysmon-event-logs.md)
3. [Event Tracing for Windows (ETW)](03-windows-event-logs/03-etw-fundamentals.md)
4. [Tapping Into ETW](03-windows-event-logs/04-tapping-into-etw.md)
5. [Get-WinEvent](03-windows-event-logs/05-get-winevent.md)
6. [Skills Assessment](03-windows-event-logs/06-skills-assessment.md)

## 📖 Índice — Módulo 19: Security Monitoring & SIEM Fundamentals

1. [SIEM Definition & Fundamentals](04-security-monitoring-siem/01-siem-definition-fundamentals.md)
2. [Introduction To The Elastic Stack](04-security-monitoring-siem/02-introduccion-elastic-stack.md)
3. [SOC Definition & Fundamentals](04-security-monitoring-siem/03-soc-definition-fundamentals.md)
4. [MITRE ATT&CK & Security Operations](04-security-monitoring-siem/04-mitre-attack-security-operations.md)
5. [SIEM Use Case Development](04-security-monitoring-siem/05-siem-use-case-development.md)
6. [SIEM Visualization Example 1: Failed Logon Attempts (All Users)](04-security-monitoring-siem/06-siem-visualization-example-1-all-users.md)
7. [SIEM Visualization Example 2: Failed Logon Attempts (Disabled Users)](04-security-monitoring-siem/07-siem-visualization-example-2-disabled-users.md)
8. [SIEM Visualization Example 3: Successful RDP Logon (Service Accounts)](04-security-monitoring-siem/08-siem-visualization-example-3-rdp-service-accounts.md)
9. [SIEM Visualization Example 4: Group Membership Changes](04-security-monitoring-siem/09-siem-visualization-example-4-group-membership-changes.md)
10. [The Triaging Process](04-security-monitoring-siem/10-the-triaging-process.md)
11. [Skills Assessment](04-security-monitoring-siem/11-skills-assessment.md)

## 📖 Índice — Módulo 20: Introduction to Threat Hunting & Hunting With Elastic *(falta Sección 4)*

1. [Threat Hunting Fundamentals](05-threat-hunting/01-threat-hunting-fundamentals.md)
2. [The Threat Hunting Process](05-threat-hunting/02-the-threat-hunting-process.md)
3. [Threat Hunting Glossary](05-threat-hunting/03-threat-hunting-glossary.md)
4. *(pendiente)*
5. [Hunting For Stuxbot](05-threat-hunting/05-hunting-for-stuxbot.md)
6. [Skills Assessment: Hunting For Stuxbot Round 2](05-threat-hunting/06-skills-assessment-stuxbot-round-2.md)

## ⚠️ Disclaimer

Estas son **notas de estudio personales**, escritas como resúmenes y reinterpretaciones propias del contenido del curso — no son una transcripción ni copia del material original de HTB Academy.

- No estoy afiliado a Hack The Box ni este repo está patrocinado o respaldado por ellos.
- **No se incluyen respuestas exactas de labs, flags, ni walkthroughs completos de ejercicios pagos** — donde aplica, se documenta la *metodología* y los *comandos/filtros* usados, no la respuesta literal.
- Todo el contenido de laboratorios prácticos (IPs, hosts, escenarios) proviene de mis propias sesiones de práctica en el entorno de HTB Academy.

Si sos parte de HTB y considerás que algo acá viola tus términos de servicio, abrí un issue y lo reviso/ajusto sin drama.

## 🎯 Sobre mí

Pentester con **CPTS** en curso hacia el lado defensivo. Este repo es parte de mi proceso hacia la certificación **CJCA**.

---

⭐ Si te sirve para tu propio estudio del path CJCA, dejá una estrella.
