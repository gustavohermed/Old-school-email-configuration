# Old School Email Server (Postfix + Dovecot)

En este proyecto he configurado un servidor de correo electrónico clásico utilizando
**Postfix** y **Dovecot**, desplegado mediante **Vagrant**
El objetivo ha sido comprender el funcionamiento interno de un servidor de correo, configurando manualmente cada uno de sus componentes.

## 🧱 Arquitectura del proyecto

El entorno está formado por dos máquinas virtuales:

- **Servidor DNS**
  - Resolución del dominio interno
  - Registros A, CNAME y MX necesarios para el correo
- **Servidor de correo**
  - Postfix como servidor SMTP
  - Dovecot como servidor IMAP y de autenticación
  - Buzones en formato **Maildir**
- **Cliente**
  - Thunderbird configurado manualmente desde el equipo anfitrión

## 🔧 Cambios realizados y justificación

Durante la realización de la práctica he tenido que realizar varios ajustes y correcciones
sobre la configuración inicial para cumplir con los requisitos del enunciado.

### 1. Configuración de Maildir en Dovecot
Inicialmente Dovecot utilizaba el formato mbox por defecto.  
He modificado la configuración para utilizar **Maildir**, tal y como exige el enunciado.

### 2. Habilitación de SMTP autenticado (submission)
He configurado Postfix para permitir el envío autenticado de correos a través del
puerto **587 (submission)** utilizando SASL con Dovecot.  
Esto ha sido necesario para que los clientes de correo puedan enviar mensajes de forma segura y autenticada.

### 3. Uso de certificados TLS autofirmados
He generado certificados TLS autofirmados para asegurar las conexiones IMAP y SMTP mediante STARTTLS.  

sudo openssl req -x509  -nodes -days 365  -newkey rsa:2048  -keyout /etc/ssl/private/mail.key  -out /etc/ssl/certs/mail-cert.pem

### 5. Creación de usuarios locales
Los usuarios de correo se han creado como **usuarios locales del sistema** en el servidor de correo, siendo utilizados por Dovecot tanto para la autenticación IMAP como para el envío autenticado SMTP.

Los usuarios han sido: Manolo y Laura, y he hecho una comprobación de que manolo le envía un correo a Laura, y Laura lo recibe, las capturas estan en el documento capturas.pdf

sudo useradd -m  manolo
sudo passwd manolo

sudo doveadm mailbox create -u manolo INBOX
ls /home/manolo/Maildir
sudo doveadm user manolo


---

## 📁 Cambios realizados en la máquina mail

A continuación se detallan los principales ficheros modificados en la máquina
**mail** y el motivo de cada cambio:

### Instalacion postfix y dovecot

sudo apt update
sudo apt install postfix mailutils

sudo apt install dovecot-core dovecot-imapd dovecot-pop3d

### `/etc/postfix/main.cf`
Se modificó este fichero para definir los parámetros básicos del servidor SMTP:
- Nombre del host y dominio
- Uso de buzones Maildir
- Redes permitidas
- Activación de SMTP AUTH mediante Dovecot



home_mailbox = Maildir/

mynetworks = 127.0.0.0/8, 192.168.57.0/24, 10.112.0.0/16

smtpd_use_tls=yes
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key



Estos cambios permiten el envío autenticado de correos desde clientes externos.

### `/etc/dovecot/conf.d/10-mail.conf`
Se modificó para definir el uso de buzones **Maildir** en el directorio personal de cada usuario, sustituyendo el formato mbox por defecto.

mail_location = maildir:~/Maildir

### `/etc/dovecot/conf.d/10-auth.conf`
Se ajustó la configuración de disable_plaintext_auth a = yes.

disable_plaintext_auth = yes

### `/etc/dovecot/conf.d/10-ssl.conf`
Se configuró el uso de TLS en Dovecot indicando la ruta del certificado y la clave autofirmados generados para el servidor de correo.

ssl = yes
ssl_cert = </etc/ssl/certs/mail-cert.pem
ssl_key = </etc/ssl/private/mail.key

### Certificados TLS
Se generaron certificados autofirmados mediante OpenSSL para asegurar las conexiones IMAP y SMTP, aceptándose posteriormente las excepciones de seguridad en Thunderbird.

---

### Hay que configurar el /etc/hosts
ej: 192.168.57.10  dns.gustavo.test
    192.168.57.20  mail.gustavo.test

## 🧪 Comprobaciones realizadas

He comprobado el correcto funcionamiento del sistema mediante:

- Verificación de los puertos en escucha (25, 143 y 587)
- Autenticación IMAP usando STARTTLS
- Envío autenticado mediante SMTP
- Envío y recepción de correos entre distintos usuarios desde Thunderbird
- Aceptación de certificados autofirmados en el cliente

Todas estas pruebas quedan reflejadas en las capturas incluidas.

## 📎 Evidencias

Las capturas que justifican la configuración y las pruebas realizadas se encuentran en:

- `capturas.pdf`

Que se encuentra en los archivos enviados a la tarea.

## ✅ Estado final

✔ Servidor DNS operativo  
✔ Postfix correctamente configurado  
✔ Dovecot usando Maildir y TLS  
✔ SMTP autenticado funcionando  
✔ Clientes Thunderbird enviando y recibiendo correos  

La práctica se ha completado correctamente y cumple con los requisitos del enunciado.
