# Sprint 2: DHCP, DNS, BBDD y web

**Objetivo:** servicios de red, BBDD con datos y web desplegados.
**Duración prevista:** 10 h. Fechas en ProofHub.

## Planificación y estado

| # | Tarea | Responsable | Estado |
|---|---|---|---|
| 1 | Configurar DHCP en `R-05` para la Intranet | Oussama | Hecho |
| 2 | Configurar DNS en `R-05` | Oussama | Hecho |
| 3 | Hacer que el DNS resuelva `R-05` y `R` | Oussama | Hecho |
| 4 | Crear la VM `B-05` | Piero | Hecho (`docs/04-bbdd`) |
| 5 | `B-05`: instalar Docker | Piero | Hecho |
| 6 | `B-05`: contenedor MySQL y usuario `bchecker` | Piero | Hecho |
| 7 | Descargar el CSV y revisar sus columnas | Piero | Hecho |
| 8 | Diseñar la tabla de la BBDD | Piero | Hecho |
| 9 | Cargar el CSV en MySQL | Piero | Hecho: 2709 filas, 1981 centros |
| 10 | Comprobar los datos con un SELECT | Piero | Hecho |
| 11 | Crear la VM `W-05` (DMZ) | Raul | Hecho (`docs/05-web`) |
| 12 | `W-05`: Docker y contenedor web | Raul | Hecho |
| 13 | Configurar SSH en las VM, fuera de Docker | Todos | Hecho en `R-05` (probado desde la Intranet), `W-05` y `B-05` |

Trasladadas del Sprint 1: pruebas del firewall (tareas 12 a 14). Se hicieron el 2026-11-10 con `W-05` y `B-05` encendidas y salieron bien.

Las tareas 1 a 3 se adelantaron porque dependen solo del router.

## Sprint review

### Qué se ha entregado
- DHCP (Kea) y DNS (bind9) en `R-05`, con los registros de `W-05`, `F-05`, `B-05` y `www`, documentados en `docs/03-dhcp-dns`.
- `B-05` con MySQL 8.0 en Docker, la tabla `equipamientos_educacion` con 2709 filas cargadas y persistencia comprobada tras reiniciar (`docs/04-bbdd`).
- `W-05` con Apache y PHP en Docker, SSH y UFW (`docs/05-web`).
- SSH en las tres VM y usuario `bchecker` con `sudo`.

### Demostración
- Hecho: un cliente recibe IP por DHCP y resuelve `R-05` y `R` por DNS.
- Un `SELECT` en `B-05` devuelve los equipamientos educativos (por distrito, con el Eixample a la cabeza).
- Desde un cliente de la Intranet, `curl -I http://W-05` devuelve `200 OK` y `W-05` conecta al 3306 de `B-05`.

### Qué no se ha completado y por qué
- Nada pendiente del sprint. Solo faltan algunas capturas de la documentación.

## Retrospectiva

- Qué ha ido bien: cada uno ha documentado su servicio en su carpeta, y las pruebas con el firewall se hicieron en una sola sesión con las tres VM encendidas.
- Qué ha fallado: `W-05` y `B-05` no se veían entre sí porque solo tenían ruta de su propia red; se arregló añadiendo la ruta por el router en cada VM. Varias VM estaban apagadas cuando se quería probar.
- Qué mejoramos: coordinarnos para tener las VM encendidas a la vez, y hacer las capturas al terminar cada tarea y no al final.
