# Guía paso a paso — UT02 Práctica 1: Odoo 17 sobre Ubuntu Server 24 LTS

> Cada 📸 es una captura que deberías sacar. Cada 💾 es una **instantánea** (snapshot) de VirtualBox que te recomiendo hacer (lo pide el enunciado).
> Cada ❓ es una pregunta de defensa: intenta contestarla tú antes de mirar nada; si no te sale, pregúntamela.
> Cada ⚠️ es un fallo típico o una errata del enunciado. **Si te pasa, es una INCIDENCIA: captura + apúntala.**

---

## Paso 0 — Preparar el disco (lo que te rompió la primera vez)

Tu `C:` tiene solo **~4 GB libres** y tu `E:` tiene **~648 GB libres**. Las dos VMs van a ocupar ~40-60 GB entre discos virtuales e instantáneas, así que **todo lo de VirtualBox tiene que vivir en `E:`**.

1. Borra la VM a medias de la vez anterior: en VirtualBox → clic derecho sobre ella → **Eliminar → Borrar todos los archivos**. (Solo quitarla de la lista no libera espacio.)
2. Crea la carpeta `E:\VirtualBox VMs`.
3. VirtualBox → **Archivo → Preferencias → General → Carpeta predeterminada de máquinas** → `E:\VirtualBox VMs`.
4. (Recomendado) Copia las dos ISO a `E:\ISOs\` y úsalas desde ahí.

📸 Preferencias con la carpeta en `E:`.

**Esto es tu INCIDENCIA nº 1** para el informe: "Al crear la base de datos / [lo que estuvieras haciendo en la captura 60] el disco C: se quedó sin espacio. Causa: las VMs se guardaban en la carpeta por defecto de C:. Solución: eliminar la VM, cambiar la carpeta predeterminada de máquinas a E: y repetir la instalación." Guarda la `60.png` como evidencia.

❓ ¿Qué significa "reserva dinámica" en un VDI y por qué aun así te quedaste sin espacio?

---

## Paso 1 — Máquina virtual Ubuntu Server 24 LTS

### 1.1 Crear la VM
VirtualBox → **Nueva**:

| Campo | Valor |
|---|---|
| Nombre | `Ubuntu Server 24 LTS` |
| Carpeta | `E:\VirtualBox VMs` (comprueba que sale) |
| Imagen ISO | `ubuntu-24.04.2-live-server-amd64.iso` |
| ⚠️ **Omitir instalación desatendida** | **MARCADO** (si no, VirtualBox instala Ubuntu solo y no podrás sacar capturas ni elegir OpenSSH) |
| RAM | 2048 MB |
| CPUs | 2 |
| Disco | 30 GB, VDI, **reservado dinámicamente** (sin marcar "pre-asignar tamaño completo") |

Antes de arrancar: **Configuración → Red → Adaptador 1 → Conectado a: Adaptador puente** → elige tu tarjeta real (la Wi-Fi o la Ethernet que estés usando).

📸 Cada pantalla del asistente + la pestaña de Red.

❓ ¿Qué diferencia hay entre NAT y adaptador puente? ¿Por qué con NAT no podrías entrar desde la VM de Windows?

### 1.2 Instalar Ubuntu Server
Arranca la VM y en el instalador:

1. Idioma: Español. Teclado: Spanish.
2. Tipo de instalación: **Ubuntu Server** (no "minimized").
3. Red: deja **DHCP**; apunta la IP que te da (ya sale aquí).
4. Proxy: vacío. Mirror: el que propone.
5. Almacenamiento: **Usar un disco entero** + LVM.
   ⚠️ Truco: en la pantalla de resumen, `ubuntu-lv` solo usa la mitad del disco (~14 GB). Puedes editarlo (`ubuntu-lv` → Editar → tamaño máximo) para usar los 30 GB. No es obligatorio, pero evita sustos de espacio.
6. Perfil:
   - Tu nombre: `alumno`
   - Nombre del servidor: ⚠️ el enunciado dice `Ubuntu Server 24 LTS`, pero **un hostname no admite espacios**. Pon `ubuntu-server-24-lts` y explícalo en el informe (es un buen detalle para el 10).
   - Usuario: `alumno` / Contraseña: `alumno`
7. Ubuntu Pro: saltar.
8. **SSH: marca "Instalar servidor OpenSSH".**
9. Snaps destacados: **no marques ninguno** (el enunciado lo pide expresamente).
10. Al terminar: **Reiniciar ahora**. Si se queda en "Please remove the installation medium", pulsa Enter (VirtualBox ya expulsa la ISO).

📸 Cada pantalla del instalador, sobre todo la de SSH y la de snaps vacía.

💾 Instantánea: `01 - Ubuntu recién instalado`.

---

## Paso 2 — Login y comprobar la IP

```bash
ip a
```
- `ip a` muestra todas las interfaces de red. Busca la que **no** es `lo` (será algo como `enp0s3`) y su línea `inet 192.168.x.x/24`. Eso es tu IP. **Apúntala.**
- ¿Por qué no `ifconfig`? Porque en Ubuntu Server 24 ya no viene instalado (es del paquete antiguo `net-tools`); `ip` es la herramienta moderna.

Alternativa más corta: `hostname -I`.

📸 Login + salida de `ip a` con la IP resaltada.

❓ Si mañana reinicias el router, ¿puede cambiar la IP? ¿Por qué? (pista: DHCP)

---

## Paso 3 — Máquina virtual Windows

Igual que el paso 1 pero con `Win10_22H2_Spanish_x64.iso`:

- Nombre: `Windows 10`, carpeta en `E:`, **Omitir instalación desatendida** marcado.
- RAM: 4096 MB (2 GB va muy justo para Windows 10), 2 CPUs, disco 50 GB dinámico.
- Red: **Adaptador puente** (la misma tarjeta que Ubuntu → así están en la misma red).
- Instalación: "No tengo clave de producto" → Windows 10 Pro → Personalizada → disco sin asignar.
- Cuenta: si te obliga a cuenta Microsoft, elige **Cuenta sin conexión / Experiencia limitada**.
- Tras instalar: **Dispositivos → Insertar imagen de CD de Guest Additions** e instálalas (te da portapapeles compartido y pantalla ajustable).

📸 Asistente, instalación, escritorio final.
💾 Instantánea: `01 - Windows recién instalado`.

⚠️ RAM del host: con las dos VMs encendidas consumes 6 GB. Si tu PC tiene 8 GB, cierra el navegador mientras trabajas.

---

## Paso 4 — Conectarse por SSH desde Windows

### 4a. PuTTY
1. En la VM de Windows, descarga PuTTY desde la web oficial (`putty.org` → enlace a la página de Simon Tatham) e instálalo.
2. Host Name: `IP_de_Ubuntu`, Port 22, SSH → Open.
3. Primera vez: "Security Alert" sobre la clave del servidor → **Accept**.
4. `login as: alumno` + contraseña.

### 4b. CMD
```cmd
ssh alumno@IP_de_Ubuntu
```
- `ssh` viene incluido en Windows 10 (cliente OpenSSH).
- `alumno@IP` = "entra como el usuario `alumno` en la máquina con esa IP".
- Te preguntará `Are you sure you want to continue connecting (yes/no)?` → `yes`.

📸 Las dos conexiones con el prompt `alumno@ubuntu-server-24-lts:~$` visible.

❓ ¿Qué es la "huella" (fingerprint) que te pide aceptar la primera vez y qué te protege?

> 💡 A partir de aquí te recomiendo trabajar **desde PuTTY o CMD**: puedes copiar y pegar comandos, y las capturas salen más limpias que la consola de VirtualBox.

---

## Paso 5 — Instalar Odoo 17 desde el código fuente

Antes de empezar, actualiza el sistema (con `alumno`):
```bash
sudo apt update && sudo apt upgrade -y
```
- `apt update` descarga la lista actualizada de paquetes (no instala nada).
- `apt upgrade -y` instala las actualizaciones; `-y` contesta "sí" automáticamente.
- `&&` ejecuta el segundo solo si el primero ha ido bien.

### 5.b.i — Usuario `odoo`
```bash
sudo adduser odoo
```
Contraseña `odoo`; el resto de datos (nombre completo, teléfono…) Enter.

- **¿Por qué un usuario aparte?** Principio de mínimo privilegio: Odoo es un servidor web expuesto a la red. Si alguien lo compromete, solo tiene los permisos de `odoo`, no los de `alumno` (que es sudoer). Por eso `odoo` **no** debe estar en el grupo `sudo`.

📸

### 5.b.ii — git
```bash
git --version
```
En Ubuntu Server 24 ya viene. Si no: `sudo apt install git -y`.

📸

### 5.b.iii — Clonar Odoo 17 (como `odoo`)
```bash
su - odoo
git clone https://github.com/odoo/odoo.git --depth 1 --branch 17.0 --single-branch
ls
```
- `su - odoo`: cambia al usuario `odoo`. ⚠️ El enunciado pone `su-odoo` todo junto: **es una errata, lleva espacios: `su - odoo`**. El guion hace que cargue el entorno de `odoo` y te deje en su home (`/home/odoo`).
- `--depth 1`: solo el último commit (sin historial) → mucho menos espacio.
- `--branch 17.0 --single-branch`: solo la rama 17.0.
- `ls`: debe aparecer la carpeta `odoo` → el código queda en `/home/odoo/odoo`.

Tarda unos minutos. 📸 inicio y final del clonado + `ls`.
```bash
exit
```
(vuelves a `alumno`).

💾 `02 - Usuario odoo + código clonado`.

### 5.b.iv — Python y pip
```bash
python3 --version
pip3 --version
```
- ⚠️ El enunciado pone `pip3 --versión` con tilde: **es `--version`** sin tilde.
- Lo normal es que `pip3` **no** esté instalado y te diga `Command 'pip3' not found, but can be installed with: sudo apt install python3-pip`. Hazle caso (con `alumno`):
```bash
sudo apt install python3-pip -y
pip3 --version
```
📸 las tres salidas (incluido el error: es una mini-incidencia resuelta).

⚠️ Concepto importante: en Ubuntu 24 **no uses `pip install` a pelo** para paquetes del sistema; da el error `externally-managed-environment` (PEP 668). Por eso el enunciado instala las dependencias con `apt` (paso vi), no con pip.

### 5.b.v — PostgreSQL
```bash
sudo apt install postgresql postgresql-client -y
```
- `postgresql`: el servidor (el "motor" que guarda los datos).
- `postgresql-client`: el programa `psql` para hablar con él desde terminal.

Crear el rol `odoo` en PostgreSQL:
```bash
sudo su postgres
createuser --createdb --pwprompt odoo
exit
```
- `sudo su postgres`: te conviertes en el usuario de sistema `postgres`, el único que de inicio puede administrar PostgreSQL.
- `createuser ... odoo`: crea el **rol** `odoo` dentro de PostgreSQL.
  - `--createdb`: puede crear bases de datos (Odoo lo necesita para el botón "Create database").
  - `--pwprompt`: te pide la contraseña → `odoo` (dos veces).
- `exit`: vuelves a `alumno`.

⚠️ Si sale `could not connect to server ... .s.PGSQL.5432` (el servidor no está arrancado):
```bash
pg_lsclusters                 # como alumno: mira versión y nombre del cluster (16 main)
sudo su postgres
pg_ctlcluster 16 main start   # ⚠️ el enunciado pone "pg_clusters": la orden correcta es pg_ctlcluster
createuser --createdb --pwprompt odoo
exit
```

📸 instalación + createuser.

❓ **Concepto clave para la defensa:** ahora existen *dos* cosas llamadas `odoo`: el usuario de **Linux** `odoo` (paso i) y el rol de **PostgreSQL** `odoo` (este paso). ¿Son lo mismo? ¿Tienen por qué tener la misma contraseña?

### 5.b.vi — Dependencias
```bash
sudo /home/odoo/odoo/setup/debinstall.sh
```
- Script que trae Odoo: lee `debian/control` (la lista de paquetes que Odoo necesita) y los instala con `apt`.
- Se lanza con `alumno` porque necesita `sudo` (y `odoo` no es sudoer, justo por lo del mínimo privilegio).

Tarda bastante. 📸 inicio y final.
💾 `03 - PostgreSQL + dependencias`.

### 5.b.vii — Ejecutar Odoo
```bash
su - odoo
cd odoo
python3 odoo-bin --addons-path=addons
```
- ⚠️ En el PDF sale `–addons-path` con un guion largo (–) porque Word lo autocorrige. **Son dos guiones normales `--`.** Si copias del PDF, fallará.
- `odoo-bin`: el ejecutable de Odoo. No se "instala": se ejecuta desde el código fuente.
- `--addons-path=addons`: dónde están los módulos (Ventas, Contabilidad…).

Debe quedarse escribiendo logs y aparecer algo como `HTTP service (werkzeug) running on ...:8069`. **No cierres esa terminal** (si la cierras, se para Odoo).

En la VM de Windows (o en tu PC), navegador → `http://IP_de_Ubuntu:8069`.

