# Informe de Seguridad - Evaluación de Host "Knife"

## 1. Resumen Ejecutivo

Se ha realizado una evaluación de seguridad sobre el host "Knife", identificado como sistema Linux con servicios expuestos en red.

Durante el análisis se detectó una vulnerabilidad crítica en una versión de desarrollo de PHP (8.1.0-dev) que contiene una puerta trasera (backdoor), permitiendo ejecución remota de código (RCE) sin autenticación.

Tras obtener acceso inicial, se identificó una mala configuración en los permisos sudo que permitió ejecutar el binario `knife` como root sin contraseña, logrando una escalada completa de privilegios.

### Impacto

- Ejecución remota de código sin autenticación
- Acceso al sistema como usuario válido
- Escalada de privilegios a root
- Compromiso total del sistema

### Nivel de riesgo: **CRÍTICO**

---

## 2. Alcance

- Objetivo: Máquina "Knife"
- Sistema: Linux (Ubuntu)
- Tipo de prueba: Black Box
- Metodología: Enumeración → Explotación → Escalada

---

## 3. Enumeración

### 3.1 Escaneo de puertos

```bash
nmap -p- -sS --min-rate 5000 -Pn <IP>
