# Clientes de la Intranet

Proyecto P0.0-ASIXc2gC-G05 · módulo 0379 · última revisión: 2026-11-10

Dos clientes en la red Intranet (`ASIXc2-ITB5a`, `192.168.20.0/24`): uno Linux (Ubuntu, hostname `cliente`) y uno Windows. Los dos reciben la configuración por DHCP del router `R-05` (Kea), sin IP fija.

| Cliente | Sistema | Red de Isard | IP | Asignación |
|---|---|---|---|---|
| `cliente` | Ubuntu | `ASIXc2-ITB5a` | Dentro de `192.168.20.100-200` | DHCP |
| Windows | Windows | `ASIXc2-ITB5a` | `192.168.20.103` | DHCP |

Configuración que reciben: máscara `255.255.255.0`, puerta de enlace `192.168.20.1`, DNS `192.168.20.1` y sufijo `g05.itb`.

## 1. Cliente Windows

Comprobaciones hechas el 2026-11-10 en PowerShell:

```powershell
ipconfig /all
nslookup www
ping R-05
ping 8.8.8.8
```

| Prueba | Resultado |
|---|---|
| `ipconfig /all` | IP `192.168.20.103/24`, puerta de enlace `192.168.20.1`, sufijo DNS `g05.itb` |
| `nslookup www` | Servidor `192.168.20.1`; `www` es alias de `W-05.g05.itb` → `192.168.10.10` |
| `ping R-05` | Responde `192.168.20.1`: 4 enviados, 4 recibidos, 0% perdidos (resuelve `R-05.g05.itb`) |
| `ping 8.8.8.8` | Responde: 0% perdidos, salida a internet por el NAT del router |
| Explorador de archivos, `ftp://192.168.10.10` | Muestra los ficheros del servidor FTP: `ftp.txt`, `index.html`, `index.php` y `prueba1` |

> Capturas: ya hecha la de `ipconfig`, `nslookup`, `ping` y el FTP en el Explorador. Pendiente: la web `http://www.g05.itb` en el navegador.

## 2. Cliente Linux

Comprobaciones hechas el 2026-11-10:

| Prueba | Resultado |
|---|---|
| IP automática | Recibida dentro de `192.168.20.100-200`, con puerta de enlace y DNS `192.168.20.1` |
| `ping R-05` y `ping R` | Responden `192.168.20.1` |
| `ping google.com` | Responde: el DNS reenvía y el NAT funciona |
| `nslookup R-05` y `nslookup www` | Resuelven a `192.168.20.1` y a `192.168.10.10` |
| `curl -I http://W-05` | `HTTP/1.1 200 OK` (Apache 2.4.69, PHP 8.2.35) |
| `ftp 192.168.10.10` con `bchecker` | `ls`, `put prueba1` y `get index.html` correctos, en modo pasivo (puertos 21103, 21105 y 21108) |
| `ssh bchecker@192.168.20.1` | Entra al router |

Herramientas que hay que instalar en la plantilla mínima de Isard:

```bash
sudo apt install -y iputils-ping curl ftp dnsutils netcat-openbsd
```

## 3. Acceso a la web

Desde cualquiera de los dos clientes: `http://www.g05.itb`, `http://W-05` o `http://192.168.10.10`.

## 4. Pendiente

- Probar la web en el navegador de Windows y hacer la captura.
- Probar `bchecker` en los dos clientes (tarea 6 del Sprint 3).
