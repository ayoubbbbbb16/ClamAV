# 🚀 Servidor de Correo Seguro con Antivirus

---

## 📖 ¿Qué es Esto?

Un **servidor de correo electrónico** que analiza automáticamente todos los emails en busca de **virus y malware** antes de dejarlos entrar. Si detecta algo peligroso, lo rechaza inmediatamente.

---

## 🔍 Conceptos Clave

### 📧 **Postfix**
Es el **gestor de correos**. Recibe emails por internet y decide si aceptarlos o rechazarlos. Es como un portero del correo.

### 🦠 **ClamAV**
Es el **antivirus**. Analiza cada correo buscando virus, troyanos y código malicioso. Es como el detector de amenazas.

### 🔗 **ClamAV-Milter**
Es el **puente de comunicación** entre Postfix y ClamAV. Le dice a Postfix: "Espera, déjame escanear este correo con ClamAV primero".

### 💾 **Socket**
Un archivo especial que permite que dos programas se comuniquen entre sí dentro del servidor.

### 🔒 **Chroot**
Una "jaula de seguridad" que aísla a Postfix. Si alguien lo hackea, no puede acceder al resto del sistema.

---

## 🛠️ Componentes Principales

| Componente | Función | Utilidad |
| :--- | :--- | :--- |
| 🐧 **Ubuntu Server** | Sistema operativo | Base del servidor |
| 📧 **Postfix** | Gestor de correo (MTA) | Recibe y envía emails |
| 🦠 **ClamAV** | Motor antivirus | Detecta virus en emails |
| 🔗 **ClamAV-Milter** | Interfaz de filtrado | Conecta Postfix + ClamAV |
| 📍 **Socket Unix** | Comunicación segura | Postfix habla con ClamAV |

---

## ⚙️ Instalación Paso a Paso

### 1️⃣ Configurar IP Estática

**¿Por qué?** Los servidores de correo necesitan una dirección IP fija. Si cambia, otros servidores no confían en tus correos (piensan que es spam).

**Archivo:** `/etc/netplan/00-installer-config.yaml`
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.109.120/24]
      routes:
        - to: default
          via: 192.168.109.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

| Línea | Explicación |
| :--- | :--- |
| `dhcp4: no` | No pedir IP automática al router |
| `addresses: [192.168.109.120/24]` | Usar esta IP siempre |
| `via: 192.168.109.1` | Puerta de enlace (router) |
| `nameservers` | Servidores DNS para resolver dominios |

---

### 2️⃣ Instalar Software

**¿Qué hace?** Descarga e instala todos los programas necesarios de forma automática.
```bash
export DEBIAN_FRONTEND=noninteractive
sudo debconf-set-selections <<< "postfix postfix/mailname string erjoay.local"
sudo debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
sudo apt update
sudo apt install -y postfix clamav-daemon clamav-milter mailutils telnet
```

| Comando | ¿Qué hace? |
| :--- | :--- |
| `DEBIAN_FRONTEND=noninteractive` | Evita que aparezcan ventanas interactivas |
| `debconf-set-selections` | Responde automáticamente las preguntas de instalación |
| `apt update` | Actualiza lista de paquetes disponibles |
| `apt install -y` | Instala paquetes sin pedir confirmación |
| `clamav-daemon` | El antivirus que corre en background |
| `clamav-milter` | El filtro que conecta con Postfix |
| `mailutils` | Herramientas para enviar/recibir emails desde terminal |
| `telnet` | Cliente para conectarse al servidor SMTP manualmente |

---

### 3️⃣ Crear Socket de Comunicación

**¿Qué hace?** Crea una carpeta especial donde Postfix y ClamAV pueden comunicarse sin salir de la jaula de seguridad.
```bash
sudo mkdir -p /var/spool/postfix/clamav
sudo chown clamav:postfix /var/spool/postfix/clamav
sudo chmod 755 /var/spool/postfix/clamav
```

| Comando | ¿Qué hace? |
| :--- | :--- |
| `mkdir -p /var/spool/postfix/clamav` | Crea la carpeta clamav dentro de postfix |
| `chown clamav:postfix` | El propietario es `clamav`, el grupo es `postfix` |
| `chmod 755` | Permisos: dueño puede leer/escribir, otros solo leer |

**🔒 Seguridad:** Al crear el socket DENTRO de /var/spool/postfix, ambos servicios pueden comunicarse sin romper la jaula.

---

### 4️⃣ Configurar ClamAV-Milter

**¿Qué hace?** Configuramos qué debe hacer ClamAV cuando encuentra un virus.

**Archivo:** `/etc/clamav/clamav-milter.conf`
```bash
MilterSocket unix:/var/spool/postfix/clamav/clamav-milter.ctl
OnInfected Reject
```

| Parámetro | Explicación |
| :--- | :--- |
| `MilterSocket unix:/var/spool/postfix/clamav/clamav-milter.ctl` | 📍 El socket donde ClamAV escucha a Postfix |
| `OnInfected Reject` | 🚫 Si detecta un virus, rechaza el correo inmediatamente |

**Alternativas:**
- `OnInfected Reject` → Rechaza directamente ❌
- `OnInfected Quarantine` → Aisla en cuarentena 📦

---

### 5️⃣ Conectar Postfix con ClamAV

**¿Qué hace?** Le ordena a Postfix que ANTES de aceptar un correo, lo escanee con ClamAV.
```bash
sudo postconf -e "smtpd_milters = unix:clamav/clamav-milter.ctl"
sudo postconf -e "non_smtpd_milters = unix:clamav/clamav-milter.ctl"
sudo postconf -e "milter_default_action = tempfail"
```

