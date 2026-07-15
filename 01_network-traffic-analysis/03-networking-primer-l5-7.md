# Módulo 16 — Intro to Network Traffic Analysis

## Sección 3/15: Manual básico de redes: Capas 5-7

## 🌐 HTTP
> [!note] Definición
> Protocolo de capa de aplicación **stateless**, transfiere datos en **texto plano** entre cliente-servidor sobre TCP. En uso desde 1990.

- **Puertos**: 80 u 8000 (TCP) — en casos excepcionales, otros puertos o incluso UDP
- Flujo: cliente solicita un recurso → servidor establece sesión → responde con el contenido (HTML, imágenes, etc.)

### Métodos HTTP

| Método | Requerido/Opcional | Descripción |
|---|---|---|
| **HEAD** | Requerido | Como GET pero sin cuerpo de respuesta — útil para info del servidor |
| **GET** | Requerido | Solicita info/contenido. Ej: `GET http://10.1.1.1/Webserver/index.html` |
| **POST** | Opcional | Envía datos (crea entidades *hijas* en la URI, ej: comentario en un post) |
| **PUT** | Opcional | Crea/actualiza el objeto **en** la URI dada |
| **DELETE** | Opcional | Elimina el objeto en la URI |
| **TRACE** | Opcional | Diagnóstico — servidor hace eco de la solicitud recibida |
| **OPTIONS** | Opcional | Lista métodos soportados por el servidor |
| **CONNECT** | Opcional | Tunneling sobre HTTP (túneles SSL), usado por proxies/firewalls |

> [!tip] PUT vs POST
> PUT = crear/actualizar el archivo en sí. POST = crear un comentario/entidad relacionada con ese archivo.

> [!warning] Únicos obligatorios por estándar
> Solo **GET** y **HEAD** deben funcionar siempre en cualquier implementación HTTP estándar.

## 🔒 HTTPS
> [!note] Definición
> HTTP + **TLS/SSL** para cifrar toda la conversación (no solo los datos).

- **Puertos**: 443 y 8443 (en vez de 80)
- Antes de TLS: vulnerable a MitM y session hijacking en la misma LAN

### Resumen del handshake TLS
```
1. Cliente/Servidor intercambian "hello" → acuerdan parámetros de conexión
2. Intercambian parámetros criptográficos → generan "premaster secret"
3. Intercambian certificados x.509 → autenticación
4. Se genera "master secret" a partir del premaster + valores aleatorios
5. Ambos emiten parámetros de seguridad a la capa de registro TLS
6. Ambos verifican que el handshake no fue manipulado
```

> [!note] En captura de tráfico
> Tras el `SYN`/`SYN,ACK`/`ACK` inicial (TCP), se envía **ClientHello** de TLS. Una vez establecida la sesión, todo el tráfico aparece como **TLS Application Data** (aunque los ACKs de TCP siguen siendo visibles).

## 📁 FTP
> [!note] Definición
> Protocolo de transferencia de archivos, considerado **inseguro** (texto plano). Reemplazado en muchos casos por SFTP. Navegadores modernos eliminaron soporte desde 2020.

- **Puertos**: 20 (datos) y 21 (comandos) — TCP
- Autenticación de usuario o acceso anónimo

### Modos de operación
| Modo | Descripción |
|---|---|
| **Activo** (default) | Servidor escucha comando `PORT` del cliente indicando qué puerto usar |
| **Pasivo** | Cliente envía `PASV`, servidor responde con IP/puerto a usar (útil detrás de NAT/firewall) |

### Comandos FTP comunes
```
USER   - especifica usuario para login
PASS   - envía contraseña
PORT   - cambia puerto de datos (modo activo)
PASV   - cambia de modo activo a pasivo
LIST   - lista archivos del directorio actual
CWD    - cambia directorio de trabajo
PWD    - imprime directorio actual
SIZE   - devuelve tamaño de un archivo
RETR   - recupera un archivo del servidor
QUIT   - finaliza la sesión
```

## 🗂️ SMB
> [!note] Definición
> Protocolo orientado a conexión para compartir recursos (impresoras, unidades, autenticación) en entornos Windows/empresariales. Requiere autenticación de usuario.

- **Legado**: NetBIOS sobre UDP 137/138
- **Moderno**: TCP directo puerto **445**, NetBIOS sobre TCP puerto 139, o QUIC

> [!warning] Objetivo atractivo para atacantes
> SMB es muy usado en movimiento lateral. **Múltiples fallos de autenticación repetidos** (más allá de 1-2 normales) pueden indicar:
> - Intento de acceso no autorizado / password spraying
> - Robo y uso de credenciales para moverse lateralmente
>
> También vigilar: hosts accediendo a shares de otros hosts de forma inusual — prestar atención a **quién** solicita, **hacia dónde**, y **qué** hace.

## 🧠 Quiz de repaso (Q&A del módulo)

> [!faq]- ¿Cuál es el modo operativo por defecto de FTP?
> **Activo**

> [!faq]- ¿Qué dos puertos usa FTP para comandos y transferencia de datos?
> **21** (comandos) y **20** (datos)

> [!faq]- ¿SMB usa TCP o UDP como protocolo de transporte?
> **TCP** (modernamente; históricamente también UDP vía NetBIOS)

> [!faq]- ¿A qué puerto TCP se ha movido SMB?
> **445**

> [!faq]- ¿Qué puerto TCP bien conocido usa HTTP?
> **80**

> [!faq]- ¿Qué método HTTP se usa para solicitar información/contenido del servidor?
> **GET**

> [!faq]- ¿Qué protocolo web usa TLS como medida de seguridad?
> **HTTPS**

> [!faq]- ¿Verdadero o Falso: en HTTPS, todos los datos enviados aparecen como TLS Application Data?
> **Verdadero**

## 🔗 Relacionado
- [[02-networking-primer-l1-4]]
- [[04-analysis-process]]
- [[Cheatsheet Puertos Comunes]]

#cjca #modulo16 #http #https #tls #ftp #smb #capa-aplicacion
