# Diagrama de arquitectura (v1)

Proyecto P0.0 Desplegament de la infraestructura (módulo 0379, ASIX).

GitHub dibuja el diagrama automáticamente al abrir este archivo.

```mermaid
flowchart TB
    internet["Internet<br/>Red NAT (default de Isard)"]
    router["R-NCC · router y firewall<br/>NAT · DHCP · DNS"]

    subgraph dmz["DMZ · 172.16.10.0/24 · gw .1"]
        web["W-NCC / F-NCC<br/>Docker: web y FTP<br/>Puertos 80 y 21<br/>172.16.10.10"]
    end

    subgraph intranet["Intranet · 172.16.20.0/24 · gw .1 · DHCP .100-.200"]
        db["B-NCC<br/>Docker: MySQL<br/>Puerto 3306<br/>172.16.20.10"]
        linux["PC Linux<br/>Cliente<br/>IP por DHCP"]
        win["PC Windows<br/>Cliente<br/>IP por DHCP"]
    end

    internet --- router
    router --- dmz
    router --- intranet
```

## Notas

- Todo el tráfico entre redes pasa por el router `R-NCC`, que es el firewall.
- El servidor web de la DMZ solo puede consultar la BBDD por el puerto 3306.
- Los hostnames llevan `NCC` como marcador hasta tener el número de equipo.
- Pendiente de confirmar: si `F-NCC` va en una VM aparte (IP `172.16.10.20`).
- Las VM tienen IP fija; los contenedores Docker publican sus puertos hacia la VM.
