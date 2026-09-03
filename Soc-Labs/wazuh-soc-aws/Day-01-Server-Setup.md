# 📡 [Write-up] Día 1: Despliegue del Servidor Wazuh y Configuración de Archives en AWS

![AWS](https://img.shields.io/badge/Plataforma-AWS_EC2-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh_v4.9-00A4E4?style=for-the-badge)
![Dificultad](https://img.shields.io/badge/Dificultad-Media-orange?style=for-the-badge)

## 1. Resumen Ejecutivo y Metadatos
* **Fecha:** 02/09/2026
* **Rol:** SOC Analyst L1 / Cloud Security Engineer
* **Entorno:** AWS Cloud (EC2)
* **Objetivo:** Desplegar una instancia centralizada de Wazuh All-in-One (Manager, Indexer y Dashboard) sobre Ubuntu 24.04 LTS en AWS, asegurando los grupos de red y habilitando el almacenamiento de telemetría completa (*Archives*) para habilitar capacidades de Threat Hunting retrospectivo.

---

## 2. Entorno de Trabajo y Especificaciones de Infraestructura
* **Instancia EC2:** Ubuntu 24.04 LTS (`t3.medium`).
* **Almacenamiento:** Volumen EBS de 100 GiB `gp3`.
* **Reglas de Red (Security Groups):**
  * `Port 22 (SSH):` Restringido exclusivamente a la dirección IP pública del analista.
  * `Port 443 (HTTPS):` Tráfico web para acceso a la consola Wazuh Dashboard.
  * `Port 1514 (TCP):` Ingesta de eventos y comunicación cifrada con los agentes.
  * `Port 1515 (TCP):` Registro y enrolamiento automatizado de agentes.

![Tpología del Laboratorio](./img/aws_architecture_animation.gif)
---

## 3. Fundamentos Teóricos: Ingesta por Alertas vs. Archives
Por diseño predeterminado, Wazuh únicamente procesa e indexa aquellos eventos que coinciden con una regla de detección de severidad mínima (`wazuh-alerts-*`). 

Desde una perspectiva analítica de SOC, este modelo genera **puntos ciegos**: eventos benignos contextuales (como inicios de sesión autorizados o consultas DNS legítimas) no disparan alertas y se pierden. Habilitar el módulo de **Archives** (`wazuh-archives-*`) garantiza la indexación del 100% de la telemetría recibida, permitiendo una reconstrucción cronológica completa de una intrusión.

---

## 4. Metodología de Implementación

### Paso 1: Configuración de Presupuestos (AWS Budgets)
Para prevenir consumos imprevistos de cómputo en la nube, se creó un presupuesto mensual recurrente de **$25 USD** en *AWS Billing & Cost Management*, estableciendo dos alertas automáticas por correo electrónico:
* **Umbral 1:** Al alcanzar el 40% ($10 USD) del consumo real.
* **Umbral 2:** Al alcanzar el 100% ($25 USD) del consumo proyectado/real.

### Paso 2: Instalación del Servidor Central Wazuh
Tras conectarse mediante SSH con la clave privada `.pem`:
```bash
chmod 400 lab-key.pem
ssh -i "lab-key.pem" ubuntu@<IP_PUBLICA_EC2>
```
A continuación se actualizanon los repositorios y se descargó el instalador automático de `Wazuh`

```bash
sudo apt update && sudo apt upgrade -y
curl -sO [https://packages.wazuh.com/4.9/wazuh-install.sh](https://packages.wazuh.com/4.9/wazuh-install.sh)
sudo bash ./wazuh-install.sh -a
```

> ❗Importante, guardar las credenciales de Wazuh❗

### Paso 3: Activación del Módulo de Archives

#### 1.Configuración del Manager (ossec.conf):

Se modificó la directiva global para forzar el guardado de todos los logs. 

```xml
<!-- /var/ossec/etc/ossec.conf -->
<ossec_config>
  <global>
    <logall>yes</logall>
    <logall_json>yes</logall_json>
  </global>
</ossec_config>
```

Ahora reiniciamos el manager para que se apliquen los cambios 
`sudo systemctl restart wazuh-manager`

Se habilita el módulo de archivos para que Filebeat envíe la totalidad de los logs hacia el Wazuh Indexer: 


```YAML
# /etc/filebeat/filebeat.yml
archives:
  enabled: true
```

Por último se reinicia Filebeat `sudo systemctl restart filebeat`

### Paso 4: Creación del Patrón de Índices en Wazuh Dashboard

  * Se accedió vía HTTPS a https://<IP_PUBLICA_EC2>.

  * Se navegó a Dashboard Management > Index Patterns.

  * Se creó el índice wazuh-archives-*, configurando @timestamp como el campo temporal principal.

  * Se validó en la vista Discover que los eventos crudos del sistema comenzaron a poblar la base de datos de OpenSearch.

### 5. Conclusiones y Lecciones Aprendidas

  Retención y Visibilidad: Configurar Archives es un requisito indispensable para un entorno de investigación forense, transformando a Wazuh de un motor de alertas pasivo a una plataforma integral de telemetría y Threat Hunting.

  Gobierno y Control de Recursos Cloud: El uso de alarmas presupuestarias junto con el apagado deliberado (Stop instance) al finalizar cada sesión de trabajo garantiza una administración eficiente de los créditos en AWS sin riesgo de comprometer la persistencia de los datos en el disco EBS.
