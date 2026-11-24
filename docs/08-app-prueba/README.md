# Aplicación de prueba

Proyecto P0.0-ASIXc2gC-G05 · módulo 0379 · responsable: Raul (con la BBDD de Piero) · última revisión: 2026-11-24

La aplicación de prueba es una web en PHP que se ejecuta en `W-05` (DMZ) y muestra los datos de la base de datos de `B-05` (Intranet). El detalle de su despliegue está en `docs/05-web`, sección 7 (código y funciones) y sección 8 (script de despliegue).

## Cómo encaja

| Pieza | Dónde | Detalle |
|---|---|---|
| Aplicación | `W-05` (`192.168.10.10`) | `index.php` en `/var/www/html`, contenedor Docker `php:8.2-apache` con la extensión `mysqli` |
| Base de datos | `B-05` (`192.168.20.10:3306`) | Tabla `equipamientos_educacion` (2709 filas), ver `docs/04-bbdd` |
| Camino de red | DMZ → Intranet por el router `R-05` | El firewall deja pasar solo `W-05` hacia `B-05` por el 3306 |
| Acceso del usuario | Cliente de la Intranet | `http://www.g05.itb`, `http://W-05` o `http://192.168.10.10` |

## Qué hace

- Muestra los registros de `equipamientos_educacion`.
- Permite insertar, editar y borrar registros, de uno en uno o en bloque.

## Pruebas

| Prueba | Resultado |
|---|---|
| `W-05` conecta al 3306 de `B-05` (`nc -zv -w 3 192.168.20.10 3306`) | Conecta (2026-11-10) |
| `W-05` hacia `B-05` por el 22 y con ping | Bloqueado por el firewall (2026-11-10) |
| `curl -I http://W-05` desde un cliente de la Intranet | `HTTP/1.1 200 OK` |
| La web muestra los registros de la BBDD desde un cliente | Captura "comprobación web" en `docs/05-web` |
