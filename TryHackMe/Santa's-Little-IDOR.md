# 🚪 [Write-up] Santa's Little IDOR - TryHackMe

![TryHackMe](https://img.shields.io/badge/TryHackMe-212c42?style=for-the-badge&logo=tryhackme&logoColor=cc0000)
![Difficulty: Medium](https://img.shields.io/badge/Dificultad-Media-orange?style=for-the-badge)

## 1. Resumen Ejecutivo y Metadatos
* **Fecha:** 05/12/2025
* **Plataforma:** TryHackMe
* **Categoría:** Web Exploitation / Broken Object Level Authorization (BOLA/IDOR)
* **Objetivo:** Identificar y explotar una vulnerabilidad de Referencia Directa Insegura a Objetos (IDOR) en el sistema de gestión del laboratorio para acceder a información confidencial de otros usuarios y recuperar la bandera oculta.
---

## 2. Entorno de Trabajo y Herramientas
* **Sistema Atacante:** Kali Linux
* **Sistema Objetivo:** Aplicación Web expuesta en el laboratorio
* **Herramientas utilizadas:** Herramientas de desarrollador del navegador (DevTools), Burp Suite (opcional para interceptar), e inspectores de red.

---
## 3. Fase de Reconocimiento y Enumeración

### Análisis de la Aplicación Web
Al ingresar al sitio web del laboratorio, se interactuó con las funciones normales de la página. El sistema está diseñado para mostrar información o configuraciones específicas basadas en el usuario autenticado.

### Identificación del Parámetro Vulnerable
Al observar la estructura de las peticiones HTTP realizadas en segundo plano, se detectó que la aplicación llama a los recursos utilizando un parámetro predecible en la URL o en un endpoint de la API. 

* **Ejemplo de la estructura identificada:** `http://<ip objetivo>/api/parents/view_accountinfo?user_id=10`.

Esta dependencia directa de un número secuencial o ID expuesto es el indicador clásico de un posible vector **IDOR**.

---

## 4. Fase de Explotación (Manipulación de Parámetros)

### Prueba de Concepto (PoC)
Para confirmar la vulnerabilidad, se procedió a realizar una manipulación manual del parámetro. Desde la ventana de **Storage** dentro de las Developer Tools se modificó el valor original del identificador en la petición web por uno inmediatamente posterior.

![Captura ventana local storage]()

El servidor procesó la solicitud **sin verificar si el usuario actual tenía autorización** para ver ese objeto, devolviendo en pantalla los datos privados correspondientes a una cuenta ajena.

### Buscando posibles IDOR
No siempre los IDOR son tan sencillos como apreciar un número y cambiarlo, dentro del laboratorio se nos muestra la técnica de **Encoding** donde se cifra el valor de nuestro interés, en la siguiente captura se puede observar una consulta en base64 con el valor Mg== el cual representa el número decimal 2.

![Captura_Encoding]()

Otra forma en la que podemos explotar un IDOR es mediante las consultas que usen funciones Hash, a primera vista pareciera ser un valor al azahar pero entendiendo el funcionamiento del algoritmo Hash sabemos que si encontramos el valor empleado para la funcion podremos replicarla, para esto se puede hacer uso de un Hash identifier.

### Inspección con BurpSuite
Para la siguiente misión se nos solicita encontrar el **user_id** del hijo nacido en 2019-04-17, para esto haremos uso de BurpSuite intruder para iterar las peticiones y encontrar lo que nos solicitan. 

**Configurando el Payload:** 
1. Mandaremos la consulta `GET /api/child/b64/<id_base64>` al intruder. 
2. Seleccionamos el parámetro <id_base64> y le damos al botón de add. 
3. Configuraremos el Payload definiendo tipo como numerico, el rango de números a probar y añadiremos una regla que codifique en base64 estos valores

![Configuración del payload]()

4. Dentro de settings en la parte Grep-Match añadiremos la fecha de nacimiento que nos dieron, así ya no tendremos que buscar dentro de cada consulta para encontrarlo. 

![Configuración Grep-Match]()

5. Por último lanzamos el ataque y buscamos la consulta que coincida con nuestra búsqueda.

![Resultados del ataque]()

Finalmente encontramos que el id solicitado es el 19. 

## 5. Conclusiones y Aprendizajes Clave

### Lecciones Aprendidas: 
En este laboratio conocí los IDOR como estos se pueden encontrar de distintas maneras dentro de una página web y que no basta con encriptar la información para que la página sea segura.
También aprendí el uso de la herramienta intruder dentro de BurpSuite, que hasta ahora no había usado la cuál facilita las consultas a servidor y permite seleccionar la información importante, codificar, etc. 

### Retos superados: 
Hacer uso de BurpSuite para enviar varias peticiones con números consecutivos que a su vez se codificaban en base64 antes de ser enviados como consuta y el manejo de la respuesta mediante la búsqueda de la fecha de nacimiento dentro de las respuestas. 

### Mitigación: 
Para solucionar las vulnerailidades de este tipo la página web debe implementar un control de acceso basado en roles, cada vez que se solicite la información de un `user_id`, el servidor debe verificar si la sesión del usuario que realiza la petición tiene autorización para verla.  
