# Metasploitable 2 – Multi-Vector Exploitation & Privilege Escalation

## 1. Resumen Ejecutivo

La máquina Metasploitable 2 expone múltiples servicios vulnerables, permitiendo diferentes vectores de ataque que conducen al compromiso completo del sistema.

Durante el análisis se identificaron vulnerabilidades críticas en servicios como FTP, SMB, MySQL, VNC y aplicaciones web, permitiendo obtener acceso inicial mediante explotación directa, credenciales débiles y vulnerabilidades conocidas.

El sistema fue completamente comprometido mediante múltiples rutas de ataque.

---

## 2. Alcance

- Objetivo: Metasploitable 2
- Tipo de prueba: Evaluación de servicios expuestos
- Objetivo: Identificar múltiples vectores de ataque y comprometer el sistema

---

## 3. Enumeración

Se realizó un escaneo completo de puertos:

```bash
nmap -p- -sS --min-rate 5000 -T4 -n -Pn <IP>
```

Servicios relevantes detectados:

- 21/tcp → FTP (vsftpd 2.3.4)
- 22/tcp → SSH
- 80/tcp → HTTP (DVWA)
- 445/tcp → SMB
- 3306/tcp → MySQL
- 5900/tcp → VNC

---

## 4. Superficie de Ataque

La máquina presenta múltiples vectores:

- Servicios expuestos con versiones vulnerables
- Credenciales débiles o por defecto
- Servicios accesibles sin autenticación
- Aplicaciones web vulnerables (SQLi)
- Configuración insegura de servicios

---

## 5. Vulnerabilidades Identificadas

### 5.1 FTP – vsftpd 2.3.4 (Backdoor)

- Vulnerabilidad conocida con backdoor
- Permite ejecución remota de comandos

Exploit:

```bash
searchsploit vsftpd 2.3.4
```

Resultado:

- Shell remota obtenida en puerto 6200

---

### 5.2 SSH – Credenciales débiles

- No se detectaron vulnerabilidades directas
- Acceso obtenido mediante fuerza bruta

Herramientas:

```bash
hydra
nmap --script ssh-brute
```

Resultado:

- Acceso válido mediante credenciales débiles

---

### 5.3 SMB – Exposición de recursos

- Posible acceso anónimo
- Enumeración de usuarios y recursos

Herramientas:

```bash
smbclient -L //<IP> -N
enum4linux -a <IP>
```

Riesgos:

- Exposición de información
- Posible obtención de credenciales

---

### 5.4 MySQL + Aplicación Web (DVWA)

- Base de datos accesible
- Vulnerabilidad de SQL Injection

Ejemplo:

```bash
' OR 1=1#
```

Automatización:

```bash
sqlmap -r request.txt --dump
```

Resultado:

- Obtención de usuarios y hashes
- Acceso a panel administrativo
- Subida de reverse shell

---

### 5.5 VNC – Contraseña débil

- Servicio expuesto sin protección
- Sin limitación de intentos

Fuerza bruta:

```bash
hydra -s 5900 -P wordlist.txt <IP> vnc
```

Resultado:

- Acceso completo al sistema gráfico

---

## 6. Explotación

Se lograron múltiples accesos mediante:

- Exploit directo (FTP backdoor)
- Fuerza bruta (SSH, VNC)
- SQL Injection + web shell
- Enumeración de servicios

---

## 7. Acceso Inicial

Se obtuvieron accesos mediante:

- Shell remota (FTP)
- Acceso SSH
- Web shell desde aplicación vulnerable
- Acceso VNC

---

## 8. Escalada de Privilegios

Dependiendo del vector:

- Uso de credenciales encontradas
- Configuraciones inseguras del sistema
- Acceso directo en entorno vulnerable

---

## 9. Impacto

- Compromiso total del sistema
- Ejecución remota de código
- Acceso a credenciales
- Acceso persistente
- Posibilidad de movimiento lateral

Impacto: Crítico

---

## 10. Recomendaciones

- Actualizar servicios vulnerables (FTP, SMB)
- Deshabilitar servicios innecesarios
- Implementar políticas de contraseñas robustas
- Restringir acceso a servicios críticos
- Validar entradas en aplicaciones web
- Implementar controles de acceso

---

## 11. Detección y Defensa (Blue Team Perspective)

Este laboratorio es especialmente útil para detección SOC.

Indicadores de compromiso:

- Conexiones a puerto 6200 (backdoor FTP)
- Intentos de login repetidos (SSH, VNC)
- Acceso a recursos SMB sin autenticación
- Consultas SQL anómalas (SQL Injection)
- Subida de archivos sospechosos (web shell)
- Conexiones salientes desde el servidor (reverse shell)

Fuentes de logs:

- auth.log (SSH)
- logs FTP
- logs web (Apache)
- logs MySQL
- tráfico de red

Medidas de detección:

- Alertas por múltiples intentos de login
- Detección de patrones de SQL Injection
- Monitorización de procesos sospechosos
- Correlación de eventos en SIEM (Splunk)
- Detección de conexiones anómalas

---

## 12. Conclusión

Metasploitable 2 demuestra un escenario realista donde múltiples configuraciones inseguras permiten el compromiso total del sistema.

La clave del ataque no fue una única vulnerabilidad, sino:

- Enumeración sistemática
- Identificación de múltiples vectores
- Explotación combinada
- Uso de credenciales débiles

Este tipo de entorno refleja situaciones reales en sistemas mal configurados.
