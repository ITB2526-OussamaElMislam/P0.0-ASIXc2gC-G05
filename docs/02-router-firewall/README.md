# Router y firewall (R-05)

Proyecto P0.0-ASIXc2gC-G05 · módulo 0379 · responsable: Oussama · última revisión: 2026-11-10

El router `R-05` es una VM de Isard con Ubuntu Server 24.04 y tres interfaces. Es el único punto de paso entre NAT, DMZ e Intranet, y aplica el firewall con nftables.

| Interfaz | Red de Isard | Red del proyecto | IP |
|---|---|---|---|
| `enp1s0` | default | NAT (internet) | Por DHCP de Isard |
| `enp2s0` | `ASIXc2-ITB5` | DMZ | `192.168.10.1/24` |
| `enp3s0` | `ASIXc2-ITB5a` | Intranet | `192.168.20.1/24` |

## 1. Hostname y usuario

```bash
sudo hostnamectl set-hostname R-05
sudo nano /etc/hosts      # línea: 127.0.1.1 R-05
echo 'preserve_hostname: true' | sudo tee /etc/cloud/cloud.cfg.d/99-hostname.cfg

sudo useradd -m -s /bin/bash bchecker
echo 'bchecker:<contraseña del enunciado>' | sudo chpasswd
```

## 2. Direcciones IP (netplan)

```yaml
network:
  version: 2
  ethernets:
    enp1s0:
      dhcp4: true
    enp2s0:
      dhcp4: false
      addresses: [192.168.10.1/24]
    enp3s0:
      dhcp4: false
      addresses: [192.168.20.1/24]
```

```bash
sudo netplan try
ip -br a
```

> Captura pendiente: salida de `ip -br a` con las tres interfaces.

## 3. Reenvío de paquetes

Fichero `/etc/sysctl.d/99-forward.conf`:

```
net.ipv4.ip_forward=1
```

```bash
sudo sysctl --system
cat /proc/sys/net/ipv4/ip_forward
```

El resultado debe ser `1`.

## 4. NAT y firewall (nftables)

Fichero `/etc/nftables.conf`. La política por defecto es **denegar** en `input` y `forward`, y solo se permite lo necesario. El NAT (masquerade por `enp1s0`) está en la tabla `ip nat`.

### 4.1 Qué se permite

| Origen | Destino | Permitido |
|---|---|---|
| Intranet y DMZ | Internet | Todo |
| Intranet | DMZ | Puertos 80, 21 y 22, el rango pasivo del FTP (21100-21110) y ping |
| `W-05` (DMZ) | `B-05` (Intranet) | Solo el puerto 3306 |
| DMZ e Intranet | Router | DNS (53); DHCP (67) solo desde la Intranet |
| Red de Isard (`enp1s0`) e Intranet | Router | SSH (22). La DMZ no puede entrar por SSH al router |
| Cualquier otro tráfico | | Denegado y registrado en el log (`R-05 input drop` y `R-05 forward drop`) |

Las respuestas a conexiones ya permitidas pasan por `ct state established,related`.

### 4.2 Configuración

```
#!/usr/sbin/nft -f

flush ruleset

table inet filter {

	chain input {
		type filter hook input priority filter; policy drop;

		iifname "lo" accept
		ct state established,related accept
		ct state invalid drop

		# Ping al router
		ip protocol icmp accept

		# DNS (bind9) desde la DMZ y la Intranet
		iifname { "enp2s0", "enp3s0" } udp dport 53 accept
		iifname { "enp2s0", "enp3s0" } tcp dport 53 accept

		# DHCP (Kea) solo en la Intranet
		iifname "enp3s0" udp dport 67 accept

		# SSH al router solo desde la red de Isard y la Intranet (no desde la DMZ)
		iifname { "enp1s0", "enp3s0" } tcp dport 22 accept

		counter limit rate 5/minute log prefix "R-05 input drop: "
	}

	chain forward {
		type filter hook forward priority filter; policy drop;

		ct state established,related accept
		ct state invalid drop

		# Intranet y DMZ hacia internet
		iifname "enp3s0" oifname "enp1s0" counter accept
		iifname "enp2s0" oifname "enp1s0" counter accept

		# Intranet hacia la DMZ: web, FTP y SSH (y ping)
		iifname "enp3s0" oifname "enp2s0" tcp dport { 21, 22, 80 } counter accept
		iifname "enp3s0" oifname "enp2s0" tcp dport 21100-21110 counter accept
		iifname "enp3s0" oifname "enp2s0" icmp type echo-request counter accept

		# DMZ hacia Intranet: solo la web (W-05) a la BBDD (B-05) por el 3306
		iifname "enp2s0" oifname "enp3s0" ip saddr 192.168.10.10 ip daddr 192.168.20.10 tcp dport 3306 counter accept

		counter limit rate 5/minute log prefix "R-05 forward drop: "
	}

	chain output {
		type filter hook output priority filter; policy accept;
	}
}

table ip nat {
	chain postrouting {
		type nat hook postrouting priority 100; policy accept;
		oifname "enp1s0" masquerade
	}
}
```

