# Sprints

Proyecto P0.0-ASIXc2gC-G05 · módulo 0379 · grupo 05 (Oussama, Piero y Raul)

Metodología Scrum con 3 sprints quincenales de 10 h, planificados en ProofHub. Cada sprint tiene su propio fichero con la planificación, el estado de las tareas, la sprint review y la retrospectiva.

| Responsable | Parte |
|---|---|
| Oussama | Router `R-05`: red, NAT, firewall, DHCP, DNS |
| Piero | Base de datos `B-05` |
| Raul | DMZ: `W-05` (web) y FTP |

## Resumen

| Sprint | Objetivo | MVP (qué se demuestra) | Estado |
|---|---|---|---|
| [Sprint 1](sprint-1.md) | Red base con router y firewall operativos | Una VM de la Intranet sale a internet a través del router | Casi completo: falta el firewall |
| [Sprint 2](sprint-2.md) | Servicios de red, BBDD con datos y web desplegados | Un cliente recibe IP por DHCP, resuelve nombres, y la web y la BBDD responden | En curso: DHCP y DNS hechos |
| [Sprint 3](sprint-3.md) | Infraestructura completa y app de prueba funcionando | Desde un cliente, la app muestra las tablas de la BBDD | Pendiente |

Las fechas de cada sprint son las de la planificación en ProofHub.

## Cómo se trabaja

- Las tareas son pequeñas (de 15 min a 1 h), con un resultado que se puede comprobar.
- Cada tarea terminada se documenta en la carpeta de `docs/` de su servicio.
- Lo que no se termina en un sprint pasa al siguiente y queda anotado en su sprint review.
