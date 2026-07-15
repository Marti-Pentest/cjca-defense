# CJCA Defense — Notas de Estudio

Notas personales de estudio del path **CJCA (Certified Junior Cybersecurity Analyst)** de HTB Academy, enfocado en los módulos de **Blue Team / Defensa**.

Vengo del lado ofensivo (ya tengo la **CPTS**), así que estas notas están escritas desde esa perspectiva: entender cómo piensa un atacante ayuda muchísimo a entender qué hay que detectar.

## 📚 Roadmap del path CJCA (módulos cubiertos aquí)

| # | Módulo | Estado |
|---|--------|--------|
| 16 | Intro to Network Traffic Analysis | ✅ Completo |
| 17 | Incident Handling Process | ⏳ Pendiente |
| 18 | Windows Event Logs & Finding Evil | ⏳ Pendiente |
| 19 | Security Monitoring & SIEM Fundamentals | ⏳ Pendiente |
| 20 | Introduction to Threat Hunting | ⏳ Pendiente |

## 🗂️ Estructura del repo

```
cjca-defense/
├── README.md
├── 01-network-traffic-analysis/
│   ├── 01-intro-nta.md
│   ├── 02-networking-primer-l1-4.md
│   ├── 03-networking-primer-l5-7.md
│   ├── 04-analysis-process.md
│   ├── 05-analysis-practice.md
│   ├── 06-tcpdump-fundamentals.md
│   ├── 07-tcpdump-lab-fundamentals.md
│   ├── 08-tcpdump-packet-filtering.md
│   ├── 09-tcpdump-lab-interrogating-traffic.md
│   ├── 10-wireshark-analysis.md
│   ├── 11-wireshark-familiarity-lab.md
│   ├── 12-wireshark-advanced-usage.md
│   ├── 13-wireshark-packet-inception-lab.md
│   ├── 14-guided-lab-traffic-analysis-workflow.md
│   └── 15-decrypting-rdp-connections.md
├── 02-incident-handling-process/      (próximamente)
├── 03-windows-event-logs/             (próximamente)
├── 04-security-monitoring-siem/       (próximamente)
└── 05-threat-hunting/                 (próximamente)
```

## 📝 Formato de las notas

Cada archivo sigue una estructura pensada para repaso rápido tipo Obsidian:

- **Callouts** (`> [!note]`, `> [!tip]`, `> [!warning]`) para conceptos clave, tips y advertencias
- **Bloques de código** con todos los comandos/filtros/sintaxis relevante
- **Tablas** para comparativas (protocolos, switches, filtros)
- **Quiz de repaso** en formato pregunta/respuesta plegable al final de cada sección
- **Tags** al estilo Obsidian para conectar temas relacionados

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