Formulario:
| Campo | Valor |
|---|---|
| Master Password | **la que sugiere** → cópiala a un txt, la necesitarás |
| Database Name | `test` |
| Email | `admin` |
| Password | `admin` |
| Phone | vacío |
| Language | Spanish / Español |
| Country | Spain |
| Demo data | ✅ |

**Create database** → tarda 1-3 min (aquí, con los datos demo, es donde se te llenó el disco la otra vez). Luego login `admin`/`admin`.

📸 terminal con Odoo arrancado, formulario relleno, login, página de aplicaciones.
💾 `04 - Odoo funcionando con BD test` (para la VM, `Ctrl+C` en la terminal antes es lo más limpio).

❓ ¿Qué es el puerto 8069? ¿Qué pasaría si intentas arrancar Odoo dos veces a la vez?

### 5.b.viii — wkhtmltopdf
Con `alumno` (si estás como `odoo`, `exit`):

1. En un navegador ve a `https://github.com/wkhtmltopdf/packaging/releases`, busca la versión **0.12.6.1** y el fichero que acaba en **`jammy_amd64.deb`**. Clic derecho → **Copiar dirección del enlace**. 📸
2. Descárgalo en Ubuntu:
```bash
cd ~
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb
```
   (Usa la URL que copiaste tú; esta es la de la release `0.12.6.1-2`.) `wget` descarga un fichero de una URL a la carpeta actual.
