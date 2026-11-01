# Router y firewall (R-05)

Proyecto P0.0-ASIXc2gC-G05 · módulo 0379 · responsable: Oussama · última revisión: 2026-10-28

El router `R-05` es una VM de Isard con Ubuntu Server 24.04 y tres interfaces. Es el único punto de paso entre NAT, DMZ e Intranet.

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

## 4. NAT (masquerade)

Fichero `/etc/nftables.conf`:

```
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
	chain input {
		type filter hook input priority filter;
	}
	chain forward {
		type filter hook forward priority filter;
	}
	chain output {
		type filter hook output priority filter;
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
sudo nft -f /etc/nftables.conf
sudo systemctl enable --now nftables
sudo nft list ruleset
```

Todo lo que sale por `enp1s0` (red NAT de Isard) se enmascara con la IP del router.

> Captura pendiente: salida de `sudo nft list ruleset`.

### Pruebas del NAT

- Desde el router: `ping -c 3 8.8.8.8` y `ping -c 3 google.com` responden.
- Desde un cliente de la Intranet: `ping -c 3 google.com` responde, por lo que los paquetes de la Intranet salen a internet a través del router.

## 5. Firewall

Estado: pendiente. De momento las cadenas `input`, `forward` y `output` de la tabla `inet filter` están vacías y aceptan todo el tráfico.

Reglas previstas (política por defecto: denegar):

| Origen | Destino | Puerto | Acción |
|---|---|---|---|
| Intranet | NAT (internet) | Cualquiera | Permitir |
| DMZ | NAT (internet) | Cualquiera | Permitir |
| Intranet | DMZ | 80, 21, 22 | Permitir |
| `W-05` (DMZ) | `B-05` (Intranet) | 3306 | Permitir |
| Resto de la DMZ | Intranet | Cualquiera | Denegar |
| NAT | DMZ e Intranet | Cualquiera | Denegar |

Cuando se apliquen, hay que dejar pasar de forma explícita en `input` el DHCP (puertos 67 y 68), el DNS (53) y el SSH (22) hacia el router, para no cortarse el acceso.

### Pruebas del firewall

Pendiente de completar cuando estén las VM de la DMZ y la BBDD:

- Permitido: Intranet hacia web, FTP y SSH de la DMZ; `W-05` hacia `B-05` por el 3306.
- Bloqueado: DMZ hacia la Intranet por cualquier otro puerto.

## 6. Comprobación del usuario bchecker

```bash
id bchecker
su - bchecker
```

> Pendiente: confirmar con el profesor si `bchecker` debe tener `sudo`.
