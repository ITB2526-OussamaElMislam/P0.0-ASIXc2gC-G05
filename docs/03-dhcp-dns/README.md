# DHCP y DNS

Proyecto P0.0-ASIXc2gC-G05 · módulo 0379 · responsable: Oussama · última revisión: 2026-11-10

Los dos servicios se ejecutan directamente en la VM del router `R-05` (sin Docker). El DHCP lo da Kea y el DNS, bind9.

| Servicio | Paquete | Escucha en | Función |
|---|---|---|---|
| DHCP | `kea-dhcp4-server` | `enp3s0` (Intranet) | Reparte `192.168.20.100-200` |
| DNS | `bind9` (servicio `named`) | `127.0.0.1`, `192.168.20.1`, `192.168.10.1` | Zona `g05.itb` y reenvío a internet |

Dominio interno: `g05.itb`.

## 1. DHCP con Kea

```bash
sudo apt install -y kea-dhcp4-server
```

Fichero `/etc/kea/kea-dhcp4.conf`:

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

Los clientes reciben rango `.100` a `.200`, puerta de enlace y DNS `192.168.20.1`, y el dominio `g05.itb`.

```bash
sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf
sudo systemctl enable --now kea-dhcp4-server
sudo systemctl restart kea-dhcp4-server
systemctl status kea-dhcp4-server
```

Los avisos `WARN` sobre multihilo que muestra la validación son normales.

### Pruebas del DHCP

- En un cliente de la Intranet con IP automática: `ip -br a` muestra una dirección entre `192.168.20.100` y `.200`, y `ip route` una ruta por defecto por `192.168.20.1`.
- En el router, los préstamos quedan en `/var/lib/kea/kea-leases4.csv`.
- Para ver las peticiones: `sudo tcpdump -ni enp3s0 port 67 or port 68`.

> Captura pendiente: cliente con su IP y `kea-leases4.csv`.

## 2. DNS con bind9

```bash
sudo apt install -y bind9 bind9-utils bind9-dnsutils
```

### Opciones: `/etc/bind/named.conf.options`

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

Escucha solo en las IPs del router, para no chocar con `systemd-resolved` (`127.0.0.53`). Responde a las dos redes internas y reenvía el resto de consultas a `8.8.8.8` y `1.1.1.1`.

### Zona: `/etc/bind/named.conf.local`

```
zone "g05.itb" {
    type master;
    file "/etc/bind/db.g05.itb";
};
```

### Fichero de zona: `/etc/bind/db.g05.itb`

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

Cumple el enunciado: `R-05` y `R` resuelven a la IP del router. Además se han añadido `W-05`, `B-05`, `F-05` (misma IP que `W-05`, porque web y FTP comparten VM) y el alias `www` hacia `W-05`, para que los equipos resuelvan el nombre de la web. Al añadirlos se subió el `serial` a 2; hay que volver a subirlo con cada cambio.

```bash
sudo named-checkconf
sudo named-checkzone g05.itb /etc/bind/db.g05.itb
sudo systemctl restart named
sudo systemctl enable named
systemctl status named
```

### Pruebas del DNS

En el router:

```bash
dig @192.168.20.1 R-05.g05.itb +short      # 192.168.20.1
dig @192.168.20.1 R.g05.itb +short         # 192.168.20.1
dig @192.168.20.1 google.com +short        # IP de Google
```

Desde un cliente de la Intranet (2026-10-28):

| Prueba | Resultado |
|---|---|
| `ping R-05` | Responde `192.168.20.1` (resuelve `R-05.g05.itb`) |
| `ping R` | Responde `192.168.20.1` (resuelve `R.g05.itb`) |
| `ping google.com` | Responde |
| `nslookup R-05` | `R-05.g05.itb` → `192.168.20.1` |
| `nslookup www` | Alias de `W-05.g05.itb` → `192.168.10.10` (2026-11-10) |

> Captura pendiente: `ping R-05` y `nslookup R-05` en el cliente.

## 3. Pendiente

- Probar `ping W-05` desde el cliente con `W-05` encendida (el nombre ya resuelve, la VM estaba apagada).
- Comprobar que `W-05` y `B-05` usan el DNS del router (`192.168.10.1` y `192.168.20.1` respectivamente) y resuelven `B-05.g05.itb`.
- Opcional: zona inversa.

## 4. Problemas habituales

- `named-checkzone` da error de sintaxis: revisar los puntos finales en `ns.g05.itb.` y `admin.g05.itb.`, y los `;` de los comentarios.
- El cliente no resuelve: comprobar con `resolvectl status` que su DNS es `192.168.20.1` y que tiene el dominio `g05.itb`.
- `kea-dhcp4 -t` da error: el JSON no admite comas sobrantes ni comentarios.
- El servicio no arranca: `sudo journalctl -u named -n 30` o `sudo journalctl -u kea-dhcp4-server -n 30`.