3. Instálalo:
```bash
sudo apt install ./wkhtmltox_0.12.6.1-2.jammy_amd64.deb -y
wkhtmltopdf --version
```
   - ¿Por qué `apt install ./fichero.deb` y no `dpkg -i`? `dpkg -i` instala el .deb pero **no** resuelve sus dependencias (`xfonts-75dpi`, `xfonts-base`…) y te da error; `apt` sí las descarga. El `./` es obligatorio para que apt sepa que es un fichero local y no el nombre de un paquete.
   - ❓ "jammy" es Ubuntu 22.04 y tú tienes 24.04 ("noble"). ¿Por qué aun así funciona? (pista: no hay .deb para noble de esa versión, y las librerías que usa siguen existiendo).
4. Rearranca Odoo (`Ctrl+C` si estaba en marcha):
```bash
su - odoo
cd odoo
python3 odoo-bin --addons-path=addons
```
5. Navegador → Aplicaciones → **Ventas → Activar**. Abre un presupuesto/pedido de los datos demo → **Vista previa** → debe verse/descargarse el PDF.

📸 descarga, instalación, versión, módulo Ventas instalado, PDF generado.
💾 `05 - wkhtmltopdf`.

### 5.b.ix — Archivo de configuración
Como `odoo`, en su home:
```bash
cd ~
ls -a
mkdir conf
cp .odoorc conf/odoo-server.conf
nano conf/odoo-server.conf
```
- `ls -a`: muestra también los ocultos (los que empiezan por `.`) → verás `.odoorc`. Se creó al pulsar "Create database" porque Odoo guardó ahí la master password.
- `cp origen destino`: copia y renombra a la vez.
- `nano`: editor de texto de terminal (`Ctrl+O` guardar, Enter, `Ctrl+X` salir).

