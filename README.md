 # 🛡️ Implementación y Análisis de ClamAV Antivirus

![Seguridad](https://img.shields.io/badge/Seguridad-🛡️-blue)
![Licencia](https://img.shields.io/badge/Licencia-GPL-orange)
![Herramienta](https://img.shields.io/badge/Antivirus-ClamAV-green)

## 📝 Descripción del Proyecto
Este repositorio contiene el análisis técnico y la configuración de **ClamAV**, el motor antivirus de código abierto estándar en entornos Linux. El objetivo de este trabajo es demostrar la capacidad de detección de malware, la gestión de firmas y la automatización de escaneos en sistemas de archivos y servidores de correo.

---

## 👥 Equipo de Seguridad (Autores)
Proyecto desarrollado por el equipo de ciberseguridad:

* 🛡️ **Ayoub** - *Configuración del Motor y Base de Datos*
* 🛡️ **Eric** - *Implementación de Freshclam y Actualizaciones*
* 🛡️ **Jose** - *Análisis de Logs y Pruebas de Detección*

---

## 🛠️ Aspectos Técnicos de ClamAV
En este trabajo hemos cubierto los pilares fundamentales de la herramienta:

### 1. 🔍 Escaneo Multihilo (`clamd`)
Hemos configurado el demonio de ClamAV para mejorar el rendimiento del sistema mediante el uso de hilos, permitiendo escaneos rápidos sin saturar la CPU.

### 2. 🔄 Actualización de Firmas (`freshclam`)
Implementación del servicio de actualización automática para asegurar que la base de datos de virus esté siempre al día contra las últimas amenazas (CVEs).

### 3. 📂 Soporte de Formatos
Análisis de la capacidad de ClamAV para profundizar en archivos comprimidos (`.zip`, `.rar`, `.tar.gz`) y ejecutables de Windows/Linux.

---

## 🚀 Comandos Principales Utilizados
Para demostrar el dominio de la herramienta, hemos incluido pruebas con:

* **Instalación:** `sudo apt install clamav clamav-daemon`
* **Escaneo de directorio:** `clamscan -r /home/usuario`
* **Limpieza de infectados:** `clamscan --remove=yes`

---

## 🛡️ Análisis Técnico: Implementación de ClamAV en Servidores de Correo

ClamAV se establece como el motor de código abierto estándar para la protección de pasarelas de correo. Su diseño está orientado específicamente al análisis de archivos, siendo capaz de identificar virus, troyanos, gusanos y exploits complejos ocultos en documentos PDF o macros de Office.

### ⚙️ Arquitectura de Integración con Postfix
Dado que Postfix prioriza la modularidad, no realiza el análisis de virus de forma nativa para evitar que un fallo en el motor detenga el flujo de mensajería. Para solventar esto, se utilizan capas intermedias de comunicación:

* **Amavisd-new:** Actúa como un gestor de tráfico que intermedia entre Postfix y ClamAV, procesando el contenido y retornando las instrucciones de filtrado.
* **Protocolo Milter (`clamav-milter`):** Permite que Postfix consulte a ClamAV en tiempo real durante la sesión SMTP. Esta técnica es altamente eficiente, ya que permite rechazar correos maliciosos antes de que la conexión llegue a completarse.

### 🛡️ Gestión de Amenazas y Políticas de Acción
El sistema permite automatizar la respuesta ante archivos infectados mediante diferentes políticas configurables:
1.  **Rechazo (Reject):** Se corta la conexión devolviendo un error al remitente.
2.  **Cuarentena:** El archivo se aísla en un directorio seguro para su posterior auditoría.
3.  **Etiquetado:** Se entrega el correo al destinatario pero con una marca de advertencia en el asunto o en las cabeceras.

### 🚀 Configuración y Seguridad Complementaria
La activación técnica se gestiona mediante directivas en el archivo `/etc/postfix/main.cf`, destacando el uso de `smtpd_milters` para vincular el socket del antivirus. 

Para obtener una protección integral de 360°, ClamAV se complementa con las siguientes herramientas:
* **SpamAssassin:** Para el análisis de texto y detección de phishing.
* **Postgrey:** Implementación de *Greylisting* para mitigar el spam de bots.
* **Fail2Ban:** Protección perimetral contra ataques de fuerza bruta.
* **RBLs:** Uso de listas negras públicas para el bloqueo de servidores maliciosos conocidos.

---

## 🏁 Conclusión
Tras las pruebas realizadas , concluimos que ClamAV es la herramienta definitiva para la protección de servidores gracias a su ligereza y su naturaleza Open Source, permitiendo una integración total en cualquier flujo de trabajo DevOps.

---

> ### 👩‍🏫 Entrega Académica 
> **Profesora:** [Alina]
> **Autores:** [Ayoub,Eric,Jose]
> **Fecha:** Febrero 2026