Las reglas con `counter` cuentan los paquetes que pasan por ellas, lo que sirve para comprobar el firewall con `sudo nft list ruleset`.

El contenedor FTP de `W-05` usa modo pasivo con el rango 21100-21110, por eso ese rango está abierto en `forward` (añadido el 2026-11-10).

### 4.3 Aplicar y dejar permanente

```bash
sudo cp /etc/nftables.conf /etc/nftables.conf.bak
sudo nft -c -f /etc/nftables.conf        # comprueba la sintaxis sin aplicar
sudo bash -c 'nft -f /etc/nftables.conf; echo "Aplicado. Prueba y pulsa Ctrl+C para quedarte con estas reglas"; sleep 90; nft -f /etc/nftables.conf.bak; echo "REVERTIDO"'
sudo systemctl enable --now nftables
sudo nft list ruleset
```

El segundo comando de aplicar carga las reglas y vuelve a las anteriores a los 90 segundos si no se cancela con Ctrl+C, para no quedarse sin acceso al router si algo falla. Como `/etc/nftables.conf` ya contiene las reglas nuevas, se cargan al arrancar.

Para volver atrás: `sudo cp /etc/nftables.conf.bak /etc/nftables.conf && sudo systemctl restart nftables`.

## 5. Pruebas del firewall

### 5.1 Realizadas el 2026-11-10 (con `W-05` y `B-05` apagadas)

Desde un cliente de la Intranet (`192.168.20.100`):

| Prueba | Resultado | Qué demuestra |
|---|---|---|
| `ping -c 2 google.com` | Responde | Intranet hacia internet permitido (1 paquete en el contador de esa regla) |
| `nslookup www` | Resuelve a `W-05.g05.itb` (`192.168.10.10`) | DNS permitido |
| `ssh bchecker@192.168.20.1` | Funciona | SSH al router permitido desde la Intranet |
| `nc -zv -w 3 192.168.10.10 80` | `timed out` | El firewall lo deja pasar: la regla de los puertos 21, 22 y 80 cuenta 3 paquetes (3 SYN de 60 bytes). No hay respuesta porque `W-05` estaba apagada |
| `nc -zv -w 3 192.168.10.10 3306` | `timed out` | Bloqueado: 3 paquetes en el contador final de `forward` y entradas `R-05 forward drop` con `DPT=3306` en el log |

El contador final de `input` está a 0, así que no se está bloqueando tráfico legítimo hacia el router.

Comandos para consultar el estado:

```bash
sudo nft list ruleset
sudo journalctl -k | grep "R-05 forward drop"
```

Las horas del `journalctl` están en UTC (una hora por detrás de la hora local).

> Capturas pendientes: `sudo nft list ruleset` con los contadores, las líneas del `journalctl` con `DPT=3306`, y los `nc` y el `ssh` en el cliente.

### 5.2 Pruebas con las VM de la DMZ y la BBDD encendidas (2026-11-10)

| Desde | Prueba | Resultado |
|---|---|---|
| Cliente | `curl -I http://W-05` | `HTTP/1.1 200 OK` (Apache 2.4.69, PHP 8.2.35): el nombre resuelve y el puerto 80 pasa |
| Cliente | `ftp 192.168.10.10` con `bchecker`: `ls`, `put prueba1`, `get index.html` | Funciona. Los datos van por los puertos pasivos 21105, 21103 y 21108, dentro del rango abierto |
| `W-05` | `nc -zv -w 3 192.168.20.10 3306` | `succeeded`: la web llega a MySQL de `B-05` a través del router |

| `W-05` | `nc -zv -w 3 192.168.20.10 22` | `timed out`: bloqueado, como se esperaba |
| `W-05` | `ping -c 2 192.168.20.10` | 100% de pérdida: bloqueado, solo se permite el 3306 |
| `W-05` | `ping -c 2 8.8.8.8` | Responde (0% de pérdida): la DMZ sale a internet |
| `B-05` | `ping -c 2 8.8.8.8` | Responde (0% de pérdida): la Intranet sale a internet |

Con esto quedan comprobadas todas las reglas de la tabla 4.1.

> Capturas: las del `curl`, el FTP, el `nc` al 3306 y los bloqueos y pings desde `W-05` y `B-05` ya están hechas y hay que subirlas a `docs/02-router-firewall/img/`.

## 6. SSH

El router acepta conexiones SSH de `bchecker` desde la Intranet (probado el 2026-11-10 desde el cliente) y desde la red de Isard, que se usa para administrarlo desde fuera de clase. Desde la DMZ el firewall no lo permite.

## 7. Usuario bchecker

```bash
id bchecker
su - bchecker
```

El usuario existe y se ha accedido con él al router. Pendiente de confirmar con el profesor si `bchecker` debe tener `sudo`.
