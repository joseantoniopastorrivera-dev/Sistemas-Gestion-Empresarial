# Guía para la validación en clase (UT02 · Odoo 17)

Orden para enseñar la práctica a la profesora, con los comandos de cada comprobación.
Las preguntas de la sección 2 son las que la profesora ha hecho a los compañeros.

## 0. Antes de empezar (cada vez que cambies de red)

1. **VirtualBox, antes de encender:** en las dos máquinas, Configuración → Red → Adaptador 1 → **Adaptador puente**, y en **Nombre** el MISMO adaptador en las dos (Wi-Fi `Intel Wireless-AC 9560` o cable `Killer E2500`).
2. Arranca **Ubuntu** y **Windows**.
3. En la consola de Ubuntu, como `alumno`:
   ```bash
   hostname -I                 # apunta la IP: cambia en cada red (casa 192.168.1.132, clase 10.45.178.x)
   sudo netplan apply          # solo si no sale IP o sale 169.254.x.x
   sudo systemctl start odoo   # el servicio está «disabled»: no arranca solo al encender
   sudo systemctl status odoo  # active (running) · salir con q
   ```
4. Si Windows pregunta por una red nueva («¿permitir que otros PC detecten tu PC?») → **No**.
5. Si `ssh` dice `REMOTE HOST IDENTIFICATION HAS CHANGED` (otra IP u otra máquina):
   ```
   ssh-keygen -R <IP>
   ```
6. Edge puede tener pestañas con la IP vieja: escribe siempre `http://<IP>:8069` (con `http://`, no `https://`).

En los comandos de abajo, `<IP>` es la IP que te ha dado `hostname -I`.

## 1. Demostración paso a paso

| Paso | Dónde | Qué enseñar |
|---|---|---|
| 1 | VirtualBox | Configuración de las dos máquinas (RAM, CPU, disco, red puente) |
| 2 | Consola Ubuntu | IP, Odoo `active (running)` |
| 3 | CMD de Windows | `ipconfig` (IP 10.45.x / 192.168.x) y `ping <IP>` con 0 % perdidos |
| 4 | CMD de Windows | `ssh alumno@<IP>` → `hostname -I` → `exit` |
| 5 | PuTTY | Host `<IP>`, puerto 22, SSH → `alumno` → `hostname -I` |
| 6 | Edge en Windows | `http://<IP>:8069` → login **admin** / admin → Ventas → presupuesto S00003 → PDF |
| 7 | SSH | `cat /home/odoo/conf/odoo-server.conf` y `cat /etc/systemd/system/odoo.service` |
| 8 | SSH | `psql`: `\du`, `\l`, `\connect test`, `\dt` (366 tablas) |
| 9 | pgAdmin | Si la IP cambió: clic derecho en cada servidor → Properties → Connection → Host = `<IP>`. Enseñar los dos servidores (postgres y odoo) y test → Tables (366) |

## 2. Preguntas de la profesora y cómo responderlas

### 2.1. Requerimientos, usuarios creados y cambiar entre ellos

**Requerimientos de la máquina** (enunciado: 2048 MB, 2 CPU, 30 GB dinámico, adaptador puente, IP por DHCP):
- En VirtualBox: seleccionar la máquina y enseñar la pestaña **Detalles** (Sistema, Almacenamiento, Red).
- En Ubuntu:
  ```bash
  cat /etc/os-release | head -4   # Ubuntu 24.04.5 LTS
  hostnamectl                     # nombre: ubuntu-server-24-lts
  free -h                         # ~2 GB de RAM
  nproc                           # 2 CPU
  df -h /                         # disco raíz
  ip a                            # enp0s3 con IP por DHCP (dynamic)
  ```

**Usuarios creados:**
```bash
getent passwd alumno odoo   # existen, con su home (/home/alumno y /home/odoo)
groups alumno               # alumno está en el grupo sudo (administrador)
groups odoo                 # odoo NO está en sudo (mínimo privilegio)
```

**Cambiar entre ellos:**
```bash
whoami        # alumno
su - odoo     # contraseña: odoo
whoami        # odoo
pwd           # /home/odoo
ls            # conf  odoo
exit          # vuelve a alumno
```
Para la defensa: `su - odoo` carga el entorno completo del usuario (su home y variables); sin el guion solo cambia de usuario.