| Opción | ¿Qué hace? |
| :--- | :--- |
| `smtpd_milters` | Escanea correos que vienen de internet 🌐 |
| `non_smtpd_milters` | Escanea correos generados localmente 💻 |
| `milter_default_action = tempfail` | Si ClamAV se cae, rechaza temporalmente (es más seguro) ⏸️ |

**Explicación de tempfail:**
- Si ClamAV no responde → rechaza el correo temporalmente
- El remitente lo intenta de nuevo más tarde
- Protege contra correos infectados si el antivirus falla

---

### 6️⃣ Permisos de Seguridad

**¿Qué hace?** Permite que el usuario `postfix` acceda al grupo `clamav` sin romper la jaula de seguridad.
```bash
sudo usermod -a -G clamav postfix
```

| Elemento | Explicación |
| :--- | :--- |
| `usermod` | Modifica usuario |
| `-a` | Añade (sin eliminar otros grupos) |
| `-G clamav` | Al grupo clamav |
| `postfix` | El usuario postfix |

**🔐 Seguridad:** Postfix sigue dentro de su jaula, pero puede leer el socket de ClamAV.

---

### 7️⃣ Iniciar Servicios

**¿Qué hace?** Reinicia los tres servicios para aplicar todos los cambios y cargar las firmas de virus.
```bash
sudo systemctl restart clamav-daemon clamav-milter postfix
```

| Servicio | ¿Qué hace? |
| :--- | :--- |
| `clamav-daemon` | El antivirus que analiza archivos |
| `clamav-milter` | El filtro que habla con Postfix |
| `postfix` | El gestor de correos |

**⏳ Nota Importante:** ClamAV tarda **2-5 minutos** en arrancar porque descarga las definiciones de virus. Espera antes de hacer pruebas.

---

### 8️⃣ Verificar que Todo Funciona

**¿Qué hace?** Comprueba que los servicios estén activos y el socket esté creado correctamente.
```bash
# ✅ Ver si el socket existe
ls -l /var/spool/postfix/clamav/clamav-milter.ctl

# ✅ Ver estado de los servicios
sudo systemctl status clamav-daemon clamav-milter postfix --no-pager | grep "Active:"
```

| Comando | Resultado Esperado |
| :--- | :--- |
| `ls -l socket` | Muestra permisos y propietario del socket |
| `grep Active:` | Debe mostrar "Active: active (running)" ✅ |

**Si ves "inactive":** Algo no funcionó, revisa los logs:
```bash
sudo journalctl -u clamav-daemon -n 20
```

---

### 9️⃣ Prueba de Fuego 🔥

**¿Qué hace?** Envía un archivo de prueba EICAR (estandarizado por la industria antivirus) para validar que el sistema detecta virus correctamente.
```bash
(
echo "HELO erjoay.local"
sleep 1
echo "MAIL FROM: <test@erjoay.local>"
sleep 1
echo "RCPT TO: <root@erjoay.local>"
sleep 1
echo "DATA"
sleep 1
echo "Subject: Test Virus"
echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*'
echo "."
sleep 2
echo "QUIT"
) | telnet localhost 25
```

| Línea | Explicación |
| :--- | :--- |
| `HELO erjoay.local` | Saludamos al servidor SMTP |
| `MAIL FROM:` | Quién envía el email |
| `RCPT TO:` | Quién recibe el email |
| `DATA` | Inicio del contenido del email |
| `EICAR-STANDARD...` | El "virus" de prueba (inofensivo, pero detectado por todos los antivirus) |
| `echo "."` | Marca el fin del email |
| `telnet localhost 25` | Se conecta al puerto SMTP del servidor |

**✅ Resultado Esperado:**
```
550 5.7.1 Command rejected
```

**🎯 Significado:** 
- Código 550 = Error permanente
- ClamAV detectó el virus de prueba
- Postfix lo rechazó automáticamente
- **¡EL SISTEMA FUNCIONA!** ✨

---

## 📊 Resumen de Todo el Proceso

| Paso | Acción | Razón |
| :--- | :--- | :--- |
| 1️⃣ | IP estática | Los servidores de correo necesitan dirección fija |
| 2️⃣ | Instalar paquetes | Descargamos Postfix, ClamAV y herramientas |
| 3️⃣ | Crear socket | Punto de encuentro para que se comuniquen |
| 4️⃣ | Configurar milter | Definimos qué hacer con virus detectados |
| 5️⃣ | Integrar en Postfix | Ordenamos escanear TODOS los emails |
| 6️⃣ | Permisos | Postfix accede a ClamAV sin romper seguridad |
| 7️⃣ | Reiniciar servicios | Aplicamos todos los cambios |
| 8️⃣ | Verificar | Confirmamos que todo está activo |
| 9️⃣ | Prueba EICAR | Validamos que el antivirus detecta amenazas |

---

## 🎯 Flujo de un Email en el Sistema
```
1. Email llega → Postfix lo recibe
   ↓
2. Postfix pregunta → ¿ClamAV, es seguro este email?
   ↓
3. ClamAV escanea → Busca virus/malware
   ↓
4. ClamAV responde:
   - ✅ Si es limpio → Postfix lo acepta
   - ❌ Si tiene virus → Postfix lo rechaza
```

---

## 🛡️ ¿Qué Protege Este Sistema?

✅ **Virus en adjuntos** → ClamAV los detecta con firmas  
✅ **Malware en el cuerpo** → Análisis heurístico (comportamiento sospechoso)  
✅ **Correos infectados** → Rechazo automático antes de llegar a usuarios  
✅ **Servidor aislado** → Jaula chroot previene que hackeos afecten el sistema  
✅ **Verificación automática** → Sin intervención manual  
