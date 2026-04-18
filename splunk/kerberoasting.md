# Use Case: Detección de Kerberoasting en Active Directory

---

## 1. Contexto del ataque

El ataque de **Kerberoasting** consiste en solicitar tickets de servicio (TGS) para cuentas con SPN registrados y posteriormente crackearlos offline para obtener contraseñas.

Este ataque no requiere privilegios elevados y es común en fases iniciales de compromiso en entornos Active Directory.

---

## 2. Objetivo de detección

Detectar comportamiento anómalo de solicitudes Kerberos que pueda indicar intento de extracción masiva de tickets (TGS).

---

## 3. Fuente de logs

* Windows Security Logs
* Event ID: **4769 (Kerberos Service Ticket Request)**

---

## 4. Query en Splunk

```spl id="kerbq1"
index=wineventlog EventCode=4769
| stats count as total_requests by Account_Name, Service_Name, src_ip
| where total_requests > 10
| sort - total_requests
```

---

## 5. Explicación de la query

1. Filtra eventos de solicitud de tickets Kerberos (4769)
2. Agrupa por:

   * Usuario (Account_Name)
   * Servicio (Service_Name)
   * IP origen (src_ip)
3. Cuenta el número de solicitudes
4. Aplica umbral (>10) para detectar comportamiento anómalo

---

## 6. Qué detecta

* Solicitudes masivas de tickets Kerberos
* Enumeración de servicios con SPN
* Posible intento de Kerberoasting

---

## 7. Ejemplo de comportamiento malicioso

```text id="kerbex1"
Usuario: svc_account
IP origen: 10.10.10.50
Solicitudes: 45 tickets en pocos segundos
Servicios: MSSQLSvc, HTTP, CIFS
```

👉 Esto NO es comportamiento normal

---

## 8. Diferenciación: normal vs malicioso

### Comportamiento legítimo

* Pocas solicitudes
* Distribuidas en el tiempo
* Asociadas a servicios reales utilizados

---

### Comportamiento sospechoso

* Alto volumen en corto periodo
* Múltiples SPN diferentes
* Desde una misma IP
* Usuario sin patrón habitual de uso

---

## 9. Posibles falsos positivos

* Aplicaciones que usan múltiples servicios automáticamente
* Escaneos internos autorizados
* Scripts de administración

---

## 10. Cómo investigar (playbook SOC)

1. Identificar usuario afectado
2. Revisar IP origen
3. Correlacionar con:

   * Event ID 4624 (logon)
   * Event ID 4776 (NTLM auth)
4. Ver si hay:

   * acceso posterior a sistemas críticos
   * movimientos laterales

---

## 11. Mitigación

* Usar contraseñas fuertes en cuentas de servicio
* Implementar cuentas gestionadas (gMSA)
* Monitorizar solicitudes Kerberos
* Aplicar detección en SIEM

---

## 12. MITRE ATT&CK

* T1558.003 – Kerberoasting
* T1558 – Steal or Forge Kerberos Tickets

---

## 13. Relación con labs

Este comportamiento se observa en:

* Active
* Forest
* Sauna

---

## 14. Conclusión

Kerberoasting es un ataque silencioso pero altamente efectivo.

Una correcta monitorización de eventos 4769 permite detectar este comportamiento en fases tempranas antes de que el atacante escale privilegios.
