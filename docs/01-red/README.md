# Red y direccionamiento

Proyecto P0.0-ASIXc2gC-G05 · módulo 0379 · última revisión: 2026-10-28

## Redes

| Red del proyecto | Red de Isard | Rango | Puerta de enlace |
|---|---|---|---|
| NAT (internet) | default | La asigna Isard | La de Isard |
| DMZ | `ASIXc2-ITB5` | `192.168.10.0/24` | `192.168.10.1` (`R-05`) |
| Intranet | `ASIXc2-ITB5a` | `192.168.20.0/24` | `192.168.20.1` (`R-05`) |

El rango de la red default de Isard (por ejemplo `192.168.122.106/22`) no se solapa con la DMZ ni con la Intranet. Las redes DMZ e Intranet no tienen DHCP propio de Isard.

## Equipos y direcciones

| Equipo | Hostname | Red | IP | Asignación |
|---|---|---|---|---|
| Router | `R-05` | NAT (`enp1s0`) | La da Isard | DHCP de Isard |
| Router | `R-05` | DMZ (`enp2s0`) | `192.168.10.1` | Fija |
| Router | `R-05` | Intranet (`enp3s0`) | `192.168.20.1` | Fija |
| Servidor web y FTP | `W-05` / `F-05` | DMZ | `192.168.10.10` | Fija |
| Base de datos | `B-05` | Intranet | `192.168.20.10` | Fija |
| Cliente Linux | Por decidir | Intranet | `192.168.20.100-200` | DHCP |
| Cliente Windows | Por decidir | Intranet | `192.168.20.100-200` | DHCP |

`F-05` comparte la VM de `W-05` y se resuelve por DNS a la misma IP. Si se separa en otra VM, tomaría `192.168.10.20`.

## Dominio interno

`g05.itb`. Los clientes reciben este dominio por DHCP como dominio de búsqueda, por lo que pueden usar nombres cortos (`R-05`, `R`).

## Reparto del trabajo

| Equipo | Responsable |
|---|---|
| Router `R-05` | Oussama |
| Base de datos `B-05` | Piero |
| DMZ `W-05` / `F-05` | Raul |

## Relación con otros documentos

- Configuración del router y firewall: `docs/02-router-firewall/README.md`
- DHCP y DNS: `docs/03-dhcp-dns/README.md`
- Diagrama de arquitectura: `diagramas/`
