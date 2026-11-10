# Sprint 1: base y red

**Objetivo:** red base con router y firewall operativos.
**Duración prevista:** 10 h. Fechas en ProofHub.
**Responsable principal:** Oussama (router).

## Planificación y estado

| # | Tarea | Estado |
|---|---|---|
| 1 | Crear proyecto en ProofHub y planificar sprints | Por confirmar |
| 2 | Crear repositorio en GitHub (README y estructura de carpetas) | Hecho |
| 3 | Definir rangos IP y hostnames | Hecho |
| 4 | Dibujar el diagrama de arquitectura | Hecho |
| 5 | Documento de justificación de la arquitectura | Hecho |
| 6 | Crear las redes DMZ e Intranet en Isard | Hecho |
| 7 | Crear la VM del router `R-05` con 3 interfaces | Hecho |
| 8 | Configurar IPs fijas y reenvío de paquetes | Hecho |
| 9 | Configurar NAT y probar la salida a internet | Hecho, probado desde la Intranet |
| 10 | Crear el usuario `bchecker` en el router | Hecho |
| 11 | Firewall: política por defecto y reglas entre redes | Hecho |
| 12 | Firewall: web hacia BBDD por el 3306 | Regla aplicada; falta probarla con `W-05` y `B-05` encendidas |
| 13 | Firewall: clientes hacia la DMZ | Regla aplicada; falta probarla con `W-05` encendida |
| 14 | Firewall: probar las reglas | En curso: comprobados el paso y el bloqueo desde el cliente |
| 15 | Documentar red y router | Hecho, faltan las capturas |

Resultado (2026-11-10): 11 tareas hechas, 2 con la regla aplicada pendientes de probar con las VM de los compañeros, 1 en curso y 1 por confirmar (ProofHub).

## Sprint review

### Qué se ha entregado

- Repositorio con la estructura de documentación y los diagramas.
- Tabla de direccionamiento y documento de justificación de la arquitectura.
- Redes de Isard: `default` (internet), `ASIXc2-ITB5` (DMZ) y `ASIXc2-ITB5a` (Intranet).
- Router `R-05` con Ubuntu Server, tres interfaces (`enp1s0`, `enp2s0`, `enp3s0`), IPs fijas, reenvío de paquetes, NAT y firewall con nftables (política por defecto denegar, aplicado el 2026-11-10).
- Documentación del router en `docs/01-red`, `docs/02-router-firewall` y `docs/configuracion-R-05.md`.

### Demostración

Desde un cliente de la Intranet:

| Prueba | Resultado |
|---|---|
| IP automática | Recibida dentro de `192.168.20.100-200` |
| `ping R-05` | Responde `192.168.20.1` |
| `ping google.com` | Responde: los paquetes salen a internet por el NAT del router |

### Qué no se ha completado y por qué

- **Pruebas del firewall (tareas 12 a 14):** las reglas están aplicadas y probadas desde el cliente (paso y bloqueo comprobados con los contadores y el log). Falta probar `W-05` hacia `B-05` por el 3306 y el acceso a la web, y eso requiere las VM de los compañeros encendidas. Pasa al Sprint 2.
- **ProofHub:** hay que confirmar y cerrar esta tarea.

### Decisiones tomadas durante el sprint

| Decisión | Motivo |
|---|---|
| Rangos `192.168.10.0/24` y `192.168.20.0/24` en lugar de `172.16.x.x` | Son los del diagrama que acordó el equipo |
| Web y FTP comparten VM (`W-05`), con `F-05` resuelto por DNS | Ahorra una VM; pendiente de confirmar con el profesor |
| Router con Ubuntu Server y netplan | Es lo que usa el equipo y se ajusta mejor a un router |
| Dominio interno `g05.itb` | Evita conflictos con `.local` (mDNS) |
| DHCP con Kea y DNS con bind9 | Servicios separados, uno por función |

## Retrospectiva

**Lo que ha ido bien**
- Las pruebas desde un cliente real detectaron enseguida que el DNS y el NAT funcionaban.
- Documentar cada paso mientras se hacía evitó tener que reconstruirlo después.

**Lo que se puede mejorar**
- El router es el cuello de botella: sin él, Piero y Raul no pueden probar conectividad.
- Instalar un servicio en la VM equivocada (el DNS se instaló primero en el cliente) cuesta tiempo. Conviene mirar el prompt antes de pegar comandos.

**Acciones para el Sprint 2**
- Probar el firewall en cuanto `W-05` y `B-05` estén en su red.
- Capturar pantalla de cada prueba mientras se hace, para no repetirla.
- Confirmar con el profesor si web y FTP pueden compartir VM y si `bchecker` necesita `sudo`.
