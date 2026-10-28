# Documentación de Configuración: Máquina W-05 (Servidor Web DMZ)

En esta guía se documenta detalladamente el procedimiento técnico para el despliegue y puesta en marcha del servidor web en la máquina virtual **W-05**, ubicada en la zona desmilitarizada (DMZ) del **Proyecto G05**. El proceso abarca la personalización del sistema, el direccionamiento de red estático, el despliegue contenerizado de Nginx con datos persistentes, la activación del servicio de administración remota OpenSSH y la seguridad perimetral local mediante cortafuegos.

---

## 1. Identificación del Sistema (Hostname)

Para asegurar una correcta identificación y gestión dentro de la topología de red del proyecto, se establece el nombre de host oficial de la máquina como `W-05`.

![cambio de nombre .png](./img/cambio%20de%20nombre%20.png)

---

## 2. Creación del Usuario de Administración `bchecker`

Se crea el usuario del sistema `bchecker` asignándole su directorio personal `/home/bchecker`, la consola interactiva Bash como shell por defecto y la contraseña estándar definida para la infraestructura (`bchecker121`). Además, se le otorgan privilegios administrativos incorporándolo al grupo `sudo`.

![creación del usuario bchecker.png](./img/creaci%C3%B3n%20del%20usuario%20bchecker.png)

---

## 3. Configuración de Red Estática (Netplan)

Para garantizar la conectividad dentro de la DMZ (`itb5`), se edita el archivo de red `/etc/netplan/01-netcfg.yaml` asignando a la interfaz `enp1s0` la dirección IP estática `192.168.10.10/24`. Se establece como puerta de enlace predeterminada y DNS la dirección del router central `192.168.10.1` (R-05).

![configuración de red.png](./img/configuraci%C3%B3n%20de%20red.png)

---

## 4. Instalación del Motor Docker y Gestión de Permisos

Se instala el servicio Docker (`docker.io`) en el sistema operativo base. Con el objetivo de permitir que el usuario `bchecker` administre y despliegue contenedores de manera ágil sin requerir la elevación constante de privilegios mediante `sudo`, se le añade al grupo local `docker`.

![instalación de docker.png](./img/instalaci%C3%B3n%20de%20docker.png)

![añadir usuario bchecker al grupo docker.png](./img/a%C3%B1adir%20usuario%20bchecker%20al%20grupo%20docker.png)

---

## 5. Preparación del Almacenamiento Local y Creación del Sitio Web

Para lograr la persistencia de los datos del servidor web fuera del ciclo de vida del contenedor, se crea la estructura de directorios `/var/www/html` en el sistema anfitrión. A continuación, se inicializa el archivo `index.html` con la plantilla HTML personalizada para la identificación del proyecto G05.

![creacion del archivo html.png](./img/creacion%20del%20archivo%20html.png)

Se edita el archivo con la herramienta `nano` para incorporar el código que servirá Nginx al recibir peticiones HTTP.

![configuración archivo html.png](./img/configuraci%C3%B3n%20archivo%20html.png)

---

## 6. Despliegue del Contenedor Web Nginx con Volumen Mapeado

Se realiza la puesta en marcha del contenedor web denominado `web-container` utilizando la imagen ligera `nginx:alpine`. Para garantizar la alta disponibilidad, se configura con la política `--restart always`. 

Se realiza el reenvío del puerto `80/TCP` del host hacia el contenedor y se aplica un bind mount (`-v`) desde `/var/www/html` de la VM hacia `/usr/share/nginx/html` en el contenedor, permitiendo que Nginx sirva directamente el código fuente generado.

![creación contenedor.png](./img/creaci%C3%B3n%20contenedor.png)

---

## 7. Comprobación y Verificación del Servicio Web

Desde una máquina cliente ubicada dentro de la red corporativa, se efectúa la prueba de acceso abriendo el navegador web e introduciendo la IP del servidor web en la DMZ (`http://192.168.10.10`). Se confirma el correcto enrutamiento entre redes y el renderizado del archivo HTML personalizado.

![comprobación web.png](./img/comprobaci%C3%B3n%20web.png)

---

## 8. Instalación y Puesta en Marcha de OpenSSH Server

Para posibilitar la administración y mantenimiento remoto seguro de la máquina virtual `W-05` desde otros puntos de la red, se instala el paquete `openssh-server` en la VM base y se habilita el demonio SSH para que inicie automáticamente junto con el sistema.

![instalacion de ssh.png](./img/instalacion%20de%20ssh.png)

---

## 9. Configuración del Firewall del Sistema (UFW)

Para securizar la máquina `W-05` dentro de la DMZ, se habilita el cortafuegos interno `ufw` permitiendo explícitamente únicamente el tráfico necesario para el funcionamiento del servidor web (puerto `80/TCP`) y para la administración remota (puerto `22/TCP`).

![configuracion firewall.png](./img/configuracion%20firewall.png)
