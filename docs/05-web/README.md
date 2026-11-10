# Documentación del Servicio Web (DMZ — W-05)

En esta guía se documenta detalladamente el procedimiento técnico para el despliegue y puesta en marcha del servidor web en la máquina virtual **W-05**, ubicada en la zona desmilitarizada (DMZ) del **Proyecto G05**. El proceso abarca la personalización del sistema, el direccionamiento de red estático, la creación de la aplicación web interactiva en PHP para la gestión de equipamientos en la base de datos MySQL, el despliegue contenerizado mediante Docker, la activación del servicio de administración remota OpenSSH y la seguridad perimetral local mediante cortafuegos.

---

## 1. Identificación del Sistema (Hostname)

Para asegurar una correcta identificación y gestión dentro de la topología de red del proyecto, se establece el nombre de host oficial de la máquina como `W-05`.

![cambio de nombre .png](./img/cambio%20de%20nombre%20.png)

---

## 2. Creación del Usuario de Administración `bchecker`

Se crea el usuario del sistema `bchecker` asignándole su directorio personal `/home/bchecker`, la consola interactiva Bash como shell por defecto y la contraseña estándar definida para la infraestructura (`bchecker121`). Además, se le otorgan privilegios administrativos incorporándolo al grupo `sudo`.

![creación del usuario bchecker.png](./img/creaci%C3%B3n%20del%20usuario%20bchecker.png)

---

## 3. Configuración de Red Estática (Netplan)

Para garantizar la conectividad dentro de la DMZ (`itb5`), se edita el archivo de red `/etc/netplan/01-netcfg.yaml` asignando a la interfaz `enp1s0` la dirección IP estática `192.168.10.10/24`. Se establece como puerta de enlace predeterminada y DNS la dirección del router central `192.168.10.1` (R-05).

![configuración de red.png](./img/configuraci%C3%B3n%20de%20red.png)

---

## 4. Instalación del Motor Docker y Gestión de Permisos

Se instala el servicio Docker (`docker.io`) en el sistema operativo base. Con el objetivo de permitir que el usuario `bchecker` administre y despliegue contenedores de manera ágil sin requerir la elevación constante de privilegios mediante `sudo`, se le añade al grupo local `docker`.

![instalación de docker.png](./img/instalaci%C3%B3n%20de%20docker.png)

![añadir usuario bchecker al grupo docker.png](./img/a%C3%B1adir%20usuario%20bchecker%20al%20grupo%20docker.png)

---

## 5. Preparación del Almacenamiento Local y Creación del Sitio Web Base

Para lograr la persistencia de los datos del servidor web fuera del ciclo de vida del contenedor, se crea la estructura de directorios `/var/www/html` en el sistema anfitrión. A continuación, se inicializa el archivo `index.html` con la plantilla HTML personalizada para la identificación del proyecto G05.

![creacion del archivo html.png](./img/creacion%20del%20archivo%20html.png)

Se edita el archivo con la herramienta `nano` para incorporar el código que servirá la portada del servidor.

![configuración archivo html.png](./img/configuraci%C3%B3n%20archivo%20html.png)

---

## 6. Instalación de PHP y Conectores de Base de Datos

Para posibilitar el procesamiento de scripts dinámicos en el servidor web y permitir la conexión con el servidor de base de datos MySQL/MariaDB remoto del proyecto (`192.168.20.10`), se instalan en el sistema los conectores FastCGI y la extensión de conexión `php-mysql` / `mysqli`.

![instalar php.png](./img/instalar%20php.png)

---

## 7. Creación de la Aplicación Web en PHP (`index.php`)

Se genera el script dinámico PHP en `/var/www/html/index.php`. Este desarrollo implementa una aplicación **CRUD completa** para la visualización, inserción, edición y borrado (individual y masivo) de registros de la tabla `equipamientos_educacion` almacenada en la base de datos remota.

![crear archivo php.png](./img/crear%20archivo%20php.png)

---

## 8. Script de Despliegue Automatizado (`deploy_web.sh`)

El aprovisionamiento del directorio web, la generación automática de los archivos `index.html` e `index.php`, el despliegue del contenedor Docker (`php:8.2-apache`) y la instalación dinámica de la extensión `mysqli` se gestionan mediante el siguiente script en Bash:

```bash
#!/bin/bash

# ==============================================================================
# Script de Despliegue Automático — Servidor Web W-05 (Proyecto G05)
# ==============================================================================

set -e

WEB_DIR="/var/www/html"
CONTAINER_NAME="web-container"

echo "=== 1. Creando directorio web en $WEB_DIR ==="
sudo mkdir -p "$WEB_DIR"
sudo chown -R bchecker:bchecker "$WEB_DIR"

echo "=== 2. Creando archivo index.html ==="
cat << 'EOF' > "$WEB_DIR/index.html"
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Servidor Web W-05 | Proyecto G05</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f4f6f9; color: #333; text-align: center; padding-top: 50px; }
        .card { background: white; max-width: 500px; margin: 0 auto; padding: 30px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        h1 { color: #2b6cb0; }
        .btn { display: inline-block; margin-top: 15px; padding: 10px 20px; background-color: #2b6cb0; color: white; text-decoration: none; border-radius: 4px; }
        .btn:hover { background-color: #1a365d; }
    </style>
</head>
<body>
    <div class="card">
        <h1>Servidor Web W-05</h1>
        <p>Entorno de DMZ — <strong>Proyecto G05</strong></p>
        <p>Servidor Apache/PHP funcionando en Docker.</p>
        <a href="index.php" class="btn">Gestión de Equipamientos</a>
    </div>
</body>
</html>
EOF

echo "=== 3. Creando archivo index.php ==="
cat << 'EOF' > "$WEB_DIR/index.php"
<?php
$host     = '192.168.20.10';
$port     = 3306;
$user     = 'bchecker';
$password = 'bchecker121';
$database = 'equipamientos';

$conn = new mysqli($host, $user, $password, $database, $port);

if ($conn->connect_error) {
    die("<div style='color:red; font-weight:bold; padding:15px; border:1px solid red; background-color:#ffe6e6; margin:20px;'>" .
        "Error de conexión con la base de datos. Por favor, inténtelo más tarde.</div>");
}

$conn->set_charset("utf8mb4");
$mensaje = "";

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['action']) && $_POST['action'] === 'delete_selected') {
    if (!empty($_POST['selected_ids']) && is_array($_POST['selected_ids'])) {
        $ids = array_map('intval', $_POST['selected_ids']);
        $in_clause = implode(',', $ids);
        if ($conn->query("DELETE FROM equipamientos_educacion WHERE id IN ($in_clause)")) {
            $count = count($ids);
            $mensaje = "<div class='alert success'>Se han eliminado $count equipamientos correctamente.</div>";
        } else {
            $mensaje = "<div class='alert error'>Error al eliminar la selección realizada.</div>";
        }
    } else {
        $mensaje = "<div class='alert warning'>No has seleccionado ningún elemento para eliminar.</div>";
    }
}

if (isset($_GET['action']) && $_GET['action'] === 'delete' && !empty($_GET['id'])) {
    $id_delete = intval($_GET['id']);
    $stmt = $conn->prepare("DELETE FROM equipamientos_educacion WHERE id = ?");
    $stmt->bind_param("i", $id_delete);
    if ($stmt->execute()) {
        $mensaje = "<div class='alert success'>Equipamiento eliminado correctamente.</div>";
    } else {
        $mensaje = "<div class='alert error'>Error al eliminar el equipamiento.</div>";
    }
    $stmt->close();
}

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['action']) && $_POST['action'] === 'save') {
    $name = $_POST['name'] ?? '';
    $register_id = $_POST['register_id'] ?? '';
    $road_name = $_POST['addresses_road_name'] ?? '';
    $district = $_POST['addresses_district_name'] ?? '';
    $zip_code = $_POST['addresses_zip_code'] ?? '';

    if (!empty($_POST['id'])) {
        $id_edit = intval($_POST['id']);
        $stmt = $conn->prepare("UPDATE equipamientos_educacion SET register_id=?, name=?, addresses_road_name=?, addresses_district_name=?, addresses_zip_code=? WHERE id=?");
        $stmt->bind_param("sssssi", $register_id, $name, $road_name, $district, $zip_code, $id_edit);
        if ($stmt->execute()) {
            $mensaje = "<div class='alert success'>Información actualizada correctamente.</div>";
        } else {
            $mensaje = "<div class='alert error'>Error al actualizar la información.</div>";
        }
        $stmt->close();
    } else {
        $stmt = $conn->prepare("INSERT INTO equipamientos_educacion (register_id, name, addresses_road_name, addresses_district_name, addresses_zip_code) VALUES (?, ?, ?, ?, ?)");
        $stmt->bind_param("sssss", $register_id, $name, $road_name, $district, $zip_code);
        if ($stmt->execute()) {
            $mensaje = "<div class='alert success'>Nuevo equipamiento registrado correctamente.</div>";
        } else {
            $mensaje = "<div class='alert error'>Error al registrar el nuevo equipamiento.</div>";
        }
        $stmt->close();
    }
}

$edit_data = null;
if (isset($_GET['action']) && $_GET['action'] === 'edit' && !empty($_GET['id'])) {
    $id_edit = intval($_GET['id']);
    $stmt = $conn->prepare("SELECT * FROM equipamientos_educacion WHERE id = ?");
    $stmt->bind_param("i", $id_edit);
    $stmt->execute();
    $res = $stmt->get_result();
    if ($res && $res->num_rows > 0) {
        $edit_data = $res->fetch_assoc();
    }
    $stmt->close();
}

$sql = "SELECT id, register_id, name, addresses_road_name, addresses_district_name, addresses_zip_code 
        FROM equipamientos_educacion 
        ORDER BY id DESC LIMIT 50";
$result = $conn->query($sql);
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestión de Equipamientos Educativos - Barcelona</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 30px; background-color: #f4f6f9; color: #333; }
        h1 { color: #1a365d; margin-bottom: 20px; font-weight: 600; }
        .alert { padding: 12px; margin-bottom: 20px; border-radius: 6px; font-weight: 500; }
        .success { background-color: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
        .warning { background-color: #fff3cd; color: #856404; border: 1px solid #ffeeba; }
        .error { background-color: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }
        .toolbar { display: flex; gap: 15px; margin-bottom: 25px; background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
        .icon-btn { display: inline-flex; align-items: center; gap: 8px; padding: 10px 18px; border: none; border-radius: 6px; font-weight: 600; cursor: pointer; text-decoration: none; color: white; font-size: 0.95em; transition: background 0.2s; }
        .btn-add { background-color: #28a745; }
        .btn-add:hover { background-color: #218838; }
        .btn-delete-bulk { background-color: #dc3545; }
        .btn-delete-bulk:hover { background-color: #c82333; }
        .form-card { background: white; padding: 25px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); margin-bottom: 30px; }
        .form-card h2 { color: #2b6cb0; margin-top: 0; font-size: 1.3em; }
        .form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; }
        .form-group label { display: block; margin-bottom: 6px; font-weight: 600; font-size: 0.9em; }
        .form-group input { width: 100%; padding: 9px; box-sizing: border-box; border: 1px solid #ccc; border-radius: 6px; }
        .form-actions { margin-top: 15px; display: flex; gap: 10px; }
        table { border-collapse: collapse; width: 100%; background: white; border-radius: 8px; overflow: hidden; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
        th, td { border-bottom: 1px solid #e2e8f0; padding: 12px; text-align: left; }
        th { background-color: #2b6cb0; color: white; font-weight: 600; }
        tr:nth-child(even) { background-color: #f8fafc; }
        tr:hover { background-color: #edf2f7; }
        .action-link { text-decoration: none; font-size: 1.1em; margin-right: 8px; }
        .checkbox-cell { text-align: center; width: 40px; }
        .checkbox-cell input { transform: scale(1.3); cursor: pointer; }
    </style>
</head>
<body>
    <h1>Gestión de Equipamientos Educativos</h1>
    <?= $mensaje ?>

    <form id="main-form" method="POST" action="index.php">
        <div class="toolbar">
            <a href="#form-section" onclick="document.getElementById('form-section').scrollIntoView();" class="icon-btn btn-add">➕ Añadir Equipamiento</a>
            <button type="submit" name="action" value="delete_selected" class="icon-btn btn-delete-bulk" onclick="return confirm('¿Seguro que deseas eliminar los equipamientos seleccionados?');">🗑️ Eliminar Seleccionados</button>
        </div>

        <?php if ($result && $result->num_rows > 0): ?>
            <table>
                <thead>
                    <tr>
                        <th class="checkbox-cell"><input type="checkbox" id="select-all" onclick="toggleSelectAll(this)"></th>
                        <th>ID</th>
                        <th>Código Registro</th>
                        <th>Nombre del Centro</th>
                        <th>Dirección</th>
                        <th>Distrito</th>
                        <th>Código Postal</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <?php while($row = $result->fetch_assoc()): ?>
                        <tr>
                            <td class="checkbox-cell"><input type="checkbox" name="selected_ids[]" value="<?= $row['id'] ?>"></td>
                            <td><?= htmlspecialchars($row['id']) ?></td>
                            <td><?= htmlspecialchars($row['register_id'] ?? 'N/D') ?></td>
                            <td><strong><?= htmlspecialchars($row['name'] ?? 'Sin Nombre') ?></strong></td>
                            <td><?= htmlspecialchars($row['addresses_road_name'] ?? 'N/D') ?></td>
                            <td><?= htmlspecialchars($row['addresses_district_name'] ?? 'N/D') ?></td>
                            <td><?= htmlspecialchars($row['addresses_zip_code'] ?? 'N/D') ?></td>
                            <td>
                                <a href="index.php?action=edit&id=<?= $row['id'] ?>#form-section" class="action-link" title="Editar">✏️</a>
                                <a href="index.php?action=delete&id=<?= $row['id'] ?>" class="action-link" title="Eliminar" onclick="return confirm('¿Deseas eliminar este equipamiento?');">❌</a>
                            </td>
                        </tr>
                    <?php endwhile; ?>
                </tbody>
            </table>
        <?php else: ?>
            <p>No se encontraron equipamientos registrados.</p>
        <?php endif; ?>
    </form>

    <br><br>

    <div class="form-card" id="form-section">
        <h2><?= $edit_data ? '✏️ Editar Equipamiento #' . htmlspecialchars($edit_data['id']) : '➕ Registrar Nuevo Equipamiento' ?></h2>
        <form action="index.php" method="POST">
            <input type="hidden" name="action" value="save">
            <?php if ($edit_data): ?>
                <input type="hidden" name="id" value="<?= htmlspecialchars($edit_data['id']) ?>">
            <?php endif; ?>

            <div class="form-grid">
                <div class="form-group">
                    <label>Código del Centro (register_id):</label>
                    <input type="text" name="register_id" required value="<?= htmlspecialchars($edit_data['register_id'] ?? '') ?>">
                </div>
                <div class="form-group">
                    <label>Nombre del Centro:</label>
                    <input type="text" name="name" required value="<?= htmlspecialchars($edit_data['name'] ?? '') ?>">
                </div>
                <div class="form-group">
                    <label>Dirección / Calle:</label>
                    <input type="text" name="addresses_road_name" value="<?= htmlspecialchars($edit_data['addresses_road_name'] ?? '') ?>">
                </div>
                <div class="form-group">
                    <label>Distrito:</label>
                    <input type="text" name="addresses_district_name" value="<?= htmlspecialchars($edit_data['addresses_district_name'] ?? '') ?>">
                </div>
                <div class="form-group">
                    <label>Código Postal:</label>
                    <input type="text" name="addresses_zip_code" value="<?= htmlspecialchars($edit_data['addresses_zip_code'] ?? '') ?>">
                </div>
            </div>

            <div class="form-actions">
                <button type="submit" class="icon-btn btn-add">💾 <?= $edit_data ? 'Guardar Cambios' : 'Guardar Equipamiento' ?></button>
                <?php if ($edit_data): ?>
                    <a href="index.php" class="icon-btn" style="background-color: #6c757d;">Cancelar</a>
                <?php endif; ?>
            </div>
        </form>
    </div>

    <script>
        function toggleSelectAll(source) {
            checkboxes = document.getElementsByName('selected_ids[]');
            for(var i=0, n=checkboxes.length; i<n; i++) {
                checkboxes[i].checked = source.checked;
            }
        }
    </script>
    <?php $conn->close(); ?>
</body>
</html>
EOF

echo "=== 4. Gestionando contenedor Docker ==="
if docker ps -a --format '{{.Names}}' | grep -q "^${CONTAINER_NAME}$"; then
    echo "Deteniendo y eliminando contenedor anterior..."
    docker stop "$CONTAINER_NAME"
    docker rm "$CONTAINER_NAME"
fi

echo "=== 5. Desplegando contenedor con php:8.2-apache ==="
docker run -d \
  --name "$CONTAINER_NAME" \
  --restart always \
  -p 80:80 \
  -v "$WEB_DIR":/var/www/html \
  php:8.2-apache

echo "=== 6. Instalando extensión mysqli en el contenedor ==="
docker exec -it "$CONTAINER_NAME" docker-php-ext-install mysqli
docker restart "$CONTAINER_NAME"

echo "=== Despliegue completado con éxito ==="
echo "Accede desde el navegador cliente a: http://192.168.10.10"
```

