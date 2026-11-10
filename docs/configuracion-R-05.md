# Configuración del router R-05

Proyecto P0.0-ASIXc2gC-G05 · módulo 0379 · responsable del router: Oussama · última revisión: 2026-11-10

Documento de referencia con toda la configuración aplicada al router `R-05`. El detalle de cada servicio, con sus pruebas, está en `docs/01-red`, `docs/02-router-firewall` y `docs/03-dhcp-dns`.

## 1. Resumen

| Elemento | Valor |
|---|---|
| Sistema | Ubuntu Server 24.04 (VM de Isard) |
| Hostname | `R-05` |
| Funciones | Router, NAT, firewall (nftables), DHCP (Kea), DNS (bind9) |
| Dominio interno | `g05.itb` |
| Usuario de comprobación | `bchecker` |

| Interfaz | Red de Isard | Red del proyecto | IP |
|---|---|---|---|
| `enp1s0` | default | NAT (internet) | Por DHCP de Isard (ejemplo: `192.168.122.106/22`) |
| `enp2s0` | `ASIXc2-ITB5` | DMZ | `192.168.10.1/24` |
| `enp3s0` | `ASIXc2-ITB5a` | Intranet | `192.168.20.1/24` |

Los nombres `enp1s0` a `enp3s0` siguen el orden en que se añadieron las redes a la VM en Isard: default, DMZ e Intranet.

## 2. Sistema

### 2.1 Hostname

```bash
sudo hostnamectl set-hostname R-05
sudo nano /etc/hosts      # debe existir la línea: 127.0.1.1 R-05
echo 'preserve_hostname: true' | sudo tee /etc/cloud/cloud.cfg.d/99-hostname.cfg
hostnamectl
```

La última línea evita que cloud-init reescriba el nombre al reiniciar.

### 2.2 Usuario bchecker

```bash
sudo useradd -m -s /bin/bash bchecker
echo 'bchecker:<contraseña del enunciado>' | sudo chpasswd
id bchecker
```

La contraseña es la que indica el enunciado de la práctica P0.0. El profesor confirmó (2026-11-10) que `bchecker` debe tener `sudo` en todas las máquinas, así que se añade al grupo `sudo`:

```bash
sudo usermod -aG sudo bchecker
id bchecker
```

## 3. Red (netplan)

Fichero: `/etc/netplan/` (el `.yaml` que genera el instalador).

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

Solo `enp1s0` lleva ruta por defecto, la que da Isard. Las redes DMZ e Intranet no tienen DHCP propio de Isard, por eso el DHCP de la Intranet lo da el router.

## 4. Reenvío de paquetes

Fichero: `/etc/sysctl.d/99-forward.conf`

```
net.ipv4.ip_forward=1
```

```bash
sudo sysctl --system
cat /proc/sys/net/ipv4/ip_forward      # debe devolver 1
```

## 5. nftables: NAT y firewall

Fichero: `/etc/nftables.conf`. Política por defecto denegar en `input` y `forward`, con NAT por `enp1s0`. El detalle de reglas y pruebas está en `docs/02-router-firewall`.

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

```bash
sudo nft -c -f /etc/nftables.conf
sudo nft -f /etc/nftables.conf
sudo systemctl enable --now nftables
sudo nft list ruleset
```

Estado: firewall aplicado el 2026-11-10 y probado desde el cliente de la Intranet. Probado con las VM encendidas: web, FTP pasivo, `W-05`→`B-05` por el 3306, bloqueos del 22 y ping desde `W-05` y salida a internet de `W-05` y `B-05` (2026-11-10).

## 6. DHCP (Kea)

Paquete: `kea-dhcp4-server`. Fichero: `/etc/kea/kea-dhcp4.conf`

```json
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": [ "enp3s0" ]
    },
    "lease-database": {
      "type": "memfile",
      "lfc-interval": 3600
    },
    "valid-lifetime": 3600,
    "subnet4": [
      {
        "id": 1,
        "subnet": "192.168.20.0/24",
        "pools": [
          { "pool": "192.168.20.100 - 192.168.20.200" }
        ],
        "option-data": [
          { "name": "domain-name", "data": "g05.itb" },
          { "name": "domain-search", "data": "g05.itb" },
          { "name": "routers", "data": "192.168.20.1" },
          { "name": "domain-name-servers", "data": "192.168.20.1" }
        ]
      }
    ]
  }
}
```

