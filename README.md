Cubietruck
==========

Playbooks Ansible para configurar mi cubietruck.

Bootstrap
---------

  - El primer paso es tener un usuario adecuado para ejecutar *ansible*. Si el usuario no existe aún,

```
# Crear usuario "ansible"
useradd -m -d /home/ansible -s /bin/bash -a -G sudo ansible

# Asignarle al usuario una clave RSA, si no la tiene
sudo -u ansible -i
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

  - Una vez preparado el usuario con el que se van a ejecutar los scripts, lo siguiente es crear el virtualenv y obtener los playbooks.

```
# Algunas de las dependencias de Ansible requieren Python-dev
sudo apt-get install python-virtualenv python-dev python-pip

# Crear el virtualenv y entrar en el directorio
virtualenv --system-site-packages cubie && cd cubie

# Clonar el proyecto
git clone https://github.com/rjrivero/Cubietruck.git inventory

# Instalar Ansible. Se recomienda instalar directamente desde github,
# en el propio virtualenv
git clone git://github.com/ansible/ansible.git --recursive

# Iniciar el entorno
source inventory/init.sh -U
```

Variables de entorno
--------------------

Los playbooks de este proyecto utilizan una serie de variables de entorno:

  - *WLAN0_ADDRESS*: Dirección IP a asignar a la interfaz wlan0 de la Cubie.
  - *WLAN0_SSID*: SSID al que conectar la wlan0.
  - *WLAN0_PSK*: PSK WPA2 a usar para la wlan0.
  - *LASTFM_USER*: Nombre de usuario de last.fm.
  - *LASTFM_PASSWORD*: Clave de last.fm.
  - *TRANSMISSION_PASSWORD*: Clave de torrent / transmission.
  - *SAMBA_PASSWORD*: Clave para usuario SMB/CIFS
  - *VNC_PASSWORD*: Clave para usuario VNC
  - *SPEAKER_MAC*: Direccion Bluetooth MAC de los altavoces con los que parear la Cubie.
  - *CF_DOMAIN*: Nombre del dominio delegado a CloudFlare.
  - *CF_RECORD*: iRecord A dentro del dominio que apunta a este host.
  - *CF_USERNAME*: Nombre de usuario (email) de CloudFlare.
  - *CF_APIKEY*: Clave API para CloudFlare.

Variables obsoletas:

  - *NOIP_DOMAIN*: Dominio dinámico registrado en noip.org.
  - *NOIP_USERNAME*: Nombre de usuario de noip.
  - *NOIP_PASSWORD*: Clave de noip.
  - *DD_DOMAIN*: Dominio dinámico registrado en Don Dominio.
  - *DD_USERNAME*: Nombre de usuario de Don Dominio.
  - *DD_APIKEY*: Clave API para Don Dominio.

Para facilitar trabajar con ellas, si existe en el directorio un fichero llamado **environment**, *init.sh* lo invoca para que pueda establecer las variables de entorno.

Seguridad
---------

Los playbooks de este proyecto utilizan algunos datos sensibles:

  - Las variables de entorno descritas arriba, algunas de las cuales contienen nombres de usuario y contraseñas.
  - El certificado del sitio web desde el cual se accede a todos los servicios (**playbooks/files/cert.crt)**.
  - La clave de dicho certificado (**playbooks/files/cert.key**)

Para protegr la información, esos ficheros no se suben al repositorio. En su lugar, se suben las versiones cifradas con [ansible-vault](http://docs.ansible.com/ansible/playbooks_vault.html):

```
# Crear un fichero cifrado
ansible-vault create environment.vault

# Editar el fichero cifrado
ansible-vault edit environment.vault

# Descifrar el fichero
ansible-vault decrypt environment.vault environment
```

El script **init.sh** descifra los ficheros en local al preparar el entorno. Si los ficheros **environment**, **cert.key** y **cert.crt** no existen, init.sh intenta descifrarlos desde las versiones *.vault* correspondientes.

**NOTA**: Para que el certificado funcione adecuadamente con seafile y nginx, debe contener toda la cadena de certificacion, desde la el certificado raiz. Se puede hacer simplemente concatenando con **cat** los certificados, desde el de site al certificado root, pasando por las intermedias:

```
cat site.crt intermediate1.crt intermediate2.crt root.crt > cert.crt
```
 
Configuración de la tarjeta SD
------------------------------

Una vez grabada la imagen de [armbian](http://www.armbian.com/cubietruck/) en la tarjeta SD, la configuración inicial se hace con el playbook **[prepare.yml](playbooks/prepare.yml)**.

```
# La tarjeta SD debe estar montada en la ruta **/mnt/cubie**
sudo mkdir -p /mnt/cubie
sudo mount /dev/<sd card device> /mnt/cubie

# Iniciar el virtualenv
source ./init.sh

# Ejecutar el playbook, preguntar el password de sudo
ansible-playbook playbooks/prepare.yml -K
```

Configuracion de la cubie
-------------------------

Despues de haber preparado la tarjeta SD y haber arrancado la cubie con ella, la configuración se completa ejecutando el playbook **[deploy.yml](playbooks/deploy.yml)**.

```
# Iniciar el virtualenv
source ./init.sh

# Ejecutar el playbook
ansible-playbook playbooks/deploy.yml
```