![creación contenedor.png](./img/creaci%C3%B3n%20contenedor.png)

---

## 9. Comprobación y Verificación del Servicio Web

Desde una máquina cliente ubicada dentro de la red corporativa, se efectúa la prueba de acceso abriendo el navegador web e introduciendo la IP del servidor web en la DMZ (`http://192.168.10.10` o `http://192.168.10.10/index.php`). Se confirma el correcto enrutamiento entre redes y el renderizado dinámico de los registros almacenados en la base de datos.

![comprobación web.png](./img/comprobaci%C3%B3n%20web.png)

---

## 10. Instalación y Puesta en Marcha de OpenSSH Server

Para posibilitar la administración y mantenimiento remoto seguro de la máquina virtual `W-05` desde otros puntos de la red, se instala el paquete `openssh-server` en la VM base y se habilita el demonio SSH para que inicie automáticamente junto con el sistema.

![instalacion de ssh.png](./img/instalacion%20de%20ssh.png)

---

## 11. Configuración del Firewall del Sistema (UFW)

Para securizar la máquina `W-05` dentro de la DMZ, se habilita el cortafuegos interno `ufw` permitiendo explícitamente únicamente el tráfico necesario para el funcionamiento del servidor web (puerto `80/TCP`) y para la administración remota (puerto `22/TCP`).

![configuración firewall.png](./img/configuraci%C3%B3n%20firewall.png)
