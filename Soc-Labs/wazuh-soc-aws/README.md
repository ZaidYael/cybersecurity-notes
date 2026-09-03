# 📡 AWS SOC Lab: Implementación de Wazuh SIEM/XDR y Detección de Amenazas

![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Wazuh](https://img.shields.io/badge/Wazuh-000000?style=for-the-badge&logo=wazuh&logoColor=00A4E4)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

## 📌 Resumen del Proyecto
Despliegue y configuración de un entorno completo de Centro de Operaciones de Seguridad (SOC) en la nube de AWS utilizando **Wazuh** como solución unificada de SIEM y XDR. El proyecto abarca desde la recolección masiva de eventos crudos (*Archives*) y enriquecimiento de telemetría con Sysmon, hasta el monitoreo de integridad (FIM), creación de reglas de detección personalizadas y mitigación automatizada mediante Active Response.

---

## 🗺️ Módulos de Trabajo (Reto de 7 Días)

| Día | Módulo Técnico | Enfoque Principal | Reporte Detallado |
| :---: | :--- | :--- | :---: |
| **01** | **Servidor Central & Ingesta de Logs** | Despliegue en EC2, Security Groups y activación de *Archives* para telemetría cruda. | [Ver Write-up](./Day-01-Server-Setup.md) |
| **02** | **Despliegue de Agentes & Sysmon** | Instalación de agentes en Windows/Linux y configuración del canal Sysmon/Operational. | ⏳ Próximamente |
| **03** | **Telemetría & Simulación Adversaria** | Ejecución de técnicas de reconocimiento, creación/borrado de cuentas y análisis de Event IDs. | ⏳ Próximamente |
| **04** | **Construcción de Dashboards SOC** | Paneles de métricas (4625), gráficos temporales y tablas de auditoría de conexiones SSH. | ⏳ Próximamente |
| **05** | **File Integrity Monitoring & Reglas XML** | Monitoreo en tiempo real de directorios sensibles y reglas personalizadas para cuentas *Guest*. | ⏳ Próximamente |
| **06** | **Active Response (SOAR)** | Automatización de mitigación: bloqueo perimetral en iptables ante ataques de fuerza bruta SSH. | ⏳ Próximamente |
| **07** | **Investigación Forense & Triaje L1** | Reconstrucción de la cadena de intrusión, análisis de IoCs e informe formal de incidente. | ⏳ Próximamente |

## 🔍 Eventos Extra, Triaje L1 y Detecciones No Planificadas

Registro de anomalías reales, falsos positivos y actividades operativas depuradas durante el funcionamiento continuo del SIEM:

| Fecha / Contexto | Tipo de Evento | Regla / Componente Afectado | Análisis y Acción de Triaje | Documentación |
| :---: | :--- | :--- | :--- | :---: |
| **Día 1** | **Falso Positivo:** Detección de binario modificado (`diff`) | `Rule ID: 510` (Rootcheck) | Verificación de integridad con `dpkg -V diffutils` y supresión mediante directiva `<ignore>` en `ossec.conf`. | [Ver Análisis](./Análisis-de-falso-positivo-rootcheck-rule-id-510) |