```bash
sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf
sudo systemctl enable --now kea-dhcp4-server
sudo systemctl restart kea-dhcp4-server
systemctl status kea-dhcp4-server
sudo cat /var/lib/kea/kea-leases4.csv
```

## 7. DNS (bind9)

Paquetes: `bind9`, `bind9-utils`, `bind9-dnsutils`. El servicio se llama `named`.

### 7.1 Opciones: `/etc/bind/named.conf.options`

```
options {
    directory "/var/cache/bind";
    listen-on { 127.0.0.1; 192.168.20.1; 192.168.10.1; };
    listen-on-v6 { none; };
    allow-query { 127.0.0.1; 192.168.20.0/24; 192.168.10.0/24; };
    recursion yes;
    allow-recursion { 127.0.0.1; 192.168.20.0/24; 192.168.10.0/24; };
    forwarders { 8.8.8.8; 1.1.1.1; };
    dnssec-validation auto;
};
```

### 7.2 Zona: `/etc/bind/named.conf.local`

```
zone "g05.itb" {
    type master;
    file "/etc/bind/db.g05.itb";
};
```

### 7.3 Fichero de zona: `/etc/bind/db.g05.itb`

```
$TTL 604800
@       IN  SOA ns.g05.itb. admin.g05.itb. (
                2       ; serial
                604800  ; refresh
                86400   ; retry
                2419200 ; expire
                604800 ) ; negative cache
@       IN  NS  ns.g05.itb.
ns      IN  A   192.168.20.1
R-05    IN  A   192.168.20.1
R       IN  A   192.168.20.1
W-05    IN  A   192.168.10.10
F-05    IN  A   192.168.10.10
B-05    IN  A   192.168.20.10
www     IN  CNAME W-05
```

Al modificar la zona hay que subir el número de `serial`.

```bash
sudo named-checkconf
sudo named-checkzone g05.itb /etc/bind/db.g05.itb
sudo systemctl restart named
sudo systemctl enable named
systemctl status named
```

## 8. Verificación

En el router:

```bash
ping -c 3 8.8.8.8
ping -c 3 google.com
dig @192.168.20.1 R-05.g05.itb +short      # 192.168.20.1
dig @192.168.20.1 R.g05.itb +short         # 192.168.20.1
```

Desde un cliente de la Intranet (resultado obtenido el 2026-10-28):

| Prueba | Resultado |
|---|---|
| IP por DHCP | Recibida dentro de `192.168.20.100-200` |
| `ping R-05` | Responde `192.168.20.1` (resuelve `R-05.g05.itb`) |
| `ping R` | Responde `192.168.20.1` (resuelve `R.g05.itb`) |
| `ping google.com` | Responde: el DNS reenvía y el NAT funciona desde la Intranet |
| `nslookup R-05` | `R-05.g05.itb` → `192.168.20.1` |

Firewall, desde el cliente de la Intranet (2026-11-10, con `W-05` y `B-05` apagadas):

| Prueba | Resultado |
|---|---|
| `ping google.com` | Responde (permitido) |
| `nslookup www` | `W-05.g05.itb` → `192.168.10.10` |
| `ssh bchecker@192.168.20.1` | Funciona |
| `nc` al 80 de `W-05` | Permitido por el firewall (3 paquetes en su regla); sin respuesta porque la VM estaba apagada |
| `nc` al 3306 de `W-05` | Bloqueado y registrado en el log |

## 9. Estado y pendiente

| Elemento | Estado |
|---|---|
| Hostname `R-05` | Hecho |
| IPs fijas y reenvío | Hecho |
| NAT (masquerade) | Hecho y probado desde la Intranet |
| DHCP (Kea) | Hecho y probado |
| DNS (bind9): `R-05` y `R` | Hecho y probado |
| Usuario `bchecker` | Hecho |
| SSH | Hecho y probado desde la Intranet |
| Firewall (nftables) | Hecho y probado (2026-11-10) |
| DNS: registros de `W-05`, `F-05`, `B-05` y `www` | Hecho; falta probar con `W-05` encendida |
