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
| `docs/07-clientes` | Clientes Linux y Windows | Oussama |
| `docs/08-app-prueba` | Aplicación de prueba | Raul |
| `docs/09-control-versiones` | Este documento | Oussama |
| `diagramas/` | Diagrama de arquitectura | Equipo |
| `sprints/` | Planificación, review y retrospectiva de cada sprint | Equipo |

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

## 7. Plan B: gestionar el repositorio sin Claude

Si Claude no está disponible, cualquiera del equipo puede subir todo con Git o desde la web de GitHub.

### 7.1 Con Git (Linux o PowerShell)

Instalar Git si no está: `sudo apt install -y git` en Ubuntu, o Git for Windows en Windows. Después, una sola vez:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu_correo"
git clone https://github.com/ITB2526-OussamaElMislam/P0.0-ASIXc2gC-G05.git
cd P0.0-ASIXc2gC-G05
```

Copiar los ficheros nuevos a su carpeta (por ejemplo, las capturas a `docs/02-router-firewall/img/`) y subirlos:

```bash
git pull                                  # traer lo último antes de tocar nada
git status                                # ver qué hay de nuevo
git add -A                                # preparar todo (o git add docs/02-router-firewall)
git commit -m "Describe lo que has hecho"
git push origin main
```

Al hacer `push`, GitHub pide usuario y contraseña. La contraseña es un **token de acceso personal** (GitHub → Settings → Developer settings → Personal access tokens), no la contraseña de la cuenta. Con una clave SSH configurada (apartado 6) no pide nada.

Para subir el repositorio entero desde cero a un repositorio vacío:

```bash
cd carpeta_del_proyecto
git init -b main
git add -A
git commit -m "Subida completa del proyecto P0.0 G05"
git remote add origin https://github.com/ITB2526-OussamaElMislam/P0.0-ASIXc2gC-G05.git
git push -u origin main
```

### 7.2 Desde la web de GitHub (sin instalar nada)

1. Abrir el repositorio y entrar en la carpeta donde va el fichero.
2. Pulsar **Add file → Upload files**.
3. Arrastrar los ficheros o las carpetas. Máximo 100 ficheros por subida y 25 MB cada uno.
4. Escribir un mensaje y pulsar **Commit changes**.

Para editar un fichero de texto: abrirlo, pulsar el lápiz, cambiarlo y confirmar el commit.

### 7.3 Problemas habituales

| Problema | Solución |
|---|---|
| `push` rechazado (`rejected`, `fetch first`) | Alguien ha subido antes: `git pull --rebase` y de nuevo `git push origin main` |
| `Please tell me who you are` | Falta `git config --global user.name` y `user.email` |
| Pide contraseña y la rechaza | Usar un token de acceso personal, no la contraseña de la cuenta |
| `This repository moved` | Es un aviso. Para quitarlo: `git remote set-url origin https://github.com/ITB2526-OussamaElMislam/P0.0-ASIXc2gC-G05.git` |
| Conflicto al hacer `pull` | Abrir el fichero, quedarse con el texto correcto, quitar las marcas `<<<<<<<`, `=======` y `>>>>>>>`, y luego `git add` y `git commit` |
| Un fichero no se sube | Comprobar que no pasa de 25 MB (web) o 100 MB (Git) y que no está en `.gitignore` |

### 7.4 Copia de seguridad antes de presentar

En la página del repositorio: **Code → Download ZIP**. Así se tiene una copia completa por si falla internet o GitHub durante la presentación. También sirve tener el repositorio clonado en una VM o en el portátil: `git clone` y `git pull` antes de empezar.

### 7.5 Comprobación final

- `git status` dice `nothing to commit, working tree clean` y `git log --oneline -5` muestra los últimos commits.
- En la web del repositorio se ven todas las carpetas (`docs`, `diagramas`, `sprints`) y los `README.md` se muestran con formato.
- Las capturas se ven en cada `README.md` (si aparece un icono roto, la ruta del `img/` es incorrecta).

## 8. Pendiente

- Decidir qué hacer con `diagramas/arquitectura_p0_0_dmz_intranet.png`, que usa los rangos antiguos `172.16.x.x` y no coincide con el diagrama vigente (`diagrama.png`).
- Completar este documento con la clave SSH cuando se cree.
