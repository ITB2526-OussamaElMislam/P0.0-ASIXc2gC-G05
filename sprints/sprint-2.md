# Sprint 2: DHCP, DNS, BBDD y web

**Objetivo:** servicios de red, BBDD con datos y web desplegados.
**Duración prevista:** 10 h. Fechas en ProofHub.

## Planificación y estado

| # | Tarea | Responsable | Estado |
|---|---|---|---|
| 1 | Configurar DHCP en `R-05` para la Intranet | Oussama | Hecho |
| 2 | Configurar DNS en `R-05` | Oussama | Hecho |
| 3 | Hacer que el DNS resuelva `R-05` y `R` | Oussama | Hecho |
| 4 | Crear la VM `B-05` | Piero | Por confirmar |
| 5 | `B-05`: instalar Docker | Piero | Por confirmar |
| 6 | `B-05`: contenedor MySQL y usuario `bchecker` | Piero | Por confirmar |
| 7 | Descargar el CSV y revisar sus columnas | Piero | Por confirmar |
| 8 | Diseñar la tabla de la BBDD | Piero | Por confirmar |
| 9 | Cargar el CSV en MySQL | Piero | Por confirmar |
| 10 | Comprobar los datos con un SELECT | Piero | Por confirmar |
| 11 | Crear la VM `W-05` (DMZ) | Raul | En curso (ver `docs/05-web`) |
| 12 | `W-05`: Docker y contenedor web | Raul | En curso (ver `docs/05-web`) |
| 13 | Configurar SSH en las VM, fuera de Docker | Todos | En curso: SSH de `R-05` hecho y probado desde la Intranet |

Trasladadas del Sprint 1: pruebas del firewall (tareas 12 a 14), que necesitan `W-05` y `B-05` encendidas. Las reglas ya están aplicadas.

Las tareas 1 a 3 se adelantaron porque dependen solo del router.

## Sprint review

> Se completa al cerrar el sprint, con lo que se demuestre de verdad.

### Qué se ha entregado
- DHCP (Kea) y DNS (bind9) en `R-05`, con los registros de `W-05`, `F-05`, `B-05` y `www`, documentados en `docs/03-dhcp-dns`.
- Por completar: BBDD, web y SSH.

### Demostración
- Hecho: un cliente recibe IP por DHCP y resuelve `R-05` y `R` por DNS.
- Por completar: un `SELECT` en `B-05` devuelve los equipamientos educativos; desde la Intranet se abre la web de `W-05`.

### Qué no se ha completado y por qué
- Por completar.

## Retrospectiva

- Por completar al cerrar el sprint.
