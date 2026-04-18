## Use Case: Detección de fuerza bruta (Windows)

### Descripción
Detección de múltiples intentos fallidos de login.

### Query (Splunk)
index=wineventlog EventCode=4625 
| stats count by Account_Name, src_ip 
| where count > 5

### Qué detecta
- Intentos repetidos de autenticación fallida

### Por qué importa
- Indicador de ataque de fuerza bruta

### Cómo investigar
- Revisar IP origen
- Ver si hay login exitoso posterior (4624)
