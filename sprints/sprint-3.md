# Sprint 3: FTP, clientes, app y cierre

**Objetivo:** infraestructura completa y app de prueba funcionando.
**Duración prevista:** 10 h. Fechas en ProofHub.

## Planificación y estado

| # | Tarea | Responsable | Estado |
|---|---|---|---|
| 1 | `W-05`: contenedor FTP | Raul | Hecho (`docs/06-ftp`) |
| 2 | FTP: usuario `bchecker` y prueba de subida y bajada | Raul | Hecho; probado también desde los clientes Linux y Windows |
| 3 | DNS: `F-05` apunta a `W-05` | Oussama | Hecho; falta probar con `W-05` encendida |
| 4 | Crear el cliente Linux en la Intranet | Oussama | Hecho |
| 5 | Crear el cliente Windows en la Intranet | Oussama | Hecho |
| 6 | Probar DHCP, DNS y `bchecker` en los dos clientes | Todos | DHCP y DNS probados en los dos; falta `bchecker` |
| 7 | Generar la app de prueba con IA | Raul | Hecho (`docs/08-app-prueba`) |
| 8 | Desplegar la app en `W-05` y conectarla a la BBDD | Raul | Hecho |
| 9 | Probar desde un cliente que la app muestra las tablas | Todos | Hecho (captura en `docs/05-web`) |
| 10 | Revisar que `bchecker` funciona en todo | Todos | `sudo` comprobado en las 3 VM; falta probarlo en los clientes |
| 11 | GitHub: clave pública/privada | Oussama | Pendiente |
| 12 | Documentar el control de versiones | Oussama | Hecho (`docs/09-control-versiones`) |
| 13 | Montar el árbol de documentación en Markdown | Todos | Hecho |
| 14 | Actualizar el diagrama final | Oussama | Hecho: diagrama vigente `diagrama.png`; eliminado el antiguo |
| 15 | Revisión final y entrega | Todos | En curso |

## Sprint review

- Qué se ha entregado: FTP en Docker con modo pasivo, clientes Linux y Windows por DHCP, la aplicación web conectada a la BBDD y toda la documentación en el repo.
- Demostración: la app muestra las tablas de la BBDD desde un cliente; subida y bajada de un fichero por FTP con `bchecker`.
- Qué no se ha completado y por qué: falta la clave SSH de GitHub (tarea 11) y probar `bchecker` en los clientes. Quedan también algunas capturas.

## Retrospectiva

- Qué ha ido bien: el router como único punto de paso hizo que el firewall y el DNS sirvieran a todos los servicios; el FTP pasivo funcionó en cuanto se abrió su rango de puertos (21100-21110) en el router.
- Qué ha fallado: el FTP pasivo se quedaba sin listar hasta abrir esos puertos en el firewall, y los nombres de equipo y rangos de IP antiguos aparecen aún en algunas capturas.
- Qué mejoramos: acordar antes los puertos de cada servicio con quien lleva el firewall.
