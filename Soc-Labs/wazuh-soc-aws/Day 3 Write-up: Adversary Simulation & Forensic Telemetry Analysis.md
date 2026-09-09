# 📡 [Write-up] Día 3: Simulación Adversaria y Análisis Forense de Telemetría 

![AWS](https://img.shields.io/badge/Plataforma-AWS_EC2-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh_v4.9-00A4E4?style=for-the-badge)
![MITRE](https://img.shields.io/badge/Framework-MITRE_ATT%26CK-RED?style=for-the-badge)
![Windows](https://img.shields.io/badge/OS-Windows_Server-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Linux](https://img.shields.io/badge/OS-Ubuntu_Linux-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

## 1. Resumen Ejecutivo y Metadatos
* **Fecha:** 09/09/2026
* **Rol:** SOC Analyst L1 / Red Team Operator
* **Entorno:** AWS Cloud 
* **Objetivo:** Simular técnicas adversarias reales de las fases de Reconocimiento, Persistencia, Creación de Cuentas y Fuerza Bruta en los *endpoints* víctima (Windows y Linux). Analizar la telemetría generada en la consola **Discover** de Wazuh, identificando Event IDs clave de Windows (`4624`, `4625`, `4720`, `4738`) y logs de autenticación SSH para validar la visibilidad total del SIEM.

---

## 2. Mapa de Técnicas MITRE ATT&CK® Simuladas

| Fase táctica | ID Técnica | Descripción de la técnica | Sistema objetivo | Comando / Acción ejecutada |
| :--- | :---: | :--- | :---: | :--- |
| **Discovery** | `T1087.001` | Account Discovery: Local Account | Windows / Linux | `whoami`, `net user`, `cat /etc/passwd` |
| **Persistence** | `T1136.001` | Create Account: Local Account | Windows | `net user Guest /active:yes` |
| **Privilege Escalation** | `T1098` | Account Manipulation | Windows | `net localgroup Administrators Guest /add` |
| **Credential Access** | `T1110.001` | Brute Force: Password Guessing | Linux (Ubuntu) | Intentos fallidos masivos de autenticación SSH |

---

## 3. Fundamentos Teóricos: Event IDs Críticos de Auditoría en Windows

Durante un análisis forense o respuesta a incidentes (*Incident Response*), los Event IDs del visor de eventos de auditoría de Windows y Sysmon proporcionan la prueba primaria del comportamiento anómalo:

* **Event ID 4624 (Successful Logon):** Un usuario o servicio inició sesión correctamente en el sistema.
* **Event ID 4625 (Failed Logon):** Intento fallido de inicio de sesión (útil para detectar ataques de fuerza bruta o *password spraying*).
* **Event ID 4720 (A User Account was Created):** Se creó una nueva cuenta de usuario local en el sistema objetivo.
* **Event ID 4738 (A User Account was Modified):** Cambios en los atributos o estado de una cuenta (ej. activación de la cuenta `Guest`).
* **Event ID 4732 (A Member was Added to a Security-Enabled Local Group):** Se otorgaron privilegios administrativos agregando un usuario a grupos privilegiados.

---

## 4. Metodología de Ejecución y Simulación

### Fase 1: Reconocimiento y Abuso de Cuentas en Windows

1. **Reconocimiento del sistema (Discovery):**
   Se accedió por RDP al servidor Windows y se ejecutó la enumeración básica del entorno desde una terminal con privilegios administrativos:
   
   ```cmd
     whoami
     ipconfig /all
     net user
   ```  

2. Activación de la cuenta integrada Guest (Persistencia):
Simulando un comportamiento adversario para habilitar vectores de acceso secundario, se activó la cuenta nativa Guest:

```cmd
net user Guest /active:yes
net user Guest "P@ssword123!"
``` 

3. Escalación de Privilegios Locales:
Se vinculó la cuenta de invitado recién activada al grupo de administradores locales:

```cmd
net localgroup Administrators Guest /add
``` 

4. Exfiltración Simulada y Archivo de Botín (Linux):
Conexión SSH desde Windows para crear una copia de las cuentas del sistema 

```cmd 
ssh ubuntu@<IP_PRIVADA_LINUX>
whoami
cat /etc/passwd > /tmp/loot.txt
``` 

### Fase 2: Simulación de Ataque de Fuerza Bruta SSH en Linux

Desde una máquina externa o terminal, se ejecutaron múltiples intentos fallidos de autenticación SSH hacia la IP de la máquina Ubuntu para generar eventos de denegación de acceso en /var/log/auth.log:

```cmd
ssh admin@<IP_PRIVADA_LINUX>
# Se ingresaron contraseñas erróneas deliberadamente (3+ intentos)
``` 

---

## 5. Análisis Forense de Telemetría en Wazuh (Discover)

Tras generar las acciones adversarias, se procedió a auditar los logs crudos capturados en la consola de Wazuh a través del patrón de índices wazuh-archives-*.

### ¿Cómo leer los logs?

+ Security ID (SID): Un SID se usa para identificar de forma única una entidad de seguridad o un grupo de seguridad. Las entidades de seguridad pueden representar cualquier entidad que el sistema operativo pueda autenticar.

| Component |	Description |
| :--- | :---: |
| S |	Indica que la cadena es un SID. |
| R |	Indica el nivel de revisión. |
| X |	Indica el valor de entidad del identificador. |
| Y |	Representa una serie de valores de subauthoridad, donde n es el número de valores. |


1. Detección de Creación y Modificación de Cuentas en Windows

    Filtro Aplicado: data.win.system.eventId: "4720"

    Resultado: Se confirmó la captación del evento en el momento exacto en que la cuenta Guest fue activada, identificando el usuario de origen que ejecutó la acción (Administrator) y el objetivo modificado (TargetUserName: Guest).

    ![event 4720]()

2. 4624 -> an account was successfully logged on 

4732 -> a member was added to a security-enabled local group.

2. Detección de Inicios de Sesión SSH Fallidos en Linux

    Filtro Aplicado: agent.name: "Linux-Endpoint" AND data.srcip: "*" AND "Failed password"

    Resultado: Wazuh ingirió correctamente los eventos de sshd, extrayendo en campos estructurados como data.srcuser el nombre del usuario atacado (admin) y la IP origen del atacante (data.srcip).

6. Conclusiones y Lecciones Aprendidas

    Validación del Enfoque "Archives": Confirmar que la activación del módulo Archives (wazuh-archives-*) realizada en el Día 1 fue crucial para visibilizar eventos contextuales que no necesariamente disparan alertas predeterminadas del SIEM de nivel alto.

    Correlación de Event IDs: El monitoreo defensivo en Windows no debe basarse solo en inicios de sesión (4624/4625), sino en eventos de gestión de cuentas (4720, 4738, 4732), ya que son los indicadores primarios de persisencia adversaria.

    Simulación para la Creación de Reglas: Experimentar el rol del atacante (Red Team) genera la perspectiva requerida para el Día 4 y 5, permitiendo saber exactamente qué campos utilizar (data.win.eventdata.targetUserName, data.srcuser) para diseñar Dashboards y Reglas XML personalizadas.







































   
   
