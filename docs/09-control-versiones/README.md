# Control de versiones (Git y GitHub)

Proyecto P0.0-ASIXc2gC-G05 · módulo 0379 · última revisión: 2026-11-10

Toda la documentación y las configuraciones del proyecto viven en un repositorio de GitHub. Cada miembro documenta su parte en la carpeta que le corresponde.

## 1. Repositorio

| Dato | Valor |
|---|---|
| Nombre | `P0.0-ASIXc2gC-G05` |
| Propietario | `ITB2526-OussamaElMislam` |
| Rama | `main` |
| Equipo | Oussama, Piero y Raul |

## 2. Estructura

| Carpeta | Contenido | Responsable |
|---|---|---|
| `docs/01-red` | Redes y direccionamiento | Oussama |
| `docs/02-router-firewall` | Router `R-05`, NAT y firewall | Oussama |
| `docs/03-dhcp-dns` | DHCP (Kea) y DNS (bind9) | Oussama |
| `docs/04-bbdd` | Base de datos `B-05` | Piero |
| `docs/05-web` | Servidor web | Raul |
| `docs/06-ftp` | Servidor FTP | Raul |
| `docs/07-clientes` | Clientes Linux y Windows | Por asignar |
| `docs/08-app-prueba` | Aplicación de prueba | Por asignar |
| `docs/09-control-versiones` | Este documento | Oussama |
| `diagramas/` | Diagrama de arquitectura | Equipo |
| `sprints/` | Planificación, review y retrospectiva de cada sprint | Equipo |
| `configs/` | Ficheros de configuración | Equipo |

Las capturas de cada servicio van en una carpeta `img/` dentro de su carpeta de documentación.

## 3. Configuración inicial de Git

```bash
sudo apt install -y git
git config --global user.name "Oussama El Mislam"
git config --global user.email "<correo del usuario>"
git clone https://github.com/ITB2526-OussamaElMislam/P0.0-ASIXc2gC-G05.git
cd P0.0-ASIXc2gC-G05
```

## 4. Flujo de trabajo

Se trabaja directamente sobre `main`, con commits pequeños y un mensaje que diga qué se ha hecho. Antes de empezar y antes de subir cambios, se trae lo que hayan subido los demás para no pisar su trabajo.

```bash
git pull                           # traer lo último
git status                         # ver qué ha cambiado
git add docs/02-router-firewall    # preparar los cambios (o git add -A)
git commit -m "Describe lo que has hecho"
git push origin main               # subir
git log --oneline -5               # comprobar el historial
```

Si el `push` se rechaza porque alguien ha subido antes, se hace `git pull` y se vuelve a intentar. Los ficheros pueden subirse también desde la web de GitHub (Add file → Upload files), como se ha hecho con algunas capturas.

### Mensajes de commit

Ejemplos reales del proyecto:

- `Documentación de red, router, DHCP y DNS`
- `Añadir planificación, review y retrospectiva de los sprints`
- `Documenta firewall nftables, registros DNS y SSH del router R-05`

## 5. Qué se sube y qué no

- Se sube: documentación en Markdown, capturas, diagramas y ficheros de configuración.
- No se sube: contraseñas, claves privadas ni ficheros con datos sensibles. En la documentación, las contraseñas se sustituyen por `<contraseña>`.

## 6. Acceso por SSH a GitHub

Pendiente (tarea 11 del Sprint 3): crear una clave SSH en la VM, añadir la pública a GitHub y comprobar con `ssh -T git@github.com`.

```bash
ssh-keygen -t ed25519 -C "P0.0-G05"
cat ~/.ssh/id_ed25519.pub
```

> Captura pendiente: clave añadida en GitHub y respuesta de `ssh -T git@github.com`.

## 7. Pendiente

- Decidir qué hacer con `diagramas/arquitectura_p0_0_dmz_intranet.png`, que usa los rangos antiguos `172.16.x.x` y no coincide con el diagrama vigente (`diagrama.png`).
- Completar este documento con la clave SSH cuando se cree.