### 2.2. PuTTY instalado y conectividad por SSH con el servidor de Odoo

**PuTTY instalado** (en Windows):
- Menú Inicio → escribir `putty` → aparece la aplicación.
- O en CMD:
  ```
  dir "C:\Program Files\PuTTY"
  "C:\Program Files\PuTTY\plink.exe" -V
  ```
  (`plink` es la versión de consola de PuTTY y muestra la versión 0.85.)

**Conectividad por SSH:**
1. PuTTY → `<IP>`, puerto 22 → `alumno` → contraseña.
2. Ya dentro, demostrar que es el servidor de Odoo y que la conexión viene de Windows:
   ```bash
   hostname -I                   # IP de Ubuntu
   who                           # sesión de alumno con la IP de Windows entre paréntesis
   systemctl status odoo         # active (running)
   sudo ss -tlnp | grep 8069     # Odoo escuchando en el puerto 8069
   ```
3. Lo mismo desde CMD: `ssh alumno@<IP>`.

### 2.3. Gestor y cliente de PostgreSQL instalados, servidor arrancado, rol postgres y rol odoo

**Paquetes instalados:**
```bash
dpkg -l postgresql postgresql-client   # «ii» = instalado
psql --version                         # cliente: psql 16.x
pg_lsclusters                          # clúster 16 main, estado «online», puerto 5432
```

**Servidor arrancado:**
```bash
systemctl status postgresql@16-main    # active (running)  ← este es el servidor real
systemctl status postgresql            # active (exited): solo lanza los clústeres, es normal
sudo ss -tlnp | grep 5432              # escuchando en el puerto 5432
```

**Demo: usuario postgres y rol odoo con Create DB:**
```bash
sudo -u postgres psql -c "\du"
```
Debe salir:
- `postgres` → Superuser, Create role, Create DB, Replication, Bypass RLS (el superusuario que crea PostgreSQL al instalarse).
- `odoo` → **Create DB** (creado con `createuser --createdb --pwprompt odoo`).

Para la defensa: el **usuario de Linux** `postgres` y el **rol de PostgreSQL** `postgres` son cuentas distintas. Se puede enseñar:
```bash
getent passwd postgres                 # usuario de Linux
sudo -u postgres psql -c "\du"         # rol de PostgreSQL
```

### 2.4. Conexión a Odoo desde Windows y dependencias instaladas

**Conexión a Odoo desde Windows:**
1. Edge → `http://<IP>:8069` (escribir `http://`; si sale ERR_SSL_PROTOCOL_ERROR es que ha puesto https).
2. Correo electrónico: **admin** · contraseña: **admin** (la base de datos se llama `test`, el usuario es `admin`).
3. Enseñar Ventas → Presupuestos → S00003 → Vista previa → PDF (demuestra wkhtmltopdf).

**Dependencias instaladas** (por SSH):
```bash
git --version                # git
python3 --version            # Python 3.12.3 (Odoo 17 necesita ≥ 3.10)
pip3 --version               # pip 24.0
wkhtmltopdf --version        # 0.12.6.1 (with patched qt)
dpkg -l wkhtmltox            # paquete instalado desde el .deb jammy_amd64
dpkg -l python3-psycopg2 python3-lxml python3-reportlab python3-babel   # algunas instaladas por debinstall.sh
```
Para la defensa: `debinstall.sh` lee `debian/control` (las dependencias que declara Odoo) e instala esos paquetes con apt. Se puede enseñar el archivo:
```bash
grep -A40 "^Depends" /home/odoo/odoo/debian/control | head -40
```

## 3. Datos útiles

| Dato | Valor |
|---|---|
| Usuario Ubuntu administrador | alumno |
| Usuario que ejecuta Odoo | odoo / odoo |
| Rol PostgreSQL de Odoo | odoo / odoo (Create DB) |
| Superusuario PostgreSQL | postgres / postgres |
| Odoo | base de datos `test`, usuario `admin` / admin |
| Windows | usuario local `alumno` |
| Archivo de configuración | /home/odoo/conf/odoo-server.conf |
| Servicio | /etc/systemd/system/odoo.service |
| PostgreSQL | /etc/postgresql/16/main/postgresql.conf y pg_hba.conf |
