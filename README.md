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

## 📂 Estructura del Repositorio
* `/config`: Archivos `clamd.conf` y `freshclam.conf` optimizados.
* `/scripts`: Automatización de escaneos programados (Cron jobs).
* `/docs`: Informe detallado sobre la efectividad frente al archivo de prueba EICAR.

---

## 🏁 Conclusión
Tras las pruebas realizadas por **Ayoub, Eric y Jose**, concluimos que ClamAV es la herramienta definitiva para la protección de servidores gracias a su ligereza y su naturaleza Open Source, permitiendo una integración total en cualquier flujo de trabajo DevOps.

---

> ### 👩‍🏫 Entrega Académica 
> **Profesora:** [Alina]
> **Autores:** [Ayoub,Eric,Jose]
> **Fecha:** Febrero 2026
