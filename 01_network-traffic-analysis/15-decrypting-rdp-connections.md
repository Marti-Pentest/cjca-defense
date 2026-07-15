# Módulo 16 — Intro to Network Traffic Analysis

## Sección 15/15 (Final): Decrypting RDP Connections

## 🎯 Escenario

> [!note] Caso — continuación del incidente de Bob
> Durante el IR, se capturó PCAP del tráfico RDP desde el host de Bob hacia otro host. Se encontró una **RDP-key oculta en un folder hive** del host de Bob. Esa clave permite **desencriptar** el tráfico RDP para inspeccionarlo.

## 🔒 Por qué RDP aparece "vacío" al filtrar

> [!warning] RDP usa TLS por defecto
> Al filtrar `rdp` sin la clave, no se ve casi nada — porque RDP cifra los datos con **TLS** por defecto. Para confirmar que existió una sesión (aunque cifrada), hay que filtrar por el puerto conocido:

```
tcp.port == 3389
```
> [!tip] Qué confirma esto
> Que se estableció una **sesión TCP** entre los dos hosts en el puerto 3389 — aunque el contenido siga cifrado.

## 🔑 Proceso para importar la clave RSA y desencriptar

```
1. Edit → Preferences → Protocols → TLS
2. Click "Edit" en RSA keys list → se abre nueva ventana
3. Click "+" para agregar una nueva clave
4. Completar:
   - IP address: (IP del servidor RDP)
   - Port: 3389
   - Protocol: tpkt (o en blanco)
   - Key File: ruta a server.key
5. Guardar y refrescar el pcap
6. Volver a filtrar por: rdp
```

> [!note] Resultado
> Ahora el filtro `rdp` sí muestra tráfico — distintos tipos de **PDU** (Protocol Data Units) y acciones dentro de la sesión RDP, ya desencriptada.

> [!tip] Cómo se obtuvo la clave (contexto técnico)
> Si se obtiene el **certificado RDP** del servidor, se puede usar **OpenSSL** para extraer la clave privada de ese certificado. El proceso completo de extracción es largo, pero el concepto central es: certificado del servidor → OpenSSL → clave privada → Wireshark puede desencriptar.

## 🔍 Una vez desencriptado — qué se puede hacer

> [!note] Capacidades disponibles con RDP en claro
> - Seguir flujos TCP (Follow TCP Stream)
> - Exportar objetos encontrados
> - Cualquier otro análisis necesario para la investigación

## 🧠 Preguntas del lab (requieren el pcap real — checklist)

> [!faq]- ¿Qué host inició la sesión RDP con el servidor?
> Se determina mirando quién envía el primer paquete de conexión (SYN) hacia el puerto 3389 del servidor — el **cliente** es quien inicia.

> [!faq]- ¿Qué cuenta de usuario se usó para iniciar la conexión RDP?
> Una vez desencriptado el tráfico, el nombre de usuario aparece en los PDUs de negociación/logon de RDP (visible en el panel de detalles del paquete tras aplicar la clave).

## 💡 Concepto clave para generalizar

> [!tip] Aplicable a cualquier protocolo cifrado
> Esta técnica **no es exclusiva de RDP**. El mismo principio aplica a **cualquier protocolo que use TLS/SSL** (HTTPS, etc.): si se tiene la clave privada usada para establecer la conexión, Wireshark puede desencriptar y mostrar el tráfico en claro. Esto es extremadamente útil en:
> - Investigaciones forenses donde se tiene acceso legítimo a las claves del servidor
> - Debugging de aplicaciones cifradas en entornos de laboratorio/desarrollo

## 🏁 Cierre del módulo

Con esta sección se completa **Intro to Network Traffic Analysis** (15/15). El recorrido completo cubrió:
1. Fundamentos de NTA y flujo de trabajo de análisis
2. Redes (capas 1-7, protocolos comunes)
3. Proceso de análisis (descriptivo → diagnóstico → predictivo → prescriptivo)
4. Tcpdump (fundamentos, filtros, laboratorios prácticos)
5. Wireshark (GUI, filtros, plugins avanzados, extracción de archivos, desencriptación)

## 🔗 Relacionado
- [[14-guided-lab-traffic-analysis-workflow]]
- [[../README]]
- [[TLS Decryption - Notas]]

#cjca #modulo16 #wireshark #rdp #tls-decryption #laboratorio #forense #modulo-completo
