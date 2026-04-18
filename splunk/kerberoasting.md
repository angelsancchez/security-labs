## Use Case: Kerberoasting

### Query
index=wineventlog EventCode=4769 
| stats count by Account_Name, Service_Name 
| where count > 10

### Qué detecta
- Solicitudes masivas de tickets Kerberos

### Por qué importa
- Posible extracción de hashes de servicio

### Relación con lab
- Active / Forest