Deja/modifica estas líneas (sin tocar `admin_passwd`):
```ini
addons_path = /home/odoo/odoo/odoo/addons,/home/odoo/odoo/addons
db_host = 127.0.0.1
db_port = 5432
db_user = odoo
db_password = odoo
```
- `addons_path`: las dos carpetas de módulos: los del núcleo (`odoo/odoo/addons`) y los estándar (`odoo/addons`). Rutas absolutas, separadas por coma (mejor sin espacio).
- `db_host = 127.0.0.1`: PostgreSQL está en la misma máquina.
- `db_port = 5432`: puerto por defecto de PostgreSQL.
- `db_user`/`db_password`: el **rol de PostgreSQL** que creaste en el paso v.

Arranca con el fichero:
```bash
cd odoo
python3 odoo-bin -c ../conf/odoo-server.conf
```
⚠️ El enunciado pone `-c conf/odoo-server.conf` estando dentro de `/home/odoo/odoo`, pero la carpeta `conf` está en `/home/odoo/conf`. Desde `~/odoo` la ruta relativa correcta es `../conf/odoo-server.conf` (o usa la absoluta `/home/odoo/conf/odoo-server.conf`). Si usas la del enunciado tal cual te dirá que no encuentra el fichero → otra incidencia resuelta y documentada.

