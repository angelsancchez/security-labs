# Use Case: Detección de Reverse Shell en sistemas Linux/Windows

---

## 1. Contexto del ataque

Una **reverse shell** permite a un atacante obtener acceso remoto a un sistema comprometido mediante una conexión saliente iniciada desde la víctima hacia el atacante.

Este comportamiento es común tras:

* Explotación de vulnerabilidades web (RCE)
* Ejecución de payloads maliciosos
* Post-explotación en entornos comprometidos

---

## 2. Objetivo de detección

Detectar:

* Procesos sospechosos (bash, sh, nc, powershell)
* Que generan conexiones salientes
* Hacia IPs externas o no habituales

---

## 3. Fuentes de logs

* Sysmon (Windows)
* Auditd / Syslog (Linux)
* Logs de red (Netflow, firewall)

Eventos clave:

* Creación de proceso
* Conexión de red saliente

---

## 4. Query en Splunk (correlación proceso + red)

```spl id="rsq1"
(index=sysmon EventCode=1 OR EventCode=3)
| eval is_suspicious_process=if(match(process_name,"(bash|sh|nc|netcat|powershell)"),1,0)
| eval is_external_connection=if(dest_ip!="127.0.0.1" AND dest_ip!="::1",1,0)
| stats values(process_name) as processes
        values(dest_ip) as destinations
        values(dest_port) as ports
        count by host, user
| where is_suspicious_process=1 AND is_external_connection=1
```

---

## 5. Explicación de la query

1. Filtra eventos de:

   * Creación de proceso (EventCode=1)
   * Conexiones de red (EventCode=3)

2. Identifica procesos sospechosos:

   * bash
   * sh
   * netcat (nc)
   * powershell

3. Detecta conexiones externas

4. Correlaciona:

   * proceso + conexión

---

## 6. Qué detecta

* Reverse shells clásicas (`bash`, `nc`)
* PowerShell reverse shells
* Conexiones salientes no habituales
* Actividad post-explotación

---

## 7. Ejemplo de comportamiento malicioso

```text id="rsex1"
Proceso: bash
Usuario: www-data
Destino: 10.10.14.5
Puerto: 4444
```

👉 patrón típico tras explotación web

---

## 8. Diferenciación: normal vs malicioso

### Comportamiento legítimo

* Procesos locales sin conexión externa
* Conexiones a servidores internos conocidos

---

### Comportamiento sospechoso

* bash → conexión externa
* nc → puerto alto
* powershell → IP desconocida
* procesos ejecutados por servicios web

---

## 9. Posibles falsos positivos

* Scripts de administración remota
* Herramientas de automatización
* conexiones legítimas a APIs externas

👉 Clave: contexto + proceso + destino

---

## 10. Cómo investigar (playbook SOC)

1. Identificar proceso:

   * ¿quién lo lanzó? (usuario/servicio)

2. Revisar origen:

   * ¿proviene de Apache / IIS?

3. Analizar conexión:

   * IP destino
   * reputación

4. Correlacionar con:

   * eventos web previos (RCE)
   * creación de procesos

5. Determinar:

   * ¿hay persistencia?
   * ¿hay movimiento lateral?

---

## 11. Mitigación

* Restringir conexiones salientes
* Implementar EDR
* Monitorizar procesos críticos
* Limitar ejecución desde servicios web
* Aplicar principio de mínimo privilegio

---

## 12. MITRE ATT&CK

* T1059 – Command Execution
* T1071 – Application Layer Protocol
* T1046 – Network Service Scanning
* T1105 – Ingress Tool Transfer

---

## 13. Relación con labs

Este comportamiento aparece en:

* Cap
* Knife
* Drupal
* Nibbles
* Optimum

---

## 14. Conclusión

Las reverse shells son uno de los indicadores más claros de compromiso activo.

La correlación entre procesos sospechosos y conexiones de red permite detectar este comportamiento de forma efectiva.
