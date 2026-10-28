# Documentación de Configuración: Máquina W-05 (Servidor Web DMZ)

En este documento se describe el proceso paso a paso para la preparación, configuración de red, instalación de Docker, creación del contenido HTML local, despliegue del contenedor web con volumen persistente, comprobación del servicio y activación del servicio SSH en la VM **W-05**.

---

## 1. Cambio del Hostname de la máquina

Para identificar correctamente la máquina dentro de la infraestructura del proyecto, se cambia el nombre del host a `W-05`.

sudo hostnamectl set-hostname W-05

### Captura: Cambio de Hostname
![cambio de nombre .png](./img/cambio%20de%20nombre%20.png)

---

## 2. Creación del usuario obligatorio `bchecker`

Se crea el usuario `bchecker` con su directorio personal, shell `/bin/bash`, la contraseña global del proyecto (`bchecker121`) y privilegios de administración en el sistema.

isard@ubuntu2404:~$ sudo useradd -m -s /bin/bash bchecker
isard@ubuntu2404:~$ echo "bchecker:bchecker121" | sudo chpasswd
isard@ubuntu2404:~$ sudo usermod -aG sudo bchecker

### Captura: Creación del usuario bchecker
![creación del usuario bchecker.png](./img/creaci%C3%B3n%20del%20usuario%20bchecker.png)

---

## 3. Configuración del direccionamiento IP estático (Netplan)

Se configura la interfaz de red `enp1s0` conectada a la red **DMZ** (`itb5`) con la dirección IP fija `192.168.10.10/24` y puerta de enlace `192.168.10.1` (Router R-05).

Contenido del archivo `/etc/netplan/01-netcfg.yaml`:

network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: false
      addresses:
        - 192.168.10.10/24
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses: [192.168.10.1]

### Captura: Configuración de red (Netplan)
![configuración de red.png](./img/configuraci%C3%B3n%20de%20red.png)

---

## 4. Instalación de Docker y asignación de permisos al usuario `bchecker`

Se instala el motor de Docker en la máquina virtual y se añade al usuario `bchecker` al grupo `docker` para permitir la administración de contenedores sin requerir `sudo`.

bchecker@W-05:~$ sudo apt update && sudo apt install -y docker.io
bchecker@W-05:~$ sudo usermod -aG docker bchecker

### Captura: Instalación de Docker
![instalación de docker.png](./img/instalaci%C3%B3n%20de%20docker.png)

### Captura: Añadir usuario bchecker al grupo docker
![añadir usuario bchecker al grupo docker.png](./img/a%C3%B1adir%20usuario%20bchecker%20al%20grupo%20docker.png)

---

## 5. Preparación del directorio y creación de la página web local

Se crea el directorio `/var/www/html` en la VM `W-05`, el archivo `index.html` y se edita para incluir el contenido personalizado del proyecto.

bchecker@W-05:~$ sudo mkdir -p /var/www/html
bchecker@W-05:~$ sudo touch /var/www/html/index.html

### Captura: Creación del archivo html
![creacion del archivo html.png](./img/creacion%20del%20archivo%20html.png)

Edición del archivo con `nano`:

bchecker@W-05:~$ sudo nano /var/www/html/index.html

Contenido del archivo `/var/www/html/index.html`:

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Proyecto G05 - DMZ Web</title>
</head>
<body>
    <h1>Servidor Web W-05</h1>
    <p>Grupo G05</p>
</body>
</html>

### Captura: Configuración del archivo html
![configuración archivo html.png](./img/configuraci%C3%B3n%20archivo%20html.png)

---

## 6. Despliegue del contenedor web (Nginx) con volumen

Se arranca el contenedor Nginx vinculando el puerto 80 del host con el del contenedor y mapeando la carpeta local `/var/www/html` hacia `/usr/share/nginx/html`:

bchecker@W-05:~$ docker run -d --name web-container --restart always -p 80:80 -v /var/www/html:/usr/share/nginx/html nginx:alpine
384ffb379486b11e7bc564a2da87ec1f7e0c1de5eb8976382dd523265e659947

### Captura: Creación del contenedor
![creación contenedor.png](./img/creaci%C3%B3n%20contenedor.png)

---

## 7. Verificación del sitio web desde el cliente

Se accede desde el navegador web apuntando a la dirección IP de la VM (`http://192.168.10.10`), comprobando que se carga la página web con el título `Proyecto G05 - DMZ Web` y el contenido configurado.

### Captura: Comprobación web
![comprobación web.png](./img/comprobaci%C3%B3n%20web.png)

---

## 8. Instalación y activación del servicio OpenSSH

Se instala y activa el servicio OpenSSH Server directamente en el sistema base de la VM (fuera de Docker) para la administración remota.

bchecker@W-05:~$ sudo apt install openssh-server
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
openssh-server is already the newest version (1:9.6p1-3ubuntu13.20).
0 upgraded, 0 newly installed, 0 to remove and 48 not upgraded.

bchecker@W-05:~$ sudo systemctl enable --now ssh
Synchronizing state of ssh.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable ssh
Created symlink /etc/systemd/system/sshd.service -> /usr/lib/systemd/system/ssh.service.
Created symlink /etc/systemd/system/multi-user.target.wants/ssh.service -> /usr/lib/systemd/system/ssh.service.

### Captura: Instalación de SSH
![instalacion de ssh.png](./img/instalacion%20de%20ssh.png)