📸 `ls -a`, contenido del fichero, Odoo arrancado con `-c`.
💾 `06 - Fichero de configuración`.

### 5.b.x — Odoo como servicio (systemd)
Para Odoo si está en marcha (`Ctrl+C`), vuelve a `alumno` (`exit`) y crea el servicio:
```bash
sudo nano /etc/systemd/system/odoo.service
```
Contenido — **compáralo con la imagen del PDF y usa el de tu profesora si difiere** (esa imagen no se puede extraer como texto):
```ini
[Unit]
Description=Odoo 17
After=network.target postgresql.service

[Service]
Type=simple
User=odoo
Group=odoo
ExecStart=/usr/bin/python3 /home/odoo/odoo/odoo-bin -c /home/odoo/conf/odoo-server.conf --logfile=/home/odoo/odoo-server.log --dev=all
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Línea a línea:
- `[Unit]` → qué es y cuándo arrancar. `After=` = "arranca después de la red y de PostgreSQL" (sin base de datos Odoo no sirve).
- `[Service]` → cómo arrancar. `User/Group=odoo` → se ejecuta como `odoo`, **no como root**. `ExecStart` → el comando, siempre con **rutas absolutas** (systemd no tiene "carpeta actual" como tu terminal). `--logfile` → los logs van a un fichero en vez de a la pantalla. `--dev=all` → modo desarrollador (recarga al cambiar código, etc.). `Restart=on-failure` → si se cae, lo relanza.
- `[Install]` → `WantedBy=multi-user.target` → con `enable`, arranca solo al encender el servidor.

Probar todo:
```bash
sudo systemctl daemon-reload     # systemd relee los .service (necesario tras crear/editar uno)
sudo systemctl start odoo
sudo systemctl status odoo       # debe decir active (running). Sal con q
sudo systemctl stop odoo
sudo systemctl status odoo       # inactive (dead)
sudo systemctl restart odoo
sudo systemctl status odoo
sudo systemctl enable odoo       # arranque automático al encender
```
Comprueba en el navegador que Odoo responde con el servicio arrancado.

📸 fichero, cada comando con su `status`, navegador.
💾 `07 - Servicio odoo`.

❓ ¿Qué ventaja tiene esto frente a lanzarlo a mano con `python3 odoo-bin`? ¿Qué pasaba si cerrabas PuTTY antes?

---

## Paso 6 — Acceder a la BD con `psql`

Con `alumno`:
```bash
sudo passwd postgres
```
Contraseña `postgres`. (Esto da contraseña al **usuario de Linux** `postgres`, para poder hacer login con él.)

```bash
su - postgres
psql
```
Dentro de `psql` (prompt `postgres=#`):
```sql
\du
\l
\connect test
\dt
\q
```
- `\du`: roles → debes ver `odoo` (con *Create DB*, sin *Superuser*) y `postgres` (Superuser).
- `\l`: bases de datos → verás `test`.
- `\connect test` (o `\c test`): te conectas a la BD de Odoo.
- `\dt`: tablas de esa BD (cientos; pulsa `q` para salir del paginador).
- `\q`: salir.

⚠️ El enunciado pone `\CREATE DATABASE` y `\CREATE TABLE` con barra: **es un error**. Las órdenes con `\` son comandos internos de psql; `CREATE DATABASE` es **SQL** y va sin barra y con `;` al final. Si quieres probarlo:
```sql
CREATE DATABASE prueba;
\l
DROP DATABASE prueba;
```

📸 cada comando.

---

## Paso 7 — pgAdmin desde Windows

### 7.i-ii Instalar
En la VM de Windows: `https://www.pgadmin.org/download/` → Windows → última versión → instalar con todo por defecto. 📸

### 7.iii Primer servidor (usuario `postgres`)
pgAdmin → **Register → Server** (o "Add New Server"):
- General: Name `Odoo Ubuntu - postgres`, Comments `Conexión superusuario a PostgreSQL de Ubuntu Server`.
- Connection: Host `IP_de_Ubuntu`, Port `5432`, Maintenance DB `postgres`, Username `postgres`, Password `postgres`.

**Antes de Save** — en Ubuntu, dentro de `psql` como `postgres`:
```sql
ALTER USER postgres WITH PASSWORD 'postgres';
```
❓ **El concepto que más se pregunta:** en el paso 6 ya pusiste contraseña a `postgres` con `passwd`. ¿Por qué hace falta otra? (Pista: `passwd` cambia la contraseña del usuario **de Linux**; `ALTER USER` cambia la del **rol de PostgreSQL**, que es la que comprueba pgAdmin. Son dos sistemas de usuarios distintos que casualmente se llaman igual.)

