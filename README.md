Práctica 1: Instalación y Gestión Web con Webmin
# 1. Instalar dependencias necesarias
sudo apt update
sudo apt install -y software-properties-common apt-transport-https wget

# 2. Descargar e instalar la clave GPG de Webmin
wget -q http://www.webmin.com/jcameron-key.asc -O- | sudo apt-key add -

# 3. Añadir el repositorio de Webmin a la lista de fuentes
sudo add-apt-repository "deb https://download.webmin.com/download/repository sarge contrib"

# 4. Actualizar la lista de paquetes e instalar Webmin
sudo apt update
sudo apt install webmin
Acceso: Una vez instalado, accede a la interfaz desde cualquier navegador en: https://[IP_DE_TU_SERVER]:10000. Usa las credenciales de tu usuario de Linux (root o sudo).

2. Demostración y Documentación 
Para completar la práctica, debes documentar con capturas de pantalla o descripciones el uso de las siguientes funciones:

Administración de Usuarios: Creación/Eliminación de un usuario de prueba.

Gestión de Servicios: Iniciar, detener o reiniciar el servicio SSH.

Configuración de Red: Captura de la configuración de IP estática.

Administración de Software: Búsqueda e instalación de una herramienta (ej. htop).

Monitorización del Sistema: Captura de la vista de rendimiento (CPU, RAM).

Práctica 2: Despliegue de VM con Terraform en DigitalOcean
# 1. Instalar Terraform (siguiendo la documentación oficial)
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# 2. Configurar la autenticación de DigitalOcean
# Reemplaza [TU_TOKEN_DO] con el token de API que generaste.
export DIGITALOCEAN_TOKEN="[TU_TOKEN_DO]"

Archivo de Configuración (main.tf)
Crea un archivo llamado main.tf y pega el siguiente código. Asegúrate de reemplazar [llavePublicaDeTuServerAnsible] con el ID o nombre de la llave SSH cargada en DigitalOcean.

Terraform

# Proveedor para DigitalOcean
terraform {
  required_providers {
    digitalocean = {
      source = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

# Configuración del proveedor (usa el token exportado)
provider "digitalocean" {}

# Recurso: Despliegue de la VM (Droplet)
resource "digitalocean_droplet" "os3_client" {
  image  = "ubuntu-22-04-x64"
  name   = "OS3vm"
  region = "nyc1"
  size   = "s-1vcpu-1gb"
  ssh_keys = [
    "[llavePublicaDeTuServerAnsible]" # ID o nombre de la llave SSH
  ]
}
Despliegue de la VM
terraform init
terraform plan
terraform apply --auto-approve

Práctica 3: Instalación y Configuración de Ansible
sudo apt update && sudo apt install -y ansible python3-pip
sudo pip3 install pywinrm --break-system-packages
ssh-keygen -t rsa -b 4096
Configuración del Cliente Linux (DigitalOcean)
Crear Usuario: En el cliente Linux: sudo adduser ansible y sudo usermod -aG sudo ansible.

Copiar Llave SSH: Desde el Kali Controller: ssh-copy-id ansible@IP_DE_LINUX.

3.3. Configuración del Cliente Windows
Ejecuta el siguiente script en PowerShell como Administrador en la VM de Windows.

PowerShell

# 1. Forzar la red a Privada (Resuelve problemas de Firewall)
Get-NetConnectionProfile | Set-NetConnectionProfile -NetworkCategory Private

# 2. Habilitar y configurar WinRM para Ansible
Enable-PSRemoting -Force
Set-Item WSMan:\localhost\Service\Auth\Basic -Value $true
Set-Item WSMan:\localhost\Service\AllowUnencrypted -Value $true
Restart-Service WinRM

# 3. Crear usuario 'ansible' y darle permisos de administrador
$Password = ConvertTo-SecureString "tu_contraseña_windows" -AsPlainText -Force
New-LocalUser -Name "ansible" -Password $Password -FullName "Ansible Admin"
Add-LocalGroupMember -Group "Administrators" -Member "ansible"
3.4. Creación del Inventario (/etc/ansible/hosts)
Crea y edita el archivo en tu VM Server (Kali). ¡Reemplaza las IPs y la contraseña!

Bash

sudo bash -c 'cat > /etc/ansible/hosts <<EOF
[linux]
ansible_client01 ansible_host=IP_DE_LINUX

[linux:vars]
ansible_user=ansible

[win]
ansible_client02 ansible_host=IP_DE_WINDOWS

[win:vars]
ansible_user=ansible
ansible_password=tu_contraseña_windows
ansible_port=5985
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
EOF'
3.5. Prueba de Conexión Final
Ejecuta estos comandos en tu Kali Controller para validar la práctica:

Bash

# Prueba de ping para Linux (vía SSH)
ansible linux -m ping

# Prueba de ping para Windows (vía WinRM)
ansible win -m win_ping
