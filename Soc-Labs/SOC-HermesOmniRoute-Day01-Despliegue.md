# 🤖 [Proyecto] Despliegue de Agente Autónomo Local con Hermes Agent y OmniRoute Gateway — Día 1: Arquitectura y Despliegue

![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Hermes Agent](https://img.shields.io/badge/Hermes_Agent-0052CC?style=for-the-badge)
![OmniRoute](https://img.shields.io/badge/OmniRoute-FF5722?style=for-the-badge)
![Dificultad](https://img.shields.io/badge/Dificultad-Media-orange?style=for-the-badge)

## Tabla de Contenidos
- [1. Resumen Ejecutivo y Metadatos](#1-resumen-ejecutivo-y-metadatos)
- [2. Objetivo y Motivación](#2-objetivo-y-motivación)
- [3. Arquitectura y Diseño](#3-arquitectura-y-diseño)
- [4. Implementación](#4-implementación)
- [5. Problemas Encontrados y Soluciones](#5-problemas-encontrados-y-soluciones)
- [6. Resultado Final y Validación](#6-resultado-final-y-validación)
- [7. Próximos Pasos](#7-próximos-pasos)
- [8. Conclusiones y Aprendizajes Clave](#8-conclusiones-y-aprendizajes-clave)
- [9. Referencias y Fuentes](#9-referencias-y-fuentes)

---

## 1. Resumen Ejecutivo y Metadatos
* **Fecha:** 20/09/2026
* **Rol:** Blue Team & AI Automation Engineer
* **Entorno:** Local Workstation (Linux, Node.js, Python)
* **Objetivo:** Desplegar y conectar un agente CLI autónomo (Hermes Agent) a un gateway de enrutamiento inteligente de modelos (OmniRoute) para automatizar el análisis de repositorios de ciberseguridad, perfiles técnicos y búsqueda de vacantes sin bloqueos por límites de API.

---

## 2. Objetivo y Motivación

La proliferación de modelos de lenguaje aplicados a ciberseguridad requiere entornos robustos capaces de procesar gran volumen de contexto (write-ups, análisis de logs, perfiles de habilidades y ofertas laborales). Muchos proveedores de inferencia imponen restricciones severas de tokens por minuto o cuotas de salida.

El propósito de este proyecto fue diseñar e implementar una arquitectura local desacoplada donde el agente de ejecución (Hermes Agent) interactúe con un proxy inteligente (OmniRoute Gateway). Esto garantiza alta disponibilidad mediante balanceo y mecanismos de respaldo (*fallbacks*) en caliente, permitiendo una automatización fluida para tareas de perfilamiento profesional y análisis de seguridad.

---

## 3. Arquitectura y Diseño

El flujo de trabajo se basa en un pipeline cliente-gateway-proveedores:

```
+---------------------+         +------------------------+         +--------------------------+
|  Hermes Agent CLI   | ------> |   OmniRoute Gateway    | ------> |  Model Providers         |
|  (Tools, Chromium,  |  HTTP   |  (Puerto Local :20128) |  Proxy  |  (OpenAI-Compatible,     |
|   Memory & Context) |  REST   |  (Combos & Fallbacks)  |         |   Groq, OpenRouter, etc) |
+---------------------+         +------------------------+         +--------------------------+
```

### Componentes del Entorno

| Componente | Tipo | Descripción |
| :--- | :---: | :--- |
| **Hermes Agent** | CLI Autonomous Agent | Motor de razonamiento y ejecución de herramientas locales (Python, shell, browser, filesystem). |
| **OmniRoute** | AI Gateway / Reverse Proxy | Enrutador local (puerto 20128) encargado de la resiliencia de API, balanceo de carga y reintentos. |
| **Context Store** | Repositorio Markdown | Archivos locales estructurados (`cv.md`, `writeups_summary.md`) para alimentar la memoria del agente. |
| **Chromium Headless** | Web Automation | Módulo integrado para scraping dinámico y evaluación de plataformas web. |

---

## 4. Implementación

### Paso 1: Configuración e Inicio del Gateway OmniRoute

Se configuró el gateway local en el puerto `20128`, definiendo listas de modelos con soporte de autocombo y cadenas de respaldo para absorber fallos por límites de tasa (*rate limiting*).

```bash
# Inicialización del servicio OmniRoute en background/puerto local
omniroute start --port 20128 --enable-fallbacks --auto-combo
```

![Configuración e inicialización del gateway en terminal](img/Personal-HermesOmniRoute-config-terminal.png)[^1]
[^1]: Terminal con la salida del gateway OmniRoute levantado en `http://localhost:20128` mostrando los endpoints activos y reglas de enrutamiento.

### Paso 2: Conexión de Hermes Agent al Endpoint Custom

Se configuró Hermes Agent para utilizar la interfaz OpenAI-compatible expuesta por OmniRoute:

```bash
# Configuración del proveedor personalizado en Hermes Agent
hermes config set provider custom
hermes config set base_url http://localhost:20128/v1
hermes config set api_key omniroute-local
```

### Paso 3: Estructuración del Contexto Local de Seguridad

Se crearon y organizaron los documentos base para el análisis del agente:
- `cv.md`: Perfil profesional enfocado en Ingeniería de Software y Blue Team.
- `writeups_summary.md`: Resumen consolidado de write-ups técnicos (CTFs, TryHackMe, análisis de logs y labs de SOC).

---

## 5. Problemas Encontrados y Soluciones

| Problema | Causa Raíz | Solución Aplicada |
| :--- | :--- | :--- |
| Bloqueo en respuestas extensas por agotamiento de tokens de salida. | Límites estrictos de longitud por solicitud en ciertos proveedores en la capa gratuita (ej. Groq). | Configuración de reglas de *hot-swapping* y autocombo en OmniRoute para redirigir dinámicamente el tráfico a modelos de mayor ventana de salida. |
| Timeouts durante la lectura simultánea de repositorios grandes. | Intentos de carga completa en un único turno conversacional. | Implementación de comandos particionados con herramientas nativas de Hermes Agent (`search_files`, `read_file` paginado). |

---

## 6. Resultado Final y Validación

El agente logró procesar el repositorio local de notas de seguridad, correlacionar las competencias técnicas con requerimientos de vacantes técnicas y emitir reportes de ajuste (*fit*) detallados con justificaciones verificables.

![Flujo de trabajo y validación del agente](img/Personal-HermesOmniRoute-workflow.png)[^2]
[^2]: Ejecución del agente analizando el perfil local y generando el reporte sin interrupciones por cuota de API.

---

## 7. Próximos Pasos

- [ ] Integrar un pipeline de ingesta automatizada para clasificar alertas de Wazuh mediante el agente.
- [ ] Implementar un cronjob local en Hermes Agent para monitorizar nuevas publicaciones técnicas y vacantes de ciberseguridad.

---

## 8. Conclusiones y Aprendizajes Clave

### Lecciones Aprendidas
Comprendí la importancia crítica de contar con una capa de abstracción de modelos (AI Gateway) en entornos de agentes autónomos. Desacoplar al agente del proveedor directo de inferencia garantiza que las tareas complejas de análisis no fallen a mitad de ejecución.

### Retos Superados
Logré solucionar las restricciones de cuota y tamaño de contexto mediante la orquestación de *fallbacks* en caliente en OmniRoute, manteniendo una latencia baja y un flujo de ejecución CLI completamente automatizado.

---

## 9. Referencias y Fuentes

* [Hermes Agent Documentation](https://hermes-agent.nousresearch.com/docs)
* [OmniRoute Gateway Architecture](https://github.com/danielfrg/omniroute)