Pulsa **Save** → **fallará** (lo dice el enunciado). 📸 el error: es esperado y es parte de la práctica.

### 7.2 Por qué falla y cómo arreglarlo
Por defecto PostgreSQL solo escucha en `localhost` y solo acepta conexiones locales. Hay que tocar dos ficheros como `root`.

Dar contraseña a root y entrar (con `alumno`):
```bash
sudo passwd root
su -
```

**postgresql.conf** (qué interfaces de red escucha):
```bash
nano /etc/postgresql/16/main/postgresql.conf
```
`Ctrl+W` para buscar `listen_addresses`. Deja:
```
#listen_addresses = 'localhost'
listen_addresses = '*'
```
`'*'` = escucha en todas las interfaces (no solo la de loopback).

**pg_hba.conf** (quién puede autenticarse y cómo; *hba* = host-based authentication):
```bash
nano /etc/postgresql/16/main/pg_hba.conf
```
Bajo `# IPv4 local connections:`:
```
#host    all    all    127.0.0.1/32    scram-sha-256
host     all    all    all             scram-sha-256
```
Columnas: tipo (`host` = por red TCP), base de datos, usuario, dirección de origen (`all` = cualquier IP), método (`scram-sha-256` = contraseña cifrada).

Reiniciar:
```bash
systemctl restart postgresql
systemctl status postgresql
exit
```
⚠️ Si tocas `postgresql.conf` y luego PostgreSQL no arranca, casi siempre es una comilla mal puesta: usa comillas rectas `'*'`, no las curvas `‘*’` del PDF.

Vuelve a pgAdmin → Save / Connect → ✅. Despliega **Databases → test → Schemas → public → Tables** → verás las tablas de Odoo (el enunciado dice 366; con Ventas instalado tendrás más, explícalo).

📸 edición de cada fichero, reinicio, conexión correcta, árbol de tablas.

❓ Abrir `listen_addresses='*'` + `all all all` en producción, ¿es buena idea? ¿Qué pondrías en vez de `all` en la columna de dirección?

### 7.7 Segundo servidor (usuario `odoo`)
Register → Server: Name `Odoo Ubuntu - odoo`, Host IP, Port 5432, Maintenance DB `postgres`, Username `odoo`, Password `odoo` → Save.

📸 el **Object Explorer con las dos conexiones**.
💾 `08 - Práctica terminada` (en las dos VMs).

---

## Paso 8 — El informe (esto es lo que separa un 8 de un 10)

Checklist del enunciado:
- [ ] Portada (nombres y primer apellido de los dos, curso, fecha, profesora).
- [ ] **Índice automático** (Word → Referencias → Tabla de contenido; usa estilos *Título 1/2/3*).
- [ ] **Índice de incidencias** (pon cada incidencia con estilo propio o como título de nivel 3 con prefijo "Incidencia X", o usa *Insertar título* con etiqueta "Incidencia" + *Insertar tabla de ilustraciones* de esa etiqueta).
- [ ] Páginas numeradas.
- [ ] Arial 12, márgenes 2 cm.
- [ ] Cada paso: texto explicativo + captura.
- [ ] Incidencias **resueltas o no**: qué pasó, causa, solución, captura.
- [ ] **Uso de IA**: el enunciado obliga a poner la herramienta y el prompt. Pon "Claude (Claude Code)" y pega el prompt que me has escrito.
- [ ] Entregar en **PDF y Word**, nombrados con nombre y primer apellido.
- [ ] Enseñar a la profesora el final de cada paso (checklist en clase).

Incidencias que ya tienes o probablemente tendrás (todas cuentan):
1. Disco C: lleno → VMs movidas a E:.
2. Hostname con espacios no permitido.
3. `pip3` no instalado.
4. Erratas del enunciado (`su-odoo`, `–addons-path`, `pg_clusters`, `\CREATE`, ruta `conf/`).
5. pgAdmin no conecta → `postgresql.conf` + `pg_hba.conf`.

### Defensa (2 minutos)
Elige 2 problemas y un aprendizaje. Propuesta: (1) el disco lleno y por qué un disco dinámico crece, (2) pgAdmin no conectaba y qué hacen `listen_addresses` y `pg_hba.conf`, (3) aprendizaje: usuarios de Linux vs roles de PostgreSQL.
