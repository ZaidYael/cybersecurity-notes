# ⚙️ [Write-up] n8n: CVE-2025-68613 - TryHackMe

![TryHackMe](https://img.shields.io/badge/TryHackMe-212c42?style=for-the-badge&logo=tryhackme&logoColor=cc0000)
![CVE-2025-68613](https://img.shields.io/badge/CVE-2025--68613-red?style=for-the-badge)

## 1. Resumen Ejecutivo y Metadatos
* **Fecha:** 27/12/2025
* **Plataforma:** TryHackMe 
* **Categoría:**  CVE Analysis / Remote Code Execution (RCE) / Vulnerability Exploitation
* **Objetivo:** Comprender y documentar la explotación del fallo crítico CVE-2025-68613 en la plataforma de automatización de flujos de trabajo n8n, el cual permite la ejecución remota de código (RCE) mediante la evasión del sandbox de JavaScript.

---

## 2. Entorno de Trabajo y Herramientas
* **Sistema Atacante:** Kali Linux
* **Software Objetivo:** n8n 0.211.0-1.120.3
* **Herramientas utilizadas:** Panel Web de n8n, herramientas de desarrollador, sintaxis de expresiones e inyección de payloads de Node.js.

---

## 3. Fase de Reconocimiento y Enumeración

### Reconocimiento de la Interfaz Web
Al iniciar el entorno, se obtuvo acceso al panel de edición de flujos de trabajo (Workflow Editor) de n8n. Esta falla requiere autenticación previa (incluso con bajos privilegios), simulando un escenario donde un atacante persistente o un usuario interno malicioso intenta comprometer la infraestructura del servidor.

### Identificación del Vector de Ataque
La vulnerabilidad radica en el sistema de evaluación de expresiones matemáticas o lógicas que utiliza la plataforma dentro de los bloques `{{ }}`. En las versiones vulnerables, n8n no aísla de forma correcta el entorno de ejecución de Node.js, lo que abre una brecha para realizar una **Inyección de Expresiones**.

## 4. Fase de Explotación y Evasión de Sandbox
Para comprobar que el nodo interpretaba correctamente las directivas dinámicas del servidor, se creó un flujo de trabajo básico y se agregó un nodo de tipo **"Set"**. Dentro de los campos de entrada que aceptan expresiones lógicas, se inyectaron variables internas del sistema como:
```javascript
{{ $version }}
```
Al ejecutar el nodo, el sistema devuelve la versión del software activo confirmando nuestro vector.

### Escape del SandBox de Node.js 
Dado que el sandbox de JavaScript era deficiente, fue posible invocar el contexto global de ejecución mediante el objeto principal this. En los entornos de Node.js desprotegidos, acceder al objeto this permite escalar privilegios hacia las directivas internas del proceso (process.mainModule.require).

Se construyó un payload malicioso estructurado de la siguiente forma para forzar al servidor a importar el módulo del sistema operativo child_process y ejecutar un comando nativo (en este caso, un whoami o listado de directorios para ubicar las flags):
```javascript
{{ this.constructor.constructor("return process.mainModule.require('child_process').execSync('ls').toString()")() }}
```

![Captura resultado ls]()

El comando se ha ejecutado con éxito y podemos observar que hay un archivo **flag.txt**. Ejecutando ahora `cat flag.txt` nos muestra la flag que búscabamos. 


###Detección y Acción 

Para identificar si nuestro sistema ha sido expuesto a esta vulnerabilidad debemos primero revisar si está dentro del rango de versiones afectadas, en caso de confirmar esto debemos buscar pistas que nos indiquen si se ha vulnerado nuestro sistema. 
Una forma rápida y sencilla de hacer esto es mediante la búsqueda de palabras clave asociadas a vulnerar nuestro sistema, por ejemplo: 

```bash
# Search for potential exploitation patterns
grep -r "this.process" /path/to/n8n/workflows/
grep -r "mainModule.require" /path/to/n8n/workflows/
grep -r "child_process" /path/to/n8n/workflows/
grep -r "binding(" /path/to/n8n/workflows/
grep -r "_load(" /path/to/n8n/workflows/
```
Para confirmar que nuestro sistema se encuntre libre de ataques debemos revisar los logs donde buscaremos: 

+ Ejecuciones de comandos inusuales.
+ Conexiones a travéz de procesos de n8n.
+ Acceso a información privilegiada.
+ Evaluación de expresiones fallida, lo cuál puede indicar intentos de exploit.

**Recomendaciones**
+ Actualizar a versiones seguras.
+ Mitigación temporal antes de actualizar mediante la restricción de editar los workflows solo permitida a usuarios de confianza.
+ Correr n8n con los mínimos privilegios.

#### Detección con Sigma 
En base a lo anteriormente mencionado podemos generar una regla que nos ayude a detectar esto de forma automática. 

```bash
title: N8N Workflow RCE Attempt
status: experimental
description: Detects attempts to inject JavaScript expressions into n8n workflow payloads that execute OS commands via "this.process.mainModule.require('child_process').execSync(...)""
author: TryHackMe Content Engineering Team
references:
  - <https://github.com/wioui/n8n-CVE-2025-68613-exploit>
date: 2025-12-23
tags:
  - attack.execution
  - attack.t1059.007
logsource:
  category: webserver
  product: generic
detection:
  selection:
    cs-method: POST
    cs-uri-stem|endswith: /rest/workflows

  keywords:
    # Strong indicators of this n8n expression injection RCE
    - "this.process.mainModule.require('child_process')"
    - ".execSync("
    - "={{ (function(){"
    - "toString() })()"

  condition: selection and all of keywords
falsepositives:
  - Security testing / red team simulations
  - Developers storing these exact strings in logged fields
level: high
```

## 5. Conclusiones y Aprendizajes Clave. 

### Lecciones Aprendidas
Gracias a este laboratorio logré comprender una vulnerabilidad actual recientemente descubierta en una plataforma tan popular como lo es n8n y dimensioné el potencial de riesgo que esta implica. 

Descubrí cómo aplicarla pero lo más importante a mi consideración el cómo detectar intentos de explotación de esta vulnerabilidad dentro de nuestro sistema y las opciones de mitigación que existen para reducir los efectos de la misma. 

### Retos superados. 
Analizar la estructura del payload necesario para invocar funciones constructoras de JavaScript sin romper la sintaxis esperada por los nodos del editor de flujos.
