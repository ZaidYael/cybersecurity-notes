# 🔍 [Write-up] Splunk Basics - TryHackMe

![TryHackMe](https://img.shields.io/badge/TryHackMe-212c42?style=for-the-badge&logo=tryhackme&logoColor=cc0000)
![Difficulty: Medium](https://img.shields.io/badge/Dificultad-Media-orange?style=for-the-badge)


## 1. Resumen Ejecutivo y Metadatos
* **Fecha:** 03/12/2025
* **Plataforma:** TryHackMe
* **Categoría:** Blue Team / SIEM / Log Analysis & Forensics
* **Objetivo:** Comprender la arquitectura fundamental de un sistema SIEM, aprender a navegar por la interfaz de Splunk y utilizar el lenguaje de búsqueda SPL (*Search Processing Language*) para auditar eventos de seguridad y responder preguntas de investigación forense.

---

## 2. Entorno de Trabajo y Herramientas
* **Sistema de Análisis:** Instancia Web de Splunk Enterprise (Desplegada en el laboratorio)
* **Fuentes de Datos (Datasources):** Logs de servidores web (Apache/Nginx), logs de eventos de Windows (*Event Logs*), registros de red/firewall y VPN.
* **Herramientas utilizadas:** Splunk Search & Reporting App, lenguaje SPL, filtros por campos (*Selected / Interesting Fields*).

---

## 3. Metodología de Análisis y Sintaxis SPL

Para responder a las preguntas de investigación del laboratorio, se aplicó una metodología estructurada dividida en tres pasos clave:

### A. Selección e Inspección del Índice (Index)
El primer paso en Splunk consiste en especificar el conjunto de datos sobre el cual queremos trabajar mediante el parámetro `index`:
```spl
index="main"
``` 
### Encontrando la IP del atacante

Como primer actividad sospechosa encontramos dentro de la gráfica se observan unos picos de actividad irregulares, en segundo lugar dentro de los **user_agent** hay algunos comandos inusuales como wget, curl, zgrab y sqlmap. 

Para filtrar nuestros resultados y acercarnos a nuestro atacante emplearemos la siguiente consulta excluyendo los agentes conocidos: 

```index=main sourcetype=web_traffic user_agent!=*Mozilla* user_agent!=*Chrome* user_agent!=*Safari* user_agent!=*Firefox* ```  

Con este filtro observamos que solo nos queda una ip relacionada a toda la actividad, por ende está es la ip del atacante 

imagen 

### ¿Qué día hubo más tráfico de logs?

Para responder esta pregunta haremos uso del comando timechart y el parámetro span=1d para contar por cada día, nuestra consulta completa quedaría de la siguiente forma: 

index=main sourcetype=web_traffic | timechart span=1d count | sort by count | reverse

Hacemos uso de reverse para que aparezca como primer valor el máximo.

El día que hubo más tráfico en los logs fue el 2025-10-12.

