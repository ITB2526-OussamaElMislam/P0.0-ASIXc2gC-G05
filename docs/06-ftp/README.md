# Configuración del Servicio FTP (DMZ — W-05)

## 1. Descripción de la Arquitectura
El servidor FTP se ha desplegado utilizando una arquitectura contenerizada en Docker sobre la máquina virtual **W-05** (`192.168.10.10`). El objetivo principal es permitir la gestión remota y la actualización de los archivos de la aplicación web en tiempo real.

* **Imagen Docker**: `fauria/vsftpd`
* **Directorio Persistente**: `/var/www/html` (mapeado al directorio raíz del usuario FTP)
* **Modo de Transferencia**: Pasivo (PASV)

---

## 2. Parámetros de Red y Firewall (UFW)
Para permitir las conexiones de control y la transferencia de datos en modo pasivo sin interferir con la seguridad de la DMZ, se han configurado los siguientes puertos en el firewall **UFW** de W-05:

| Servicio | Puerto / Rango | Protocolo | Descripción |
| :--- | :--- | :--- | :--- |
| **FTP Control** | `21` | TCP | Establecimiento de sesión y comandos |
| **FTP Pasivo** | `21100-21110` | TCP | Transferencia de datos y listado de archivos |

Comandos de firewall aplicados:
```bash
sudo ufw allow 21/tcp comment 'FTP Command Port'
sudo ufw allow 21100:21110/tcp comment 'FTP Passive Ports'
sudo ufw reload
