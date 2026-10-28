# Documentación de Configuración: Máquina W-05 (Servidor Web DMZ)

En este documento se describe el proceso paso a paso para la preparación, configuración de red, despliegue del contenedor web y activación del servicio SSH en la VM **W-05**.

---

## 1. Cambio del Hostname de la máquina

Para identificar correctamente la máquina dentro de la infraestructura del proyecto, se cambia el nombre del host a `W-05`.

```bash
sudo hostnamectl set-hostname W-05
```

> **Ruta de la captura:** `docs/05-web/img/1.Cambio de nombre .png`

![1. Cambio de nombre](./img/1.Cambio%20de%20nombre%20.png)

---

## 2. Creación del usuario obligatorio `bchecker`

Se crea el usuario `bchecker` con su directorio personal, shell `/bin/bash`, la contraseña global del proyecto (`bchecker121`) y privilegios de administración en el sistema.

```bash
isard@ubuntu2404:~$ sudo useradd -m -s /bin/bash bchecker
isard@ubuntu2404:~$ echo "bchecker:bchecker121" | sudo chpasswd
isard@ubuntu2404:~$ sudo usermod -aG sudo bchecker
```

> **Ruta de la captura:** `docs/05-web/img/2.Creación del usuario bchecker.png`

![2. Creación del usuario bchecker](./img/2.Creaci%C3%B3n%20del%20usuario%20bchecker.png)

---

## 3. Configuración del direccionamiento IP estático (Netplan)

Se configura la interfaz de red `enp1s0` conectada a la red **DMZ** (`itb5`) con la dirección IP fija `192.168.10.10/24` y puerta de enlace `192.168.10.1` (Router R-05).

Contenido del archivo `/etc/netplan/01-netcfg.yaml`:

```yaml
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
```

> **Ruta de la captura:** `docs/05-web/img/3.Configuración de red.png`

![3. Configuración de red](./img/3.Configuraci%C3%B3n%20de%20red.png)

---

## 4. Asignación del usuario `bchecker` al grupo Docker

Se añade a `bchecker` al grupo `docker` para permitir la administración de contenedores sin necesidad de utilizar `sudo`.

```bash
bchecker@W-05:~$ sudo usermod -aG docker bchecker
```

> **Ruta de la captura:** `docs/05-web/img/Añadir usuario bchecker al grupo docker.png`

![Añadir usuario bchecker al grupo docker](./img/A%C3%B1adir%20usuario%20bchecker%20al%20grupo%20docker.png)

---

## 5. Despliegue del contenedor web (Nginx)

Se descarga la imagen ligera `nginx:alpine` y se inicia el contenedor vinculando el puerto 80 de la máquina virtual con reinicio automático.

```bash
bchecker@W-05:~$ docker run -d --name web-container --restart always -p 80:80 nginx:alpine
```

Salida del comando y descarga de capas:

```text
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
b9b388a21ecf: Pull complete
9b0f7dfe35b2: Pull complete
657023555a2a: Pull complete
26185d08a3ec: Pull complete
7cc330898115: Pull complete
7b229517ed22: Pull complete
2ec5fb117ffa: Pull complete
12b85cff06fd: Pull complete
a4d5b1f57abc: Download complete
acc2d54c3743: Download complete
Digest: sha256:a3554c0d0b48177b1413c26e01f695334428020d9df01243fbae3e11cfd17417
Status: Downloaded newer image for nginx:alpine
bccdd2bea65c3ddd456f7f2c80e2cafd065dc820f2d6f0c21d03f5c647507691
```

> **Ruta de la captura:** `docs/05-web/img/Creación del contenedor web.png`

![Creación del contenedor web](./img/Crea%C3%A7i%C3%B3n%20del%20contenedor%20web.png)

---

## 6. Instalación y activación del servicio OpenSSH

Se instala y activa el servicio OpenSSH Server directamente en el sistema base de la VM (fuera de Docker) para la administración remota de la máquina.

```bash
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
```

> **Ruta de la captura:** `docs/05-web/img/Instalacion de ssh.png`

![Instalación de SSH](./img/Instalacion%20de%20ssh.png)
