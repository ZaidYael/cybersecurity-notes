# 📊 [Write-up] Wazuh SOC Analyst Challenge — Día 4: Construcción del Dashboard SOC

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Wazuh](https://img.shields.io/badge/Wazuh-005571?style=for-the-badge&logo=wazuh&logoColor=white)
![Dificultad](https://img.shields.io/badge/Dificultad-Media-orange?style=for-the-badge)

## Tabla de Contenidos
- [1. Resumen Ejecutivo y Metadatos](#1-resumen-ejecutivo-y-metadatos)
- [2. Entorno de Trabajo y Herramientas](#2-entorno-de-trabajo-y-herramientas)
- [3. Introducción y Contexto Teórico](#3-introducción-y-contexto-teórico)
- [4. Metodología: Construcción del Dashboard](#4-metodología-construcción-del-dashboard)
- [5. Paneles Construidos](#5-paneles-construidos)
- [6. Detecciones y Event IDs Relevantes](#6-detecciones-y-event-ids-relevantes)
- [7. Conclusiones y Aprendizajes Clave](#7-conclusiones-y-aprendizajes-clave)
- [8. Referencias y Fuentes](#8-referencias-y-fuentes)

---

## 1. Resumen Ejecutivo y Metadatos
* **Fecha:** 20/09/2026
* **Plataforma:** Proyecto Personal / Wazuh SOC Analyst Challenge (Parte 4 de 7)
* **Categoría:** Blue Team / SIEM / Dashboard
* **Dificultad:** Media
* **Objetivo:** Construir un dashboard SOC de tres paneles en Wazuh que consolide telemetría de endpoints Windows y Linux, cubriendo fallos de autenticación, cambios de cuentas y actividad de comandos Sysmon.

---

## 2. Entorno de Trabajo y Herramientas

* **Sistema de Análisis:** Wazuh Dashboard (OpenSearch Dashboards)
* **Fuentes de Datos:** `wazuh-archives-*` (eventos de agentes Windows y Linux)
* **Herramientas utilizadas:**
  * **Wazuh Dashboard:** Visualización y construcción de paneles SIEM.
  * **Wazuh Discover:** Exploración de campos de eventos e identificación de field names clave.
  * **Endpoint Windows (agente Wazuh):** Generación de telemetría de autenticación y cambios de cuenta.
  * **Endpoint Linux con Sysmon (agente Wazuh):** Generación de telemetría de ejecución de comandos.

---

## 3. Introducción y Contexto Teórico

### ¿Qué es un Dashboard SOC en Wazuh?

Un dashboard SOC en Wazuh es una vista centralizada que agrega y visualiza en tiempo real la telemetría de múltiples agentes. Permite al analista detectar de un vistazo anomalías como picos de fallos de autenticación, creaciones masivas de cuentas o ejecución inusual de comandos, sin tener que realizar búsquedas manuales en cada fuente de logs.

### Conceptos Clave

| Concepto / Event ID | Descripción |
| :--- | :--- |
| `4625` | Failed Logon — intento de autenticación fallido en Windows. |
| `4720` | User Account Created — cuenta de usuario creada exitosamente. |
| `4722–4738` | Cambios de estado y atributos de cuentas de usuario Windows. |
| `4733` | Member Removed from Security-Enabled Group. |
| `data.win.system.eventid` | Campo Wazuh que contiene el Event ID de eventos Windows. |
| `data.srcuser` | Campo Wazuh con el usuario origen en eventos de autenticación Linux. |
| `data.srcip` | Campo Wazuh con la IP origen de la conexión. |
| Sysmon (Linux) | Herramienta de monitoreo de procesos y comandos en endpoints Linux. |

---

## 4. Metodología: Construcción del Dashboard

### Paso 1: Identificar el field name correcto para Event IDs

Antes de construir cualquier panel, es necesario conocer el nombre exacto del campo que contiene los Event IDs en el índice `wazuh-archives-*`. El campo correcto es:

```
data.win.system.eventid
```

Si el campo no aparece en el autocompletado del editor de visualizaciones, es necesario refrescar el listado de campos del index pattern:

```
Hamburger → Dashboard Management → Index Patterns → wazuh-archives-* → Refresh field list (ícono top-right)
```

Esto actualiza el mapeo de campos, incrementando el conteo de ~600 a ~895 campos disponibles.

### Paso 2: Validar eventos con Discover antes de visualizar

Para cualquier query que se vaya a construir en un panel, primero se valida en **Discover** bajo el índice `wazuh-archives-*`. Esto evita crear paneles vacíos por field names incorrectos o por ventanas de tiempo sin datos.

```
data.win.system.eventid: 4625
```

---

## 5. Paneles Construidos

### Panel 1: Failed Windows Logon (Métrica)

Tipo de visualización: **Metric** — muestra el conteo total como un número único.

Query aplicada:

```
data.win.system.eventid: 4625
```

Este panel muestra el número de intentos de autenticación fallidos en el endpoint Windows dentro de la ventana de tiempo seleccionada. El Event ID `4625` es el indicador primario de fallos de logon en Windows.

### Panel 2: Windows Account Changes Over Time (Gráfica de Línea)

Tipo de visualización: **Line Chart** — traza la frecuencia de eventos a lo largo del tiempo.

Query aplicada (cubre creación, modificación y eliminación de cuentas):

```
data.win.system.eventid: (4720 OR 4722 OR 4723 OR 4724 OR 4725 OR 4726 OR 4732 OR 4733 OR 4738)
```

Configuración de ejes:
- **Y Axis:** Count (métrica de agregación)
- **X Axis:** Date Histogram sobre el campo `timestamp`
- **Split Series:** Terms sobre `data.win.system.eventid` (para distinguir cada tipo de cambio de cuenta)

Este panel permite detectar picos anómalos en la gestión de cuentas, como una creación masiva de usuarios (`4720`) o eliminaciones de grupos de seguridad (`4733`).

### Panel 3: Sysmon Activities (Tabla de Datos)

> **Variación personal respecto al laboratorio base:** En lugar de una tabla genérica de SSH fallido, este panel rastrea la actividad de comandos ejecutados desde el endpoint Linux a través de Sysmon, proporcionando visibilidad de procesos y ejecución de comandos.

Tipo de visualización: **Data Table**

Filtro de agente aplicado:

```
agent.name: <nombre-agente-linux>
```

Buckets (columnas) configurados en orden:

| Bucket | Agregación | Campo |
| :--- | :--- | :--- |
| 1 | Terms | `agent.name` |
| 2 | Date Histogram | `timestamp` |
| 3 | Terms | `data.srcuser` |
| 4 | Terms | `data.srcip` |

Esta tabla permite al analista correlacionar de forma rápida qué comandos o procesos se ejecutaron, desde qué usuario y en qué momento, facilitando la detección de ejecución maliciosa o no autorizada.

### Dashboard Final

![Dashboard SOC con los tres paneles terminados](img/SOC-Wazuh-Day04-dashboard-final.png)[^1]
[^1]: Vista completa del dashboard con los paneles: Failed Windows Logon (métrica), Windows Account Changes Over Time (línea) y Sysmon Activities (tabla de datos).

---

## 6. Detecciones y Event IDs Relevantes

| Tipo | Valor | Descripción |
| :--- | :--- | :--- |
| Event ID | `4625` | Failed Logon en Windows — brute force o credenciales incorrectas. |
| Event ID | `4720` | User Account Created — posible creación no autorizada de cuentas. |
| Event ID | `4733` | Member Removed from Security-Enabled Group — modificación de privilegios. |
| Campo | `data.srcip` | IP origen de autenticaciones — útil para correlacionar ataques externos. |
| Campo | `data.srcuser` | Usuario utilizado en el intento de acceso. |

---

## 7. Conclusiones y Aprendizajes Clave

### Lecciones Aprendidas

Comprendí que antes de construir cualquier visualización en Wazuh es indispensable validar los field names en Discover, ya que el índice de archivos puede tener campos no mapeados hasta que se refresca manualmente el index pattern. Asumir que un campo existe sin verificarlo genera paneles vacíos y pérdida de tiempo.

También aprendí que la elección del tipo de visualización importa: una métrica comunica urgencia (cuántos fallos ahora), una línea temporal comunica tendencia (¿está aumentando?), y una tabla comunica contexto (¿quién, desde dónde y cuándo?). Los tres tipos juntos forman una vista completa para un analista de primer nivel.

### Retos Superados

El principal reto fue que el campo `data.win.system.eventid` no aparecía en el autocompletado del editor de visualizaciones. Lo resolví refrescando el field list del index pattern desde Dashboard Management, lo que incrementó los campos disponibles de ~600 a ~895 y desbloqueó el acceso a todos los campos de telemetría Windows.

Para el panel de Sysmon Activities, opté por rastrear la ejecución de comandos en Linux en lugar de solo fallos SSH, lo que enriquece la cobertura del dashboard y da visibilidad sobre la cadena de ejecución de procesos en el endpoint.

---

## 8. Referencias y Fuentes

* [Wazuh Documentation — OpenSearch Dashboards](https://documentation.wazuh.com/current/user-manual/wazuh-dashboard/index.html)
* [Microsoft Event ID 4625 — An account failed to log on](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)
* [Microsoft Event ID 4720 — A user account was created](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4720)
* [MITRE ATT&CK T1078 — Valid Accounts](https://attack.mitre.org/techniques/T1078/)
* [MITRE ATT&CK T1136 — Create Account](https://attack.mitre.org/techniques/T1136/)
* [Wazuh SOC Analyst Challenge — Parte 4](https://www.youtube.com/watch?v=wkfW_qu6kDw)
