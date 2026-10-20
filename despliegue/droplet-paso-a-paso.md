# Droplet de DigitalOcean — paso a paso

**Actualizado:** 20/10/2026 · **Contraste con:** la auditoría de solo lectura del Droplet `dev-erp-millennium-mobility-sac` del 20/10/2026 · **Acompaña a:** [droplet-digitalocean.md](droplet-digitalocean.md) (el porqué de cada pieza, las lecciones y el diagnóstico) y [linux-basico.md](linux-basico.md) (carpetas, comandos y permisos de Linux). Si este documento y el código del backend no coinciden, manda el código.

Esta guía sirve para dos cosas:

1. **Montar un backend Spring Boot en un Droplet nuevo**, cualquiera. Cada fase trae la configuración **ideal** como plantilla: los comandos y archivos usan marcadores como `<ip_del_droplet>`, que se reemplazan, **signos `< >` incluidos**, por los valores de la implementación.
2. **Saber qué le falta a nuestro Droplet.** Cada fase termina con un cuadro **En este Droplet** que compara el ideal con lo que encontró la auditoría:
   - **Igual**: está como en el ideal.
   - **Distinto**: existe, pero de otra forma; se dice si importa.
   - **Falta**: no existe todavía.

Los comandos se corren en el Droplet como `root` (o con `sudo`), salvo los que dicen *en la laptop*. Ningún secreto se escribe en la línea de comandos ni en este documento: los valores secretos se escriben dentro de `nano`.

---

## Los marcadores

| Marcador | Qué es | En este Droplet |
|---|---|---|
| `<ip_del_droplet>` | La IPv4 pública del Droplet | `137.184.231.160` |
| `<hostname>` | El nombre del servidor | `dev-erp-millennium-mobility-sac` |
| `<app>` | Nombre corto de la aplicación: se usa en carpetas, servicio y logs | `millennium-erp` |
| `<usuario_app>` | El usuario que ejecuta Java | `app-millennium-erp` |
| `<usuario_deploy>` | El usuario con el que entra GitHub Actions | `deploy-millennium-erp` |
| `<dominio_api>` | El nombre público del backend | `api.leasing.dev.millennium.ky` |
| `<dominio_frontend>` | El nombre del frontend, que reenvía `/api/` al backend | `dev.erp.millennium.ky` |
| `<puerto_backend>` | El puerto en el que escucha Java, solo en `127.0.0.1` | `8080` |
| `<perfil_spring>` | El perfil de Spring | `prod` |
| `<host_bd>` | El host del clúster de PostgreSQL | No se escribe aquí |
| `<puerto_bd>` | El puerto del clúster | `25060` |
| `<nombre_bd>` | La base de datos | `millennium_erp_dev` |
| `<cuenta_bd_app>` | La cuenta con la que se conecta la aplicación | `erp_mm_app` |
| `<cuenta_bd_migraciones>` | La cuenta dueña, con la que migra Flyway | `erp_mm_owner` |
| `<tu_ip_de_oficina>` | La IP desde la que se administra, para limitar SSH | Sin definir |
| `<nombre_llave_admin>` | Etiqueta de la llave del administrador | Por crear |
| `<nombre_llave_deploy>` | Etiqueta de la llave de GitHub Actions | `github-actions-millennium-dev` |
| `<huella_host>` | La huella ED25519 del servidor | `SHA256:eWy2BzWHuFSaHGFO8ABXcX/0qIp0YJ1N1pKek1vCmXw` |
| `<porcentaje_heap>` | Parte de la RAM para Java: `50` con 2 GB, `60`–`70` con 4 GB o más | `50` |
| `<tamaño_swap>` | El swap: igual a la RAM hasta 2 GB, 2 GB por encima | `2G` |
| `<repo_backend>` | El repositorio en GitHub | `erp-millennium-mobility-sac/erp-millennium-mobility-backend` |
| `<environment_github>` | El environment de GitHub con los secretos | `development` |
| `<rama>` | La rama que se despliega | `develop` |

---

## Qué le falta a este Droplet

Lo que hay hoy **es válido**: si se copiara un JAR a `current` y se arrancara el servicio, el backend respondería por Nginx en `:80`. Lo que falta se divide así:

| Punto | Hoy | Ideal | Importancia | Fase |
|---|---|---|---|---|
| Probar las dos cuentas de la base | Solo red y TLS comprobados | `psql` entra con las dos | **Necesario** antes del primer arranque | [6](#6-base-de-datos-administrada) |
| HTTPS | Cloudflare no tiene certificado para el nombre; Nginx solo en `:80` | Certificado de borde y de origen, `:443`, Full (strict) | **Necesario** para terminar | [11](#11-https-con-cloudflare) |
| `DROPLET_KNOWN_HOSTS` | Probablemente cargado desde un archivo vacío o de `127.0.0.1` | La línea de `<ip_del_droplet>`, comprobada por huella | **Necesario** para terminar | [12](#12-github) |
| `deploy-dev.yml` | No existe | Workflow manual con activación y vuelta atrás | **Necesario** para terminar | [12](#12-github) |
| Primer despliegue | — | Backend sano, `enable`, `BACKEND_ORIGIN` apuntando aquí | **Necesario** para terminar | [13](#13-primer-despliegue) |
| SSH | Root entra con contraseña desde cualquier IP | Solo con llave | **Seguridad**: muy recomendado | [5](#5-acceso-del-administrador-y-firewall) |
| Firewall | Ninguno | Solo 22, 80 y 443 | **Seguridad**: muy recomendado | [5](#5-acceso-del-administrador-y-firewall) |
| `DO_ADMIN_*` en `backend.env` | Presentes, con clave | En un archivo aparte que systemd no carga | **Seguridad** | [7](#7-backendenv) |
| `/root/github-deploy-setup` | Conserva la llave privada de deploy | Borrada después de probar Actions | **Seguridad** | [12](#12-github) |
| Swap | Ninguno | `<tamaño_swap>` | **Estabilidad** | [2](#2-paquetes-java-y-swap) |
| `StartLimit*` en la unidad | Sin ellas: reintenta sin fin | Se detiene tras 5 fallos en 10 minutos | **Estabilidad** | [8](#8-unidad-de-systemd) |
| Memoria de Java | Lo que Java toma por defecto: 25 % (unos 490 MB) | `<porcentaje_heap>` % y salir ante falta de memoria | **Estabilidad** | [8](#8-unidad-de-systemd) |
| Modo del archivo de sudoers | `640` | `440` | **Orden**: hoy funciona igual | [9](#9-llave-de-deploy-y-sudoers) |
| Sitio viejo de Nginx y sitio `default` | Siguen ahí | Borrado uno, el otro sin responder | **Orden** | [10](#10-nginx-en-80) |

---

## Mapa del servidor

Todo lo que la configuración crea o toca, en tres vistas: dónde está cada cosa, cómo se usan entre sí y el detalle de cada una.

### De un vistazo

```text
/
├── opt/apps/backends/<app>/                   ① LA APLICACIÓN
│   ├── releases/                                 una carpeta por versión desplegada
│   │   └── <versión>/app.jar                     cada JAR (lo deja cada despliegue)
│   ├── current → releases/<versión>              cuál versión corre
│   └── shared/                                   reservada
│
├── etc/                                       ② LA CONFIGURACIÓN
│   ├── apps/backends/<app>/
│   │   ├── backend.env                           las variables del backend
│   │   └── db-admin.env                          la cuenta administradora de la base, aparte
│   ├── systemd/system/<app>.service              cómo arranca el backend
│   │
│   ├── nginx/                                 ③ LA PUERTA DE ENTRADA
│   │   ├── sites-available/<dominio_api>         el sitio de la API
│   │   └── sites-enabled/<dominio_api>           el enlace que lo activa
│   ├── ssl/<app>/                                el certificado de origen
│   │
│   ├── ssh/sshd_config.d/10-<app>.conf        ④ LOS ACCESOS     SSH solo con llave
│   └── sudoers.d/<usuario_deploy>                lo que deploy puede hacer como root
│
├── home/<usuario_deploy>/.ssh/authorized_keys   la llave de GitHub Actions
├── root/.ssh/authorized_keys                    la llave del administrador
│
├── var/                                       ⑤ DATOS Y LOGS
│   ├── lib/apps/backends/<app>/                  lo único que el backend puede escribir
│   └── log/nginx/<app>-*.log                     lo que pasa por el proxy
│
└── swapfile                                   memoria de reserva
```

Los logs de Java no están en ningún archivo: los guarda systemd y se leen con `journalctl -u <app>`.

### Cómo se usan entre sí

**Cuando llega una petición:**

```text
Cloudflare
   │  https://<dominio_api>
   ▼
Nginx ─────────── lee /etc/nginx/sites-enabled/<dominio_api>   (y /etc/ssl/<app>/ para HTTPS)
   │              anota en /var/log/nginx/<app>-*.log
   ▼  127.0.0.1:<puerto_backend>
Java (backend) ── systemd lo arrancó según /etc/systemd/system/<app>.service:
   │                 como <usuario_app>,
   │                 con las variables de /etc/apps/backends/<app>/backend.env,
   │                 ejecutando /opt/apps/backends/<app>/current/app.jar
   ▼
PostgreSQL administrado (fuera del Droplet)
```

**Cuando se despliega una versión:**

```text
GitHub Actions
   │  entra por SSH como <usuario_deploy>      ← permitido por /home/<usuario_deploy>/.ssh/authorized_keys
   ▼
copia el JAR a /opt/apps/backends/<app>/releases/<versión>/app.jar
   ▼
mueve current para que apunte a esa versión
   ▼
reinicia el servicio con sudo                  ← permitido por /etc/sudoers.d/<usuario_deploy>
```

### Detalle por zona

Los permisos se leen en [linux-basico.md §3](linux-basico.md#3-permisos). La última columna es el estado en este Droplet.

**① La aplicación**

| Ruta | Qué es | Dueño | Permisos | Fase | En este Droplet |
|---|---|---|---|---|---|
| `/opt/apps/backends/<app>/` | La carpeta de la aplicación | `<usuario_app>` | `2775` | 4 | Igual |
| `…/releases/` | Una carpeta por versión | `<usuario_app>` | `2775` | 4 | Igual, vacía |
| `…/releases/<versión>/app.jar` | Cada JAR; no se modifica nunca | `<usuario_deploy>`, grupo `<usuario_app>` | `644` | 13 | Falta: lo crea el despliegue |
| `…/current` | Enlace a la versión que corre | — | — | 13 | Falta: lo crea el despliegue |
| `…/shared/` | Reservada | `<usuario_app>` | `755` | 4 | Igual, vacía |

**② La configuración del backend**

| Ruta | Qué es | Dueño | Permisos | Fase | En este Droplet |
|---|---|---|---|---|---|
| `/etc/apps/backends/<app>/backend.env` | Las variables del backend, secretos incluidos | `root` | `600` | 7 | Distinto: sobra `DO_ADMIN_*` |
| `/etc/apps/backends/<app>/db-admin.env` | La cuenta administradora de la base, que systemd no carga | `root` | `600` | 7 | Falta |
| `/etc/systemd/system/<app>.service` | La unidad: usuario, variables y JAR con que arranca | `root` | `644` | 8 | Distinto: sin `StartLimit*` ni memoria |

**③ La puerta de entrada**

| Ruta | Qué es | Dueño | Permisos | Fase | En este Droplet |
|---|---|---|---|---|---|
| `/etc/nginx/sites-available/<dominio_api>` | El sitio de la API | `root` | `644` | 10–11 | Distinto: solo `:80` |
| `/etc/nginx/sites-enabled/<dominio_api>` | Enlace que activa el sitio | — | — | 10 | Igual |
| `/etc/nginx/sites-enabled/default` | El sitio de Ubuntu: responde a cualquier otro nombre | — | — | 10 | Distinto: activo y responde `200` |
| `/etc/ssl/<app>/origin.pem` | Certificado de origen | `root` | `644` | 11 | Falta |
| `/etc/ssl/<app>/origin.key` | Su clave privada | `root` | `600` | 11 | Falta |

**④ Los accesos**

| Ruta | Qué es | Dueño | Permisos | Fase | En este Droplet |
|---|---|---|---|---|---|
| `/home/<usuario_deploy>/.ssh/authorized_keys` | La llave pública de GitHub Actions | `<usuario_deploy>` | `600` | 9 | Igual |
| `/etc/sudoers.d/<usuario_deploy>` | Lo único que deploy puede hacer como root | `root` | `440` | 9 | Distinto: `640` |
| `/root/.ssh/authorized_keys` | La llave pública del administrador | `root` | `600` | 5 | Falta |
| `/etc/ssh/sshd_config.d/10-<app>.conf` | SSH sin contraseña | `root` | `644` | 5 | Falta |

**⑤ Datos, logs y sistema**

| Ruta | Qué es | Dueño | Permisos | Fase | En este Droplet |
|---|---|---|---|---|---|
| `/var/lib/apps/backends/<app>/` | La única carpeta en la que el backend escribe | `<usuario_app>` | `750` | 4 | Igual |
| `/var/log/nginx/<app>-access.log` y `-error.log` | Peticiones y errores del proxy | `www-data`, grupo `adm` | `640` | 10 | Los crea Nginx |
| `/swapfile` | Memoria de reserva | `root` | `600` | 2 | Falta |

**Verlo en el servidor:**

```bash
tree -a -pug /opt/apps                                  # la aplicación, con permisos y dueños
tree -a -pug /etc/apps /var/lib/apps                    # configuración y datos
tree /etc/nginx/sites-available /etc/nginx/sites-enabled
tree -a -pug /home/<usuario_deploy>/.ssh /etc/sudoers.d
namei -l /opt/apps/backends/<app>/current/app.jar      # permisos de cada tramo de la ruta
```

---

## 0. Entrar al Droplet

*En la laptop* (cmd o PowerShell traen `ssh`):

```bat
ssh root@<ip_del_droplet>
```

La primera vez, `ssh` muestra la huella del servidor y pregunta si se confía. Se responde `yes` solo si coincide con la que da la consola web del Droplet (DigitalOcean → el Droplet → *Access* → *Launch Droplet Console*):

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub    # anotar: es <huella_host>
```

Si algún día `ssh` avisa `REMOTE HOST IDENTIFICATION HAS CHANGED`, no se borra la línea de `known_hosts` sin antes comparar la huella otra vez.

Al entrar, para saber dónde se está:

```bash
whoami; hostname          # root, <hostname>
ip -4 -brief address      # las IPs del servidor
df -h /; free -h          # disco y memoria
uptime                    # cuánto lleva encendido
```

| En este Droplet | Estado |
|---|---|
| Se entra como root con contraseña; la huella es `SHA256:eWy2Bz…mXw` | Igual (la fase 5 cambia la contraseña por llave) |

---

## 1. Crear el Droplet

En el panel de DigitalOcean (*Create* → *Droplets*):

- **Imagen:** Ubuntu 24.04 LTS x64.
- **Tamaño:** 2 GB de RAM como mínimo para un backend Spring Boot.
- **Región:** la misma del clúster de PostgreSQL, para poder usar su red privada.
- **Autenticación:** *SSH Key* si ya se tiene la llave del administrador (fase 5.1); si se elige contraseña, la fase 5 la apaga.
- **Nombre:** `<hostname>`.

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| Imagen | Ubuntu 24.04 LTS | Ubuntu 24.04.5 LTS | Igual |
| Tamaño | 2 GB o más | 1 vCPU, 1,9 GiB, 67 GB de disco | Igual, justo |
| Autenticación inicial | Llave SSH | Contraseña de root | Distinto: se corrige en la fase 5 |

---

## 2. Paquetes, Java y swap

```bash
apt update && apt upgrade -y
apt install -y nginx curl netcat-openbsd openssl postgresql-client tree

# Java 25 de Microsoft, desde su repositorio
wget -q https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb -O /tmp/packages-microsoft-prod.deb
dpkg -i /tmp/packages-microsoft-prod.deb && rm /tmp/packages-microsoft-prod.deb
apt update
apt-cache search msopenjdk          # confirmar el nombre exacto del paquete de la versión 25
apt install -y msopenjdk-25
```

No se instalan Maven ni PostgreSQL Server: GitHub Actions compila el JAR, y la base es administrada. `postgresql-client` solo sirve para diagnosticar.

**Swap.** Espacio del disco que Linux usa como memoria de reserva cuando la RAM se llena ([linux-basico.md §5](linux-basico.md#5-memoria-y-swap)). Sin él, en un pico de memoria el kernel mata a Java.

```bash
fallocate -l <tamaño_swap> /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab     # para que vuelva al reiniciar
```

**Comprobar:**

```bash
grep PRETTY_NAME /etc/os-release                    # Ubuntu 24.04.x LTS
/usr/bin/java -version                              # openjdk 25 … Microsoft
timedatectl show -p Timezone -p NTPSynchronized     # Etc/UTC, yes
free -h                                             # la fila Swap con <tamaño_swap>
```

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| Java | OpenJDK 25 en `/usr/bin/java` | Microsoft OpenJDK 25.0.4.1 | Igual |
| Herramientas | `nginx`, `curl`, `nc`, `openssl`, `psql`, `tree` | Todas | Igual |
| Hora | UTC, sincronizada | UTC, sincronizada | Igual |
| Swap | `<tamaño_swap>` | Ninguno | **Falta** |

---

## 3. Usuarios

```bash
# Ejecuta Java. De sistema: sin contraseña ni shell, nadie entra con él.
useradd --system --home-dir /opt/apps/backends/<app> --shell /usr/sbin/nologin <usuario_app>

# Recibe las versiones desde GitHub Actions. Sin contraseña: entra solo con llave.
adduser --disabled-password --gecos "" <usuario_deploy>
usermod -aG <usuario_app> <usuario_deploy>          # lo mete en el grupo de la app
```

**Comprobar:**

```bash
id <usuario_app>                                    # uid y gid de sistema (menores que 1000)
id <usuario_deploy>                                 # en groups aparece <usuario_app>
getent passwd <usuario_app> | cut -d: -f7           # /usr/sbin/nologin
```

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| Usuario de la app | De sistema, `nologin` | `app-millennium-erp`, UID 999, `nologin` | Igual |
| Usuario de deploy | Con carpeta personal, sin contraseña, en el grupo de la app | `deploy-millennium-erp`, UID 1000, en `app-millennium-erp` | Igual |

---

## 4. Carpetas

Tres zonas, cada una en la carpeta que Linux reserva para eso ([linux-basico.md §1](linux-basico.md#1-las-carpetas-de-un-servidor-linux)):

| Carpeta | Para qué | Dueño | Permisos |
|---|---|---|---|
| `/opt/apps/backends/<app>/` | El programa: las versiones y el enlace `current` | `<usuario_app>` | `2775` |
| `/var/lib/apps/backends/<app>/` | Lo que el servicio escribe; la única carpeta escribible para systemd | `<usuario_app>` | `750` |
| `/etc/apps/backends/<app>/` | La configuración: `backend.env` | `root` | `755` |

**4.1 Crear.** `mkdir -p` crea también las intermedias y no falla si ya existen; esas quedan de root con `755`:

```bash
mkdir -p /opt/apps/backends/<app>/releases
mkdir -p /opt/apps/backends/<app>/shared
mkdir -p /var/lib/apps/backends/<app>
mkdir -p /etc/apps/backends/<app>
```

**4.2 Darle las carpetas de la app a su usuario.** La de `/etc` se queda de root: la aplicación no debe poder cambiar su propia configuración.

```bash
chown <usuario_app>:<usuario_app> \
  /opt/apps/backends/<app> \
  /opt/apps/backends/<app>/releases \
  /opt/apps/backends/<app>/shared \
  /var/lib/apps/backends/<app>
```

**4.3 Permisos:**

```bash
chmod 2775 /opt/apps/backends/<app> /opt/apps/backends/<app>/releases
chmod 755  /opt/apps/backends/<app>/shared
chmod 750  /var/lib/apps/backends/<app>
```

- `2775`: dueño y grupo leen, escriben y entran; los demás leen y entran. El `2` (setgid) hace que lo que deploy cree dentro quede del grupo de la app, así la app puede leerlo.
- `750`: la app escribe, su grupo lee, nadie más entra.

`current` no se crea a mano: lo crea el primer despliegue.

**Comprobar:**

```bash
tree -a -pug /opt/apps
sudo -u <usuario_deploy> test -w /opt/apps/backends/<app>/releases && echo "deploy escribe: OK"
sudo -u <usuario_deploy> test -w /opt/apps/backends/<app> && echo "deploy mueve current: OK"
sudo -u <usuario_app> test -x /opt/apps/backends/<app>/releases && echo "app atraviesa: OK"
```

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| `/opt/apps/backends/<app>` y `releases` | `<usuario_app>`, `2775` | Así | Igual |
| `shared` | `<usuario_app>`, `755` | Así | Igual |
| `/var/lib/apps/backends/<app>` | `<usuario_app>`, `750` | Así | Igual |
| `/etc/apps/backends/<app>` | `root`, `755` | Así | Igual |
| Pruebas de acceso | Las tres OK | Las tres OK | Igual |

---

## 5. Acceso del administrador y firewall

Se hace en este orden, **sin cerrar la sesión abierta** hasta comprobar la nueva. Si algo sale mal, la consola web del Droplet en DigitalOcean sigue entrando.

**5.1 Una llave para el administrador** (*en la laptop*, cmd o PowerShell):

```bat
ssh-keygen -t ed25519 -C "<nombre_llave_admin>"
type %USERPROFILE%\.ssh\id_ed25519.pub | ssh root@<ip_del_droplet> "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

`ssh-keygen` pide una frase de paso: conviene ponerla. El segundo comando pide la contraseña de root por última vez.

**5.2 Probarla desde otra ventana** (*en la laptop*). Si no responde, no se sigue:

```bat
ssh -o PasswordAuthentication=no root@<ip_del_droplet> "echo entra con llave"
```

**5.3 Apagar la contraseña:**

```bash
cat > /etc/ssh/sshd_config.d/10-<app>.conf <<'EOF'
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitRootLogin prohibit-password
EOF
sshd -t && systemctl reload ssh
sshd -T | grep -Ei '^(passwordauthentication|permitrootlogin) '    # no, prohibit-password
```

El archivo empieza con `10-` a propósito: sshd se queda con el **primer** valor que lee, y lee `sshd_config.d/` en orden alfabético. En un Droplet creado con contraseña, cloud-init suele dejar un `50-cloud-init.conf` con `PasswordAuthentication yes` (`ls /etc/ssh/sshd_config.d/`); con el `10-` delante queda sin efecto. Por eso se comprueba con `sshd -T`, que muestra lo que sshd usa de verdad. Después se repite 5.2 desde otra ventana.

**5.4 Firewall.** El 22 se permite **antes** de activar, para no cortar la sesión:

```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow from <tu_ip_de_oficina> to any port 22 proto tcp     # sin IP fija: ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
ufw status verbose
```

El puerto del backend no se abre: Java escucha solo en `127.0.0.1`. Con HTTPS funcionando, `80` y `443` se pueden limitar a los rangos de Cloudflare. Un *Cloud Firewall* de DigitalOcean logra lo mismo desde el panel.

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| Llave del administrador | En `/root/.ssh/authorized_keys` | No hay | **Falta** |
| `PasswordAuthentication` | `no` | `yes` | **Distinto**: cualquiera puede probar contraseñas |
| `PermitRootLogin` | `prohibit-password` | `yes` | **Distinto** |
| Firewall | Solo 22, 80 y 443 | `ufw` inactivo | **Falta** |
| Puertos que escuchan | 22, 80, 443 y el backend solo en `127.0.0.1` | 22 y 80 | Igual por ahora (falta el 443) |

---

## 6. Base de datos administrada

**6.1 Permitir al Droplet** (DigitalOcean → *Databases* → el clúster → *Settings* → *Trusted sources* → agregar el Droplet).

**6.2 Cuentas y base.** Una sola vez por clúster, con la cuenta administradora `doadmin` y los scripts del backend `create-roles.sql` y `prepare-managed-database.sql` (su README, "Despliegue en DigitalOcean"). Si la base ya la usa otro despliegue, este paso está hecho.

**6.3 Comprobar desde el Droplet:**

```bash
nc -vz -w 10 <host_bd> <puerto_bd>                                       # succeeded
psql "host=<host_bd> port=<puerto_bd> dbname=<nombre_bd> user=<cuenta_bd_app> sslmode=require" -c 'select current_user'
psql "host=<host_bd> port=<puerto_bd> dbname=<nombre_bd> user=<cuenta_bd_migraciones> sslmode=require" -c 'select current_user'
```

`psql` pide cada clave sin mostrarla; cada una tiene que responder su propio nombre. Para no escribir el host, se lee de `backend.env`: `H=$(sed -n 's/^DATABASE_HOST=//p' /etc/apps/backends/<app>/backend.env)` y se usa `host=$H`.

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| *Trusted sources* | El Droplet agregado | El puerto `25060` responde | Igual |
| TLS | Cifrado | TLS 1.3 con la CA del proyecto | Igual |
| Cuenta de la aplicación | Entra con `psql` | No probado | **Falta comprobar** |
| Cuenta de migraciones | Entra con `psql` | No probado | **Falta comprobar** |
| Red | Host privado de la VPC si están en la misma región | Host público, cifrado | Distinto, aceptable |

---

## 7. `backend.env`

```bash
install -m 600 -o root -g root /dev/null /etc/apps/backends/<app>/backend.env
nano /etc/apps/backends/<app>/backend.env
```

Contenido. Las claves son las de `.env.example` del backend; lo que va entre `< >` se escribe dentro de nano:

```bash
SPRING_PROFILES_ACTIVE=<perfil_spring>
PORT=<puerto_backend>
SERVER_ADDRESS=127.0.0.1
DATABASE_HOST=<host_bd>
DATABASE_PORT=<puerto_bd>
DATABASE_NAME=<nombre_bd>
DATABASE_SSL_MODE=require
DATABASE_USER=<cuenta_bd_app>
DATABASE_PASSWORD=<clave_de_cuenta_bd_app>
FLYWAY_USER=<cuenta_bd_migraciones>
FLYWAY_PASSWORD=<clave_de_cuenta_bd_migraciones>
IAM_BOOTSTRAP_ADMIN_EMAIL=<correo_del_primer_superadmin>
IAM_BOOTSTRAP_ADMIN_PASSWORD=<clave_provisional_del_primer_superadmin>
IAM_PUBLIC_SIGN_UP_ENABLED=false
```

Se guarda con `Ctrl+O`, `Enter`, y se sale con `Ctrl+X`. Si se pegó desde Windows, se quitan los `\r`: `sed -i 's/\r$//' /etc/apps/backends/<app>/backend.env`.

La cuenta administradora de la base (`DO_ADMIN_USER`, `DO_ADMIN_PASSWORD` de `.env.example`) **no va aquí**: la aplicación no la lee, y systemd le pasa este archivo entero al proceso Java. Si se quiere en el Droplet, va aparte:

```bash
install -m 600 -o root -g root /dev/null /etc/apps/backends/<app>/db-admin.env
nano /etc/apps/backends/<app>/db-admin.env      # DO_ADMIN_USER=doadmin y DO_ADMIN_PASSWORD=<clave_de_doadmin>
```

**Comprobar** (sin mostrar secretos):

```bash
stat -c '%a %U:%G' /etc/apps/backends/<app>/backend.env               # 600 root:root
grep -E '^(SPRING_PROFILES_ACTIVE|PORT|SERVER_ADDRESS)=' /etc/apps/backends/<app>/backend.env
grep -oE '^[A-Z_]+=' /etc/apps/backends/<app>/backend.env             # solo los nombres
grep -c $'\r' /etc/apps/backends/<app>/backend.env                    # 0
```

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| Archivo | `root:root`, `600`, sin `\r` ni claves repetidas | Así | Igual |
| Perfil, puerto y dirección | `<perfil_spring>`, `<puerto_backend>`, `127.0.0.1` | `prod`, `8080`, `127.0.0.1` | Igual |
| Base y cuentas | Las de los marcadores, cuentas distintas | Así | Igual |
| Registro público | `false` | `false` | Igual |
| `DO_ADMIN_*` | En `db-admin.env` | En `backend.env`, con clave | **Distinto**: llega al proceso Java |

**Arreglarlo en este Droplet** (mueve las dos líneas sin mostrarlas):

```bash
install -m 600 -o root -g root /dev/null /etc/apps/backends/millennium-erp/db-admin.env
grep -E '^DO_ADMIN_' /etc/apps/backends/millennium-erp/backend.env > /etc/apps/backends/millennium-erp/db-admin.env
sed -i '/^DO_ADMIN_/d' /etc/apps/backends/millennium-erp/backend.env
```

---

## 8. Unidad de systemd

```bash
nano /etc/systemd/system/<app>.service
```

```ini
[Unit]
Description=<descripción del servicio>
Wants=network-online.target
After=network-online.target
# Si arranca y cae 5 veces en 10 minutos, queda en failed en vez de reintentar sin fin.
StartLimitIntervalSec=600
StartLimitBurst=5

[Service]
Type=simple
User=<usuario_app>
Group=<usuario_app>
WorkingDirectory=/opt/apps/backends/<app>/current
EnvironmentFile=/etc/apps/backends/<app>/backend.env
# Java toma <porcentaje_heap> % de la RAM y, si se le acaba, termina para que systemd lo reinicie.
ExecStart=/usr/bin/java -XX:MaxRAMPercentage=<porcentaje_heap> -XX:+ExitOnOutOfMemoryError -jar /opt/apps/backends/<app>/current/app.jar
Restart=on-failure
RestartSec=5
SuccessExitStatus=143
TimeoutStopSec=60
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/apps/backends/<app>
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemd-analyze verify /etc/systemd/system/<app>.service    # sin salida = sin avisos
```

**No** se corre `enable` ni `start` todavía: sin `current/app.jar` la unidad falla con `200/CHDIR`. El primer arranque lo hace el despliegue (fase 13).

Qué hace cada línea, en [droplet-digitalocean.md §3.4](droplet-digitalocean.md#34-systemd). Las dos que más importan elegir:

- **`StartLimitIntervalSec` y `StartLimitBurst`.** `Restart=on-failure` reintenta cada 5 s; sin un límite, un backend que no puede arrancar (una migración detenida, una clave equivocada) lo intenta para siempre, y cada intento vuelve a correr Flyway contra la base.
- **`-XX:MaxRAMPercentage`.** Sin él, Java usa como máximo el 25 % de la RAM para sus objetos. Con 2 GB son unos 490 MB, poco margen para exportaciones y picos. Con `50` llega a unos 950 MB y deja el resto para lo que Java usa fuera de ese espacio, Nginx y el sistema. `-XX:+ExitOnOutOfMemoryError` evita que quede vivo pero roto si se queda sin memoria.

**Comprobar:**

```bash
systemctl cat <app>                                          # la unidad tal como la tiene cargada
systemctl is-enabled <app>; systemctl is-active <app>        # disabled, inactive (antes del despliegue)
```

| Línea | Ideal | En este Droplet | Estado |
|---|---|---|---|
| Usuario, carpeta, variables, JAR | Los de los marcadores | Los de `millennium-erp` | Igual |
| Reinicio y parada (`Restart`, `RestartSec`, `SuccessExitStatus`, `TimeoutStopSec`) | Como arriba | Así | Igual |
| Aislamiento (`NoNewPrivileges` … `ReadWritePaths`) | Como arriba | Así | Igual |
| `StartLimitIntervalSec`, `StartLimitBurst` | `600`, `5` | No están | **Falta**: reintentaría sin fin |
| `-XX:MaxRAMPercentage` | `<porcentaje_heap>` (`50`) | No está: 25 % por defecto | **Falta**: poco margen de memoria |
| `-XX:+ExitOnOutOfMemoryError` | Presente | No está | **Falta** |
| Estado | `disabled`, `inactive` hasta el despliegue | Así | Igual |

---

## 9. Llave de deploy y sudoers

**9.1 La llave.** La crea root en el Droplet; la privada sale una sola vez, hacia el secreto de GitHub (fase 12):

```bash
install -d -m 700 /root/github-deploy-setup
ssh-keygen -t ed25519 -N '' -C <nombre_llave_deploy> -f /root/github-deploy-setup/id_ed25519
install -d -m 700 -o <usuario_deploy> -g <usuario_deploy> /home/<usuario_deploy>/.ssh
install -m 600 -o <usuario_deploy> -g <usuario_deploy> \
  /root/github-deploy-setup/id_ed25519.pub /home/<usuario_deploy>/.ssh/authorized_keys
```

**9.2 Probarla en local:**

```bash
ssh-keyscan -t ed25519 127.0.0.1 > /root/github-deploy-setup/known_hosts_local
ssh -i /root/github-deploy-setup/id_ed25519 -o IdentitiesOnly=yes -o BatchMode=yes \
    -o StrictHostKeyChecking=yes -o UserKnownHostsFile=/root/github-deploy-setup/known_hosts_local \
    <usuario_deploy>@127.0.0.1 whoami                      # <usuario_deploy>
```

Esta prueba confirma la llave y el usuario, no el acceso desde GitHub: ese se prueba en la fase 12.

**9.3 Sudoers.** Se edita con `visudo -f`, que revisa la sintaxis y deja el modo `440`:

```bash
visudo -f /etc/sudoers.d/<usuario_deploy>
```

```text
<usuario_deploy> ALL=(root) NOPASSWD: /usr/bin/systemctl restart <app>.service, /usr/bin/systemctl is-active --quiet <app>.service
```

sudo compara el comando letra por letra: el despliegue tiene que llamar exactamente esas dos líneas.

**Comprobar:**

```bash
visudo -c                              # todo "parsed OK", sin "bad permissions"
sudo -l -U <usuario_deploy>            # las dos reglas, NOPASSWD
```

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| Llave de deploy | Una, ED25519, en `authorized_keys` (`700`/`600`) | `github-actions-millennium-dev` | Igual |
| Prueba local | `whoami` responde el usuario | Respondió | Igual |
| Reglas de sudoers | Las dos, exactas | Las dos, en vigor | Igual |
| Modo del archivo de sudoers | `440` | `640`; `visudo -c` avisa | **Distinto**: `chmod 0440 /etc/sudoers.d/deploy-millennium-erp` |

---

## 10. Nginx en `:80`

Un sitio provisional, solo HTTP, para probar el proxy antes de tener certificados. La fase 11 lo reemplaza.

```bash
nano /etc/nginx/sites-available/<dominio_api>
```

```nginx
server {
    listen 80;
    server_name <dominio_api>;

    access_log /var/log/nginx/<app>-access.log;
    error_log  /var/log/nginx/<app>-error.log warn;

    client_max_body_size 20M;

    location / {
        proxy_pass http://127.0.0.1:<puerto_backend>;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 10s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

```bash
ln -s /etc/nginx/sites-available/<dominio_api> /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

**El sitio `default` de Ubuntu** responde `200` a cualquier petición por IP o con otro nombre. En el ideal no contesta nada que no sea la API:

```bash
cp /etc/nginx/sites-available/default /root/nginx-default.bak-$(date +%F)    # copia del original
cat > /etc/nginx/sites-available/default <<'EOF'
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;
    return 444;
}
EOF
nginx -t && systemctl reload nginx
```

`444` es un código propio de Nginx: cierra la conexión sin responder.

**Comprobar:**

```bash
curl -s -o /dev/null -w '%{http_code}\n' -H 'Host: <dominio_api>' http://127.0.0.1/   # 502 sin backend
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1/                           # 000: el default no responde
```

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| Sitio de la API en `:80` | Como arriba | Igual, con logs `millennium-erp-leasing-dev-*.log` | Igual |
| Sitio activo | Enlace en `sites-enabled` | Así; con su `Host` responde `502` | Igual |
| Sitio `default` | `return 444` | Activo, responde `200` | **Distinto**: menor |
| Sitios viejos | Ninguno | `sites-available/millennium-erp-api` (apuntaba al `8081`) | **Distinto**: `rm /etc/nginx/sites-available/millennium-erp-api` |

---

## 11. HTTPS con Cloudflare

**11.1 DNS** (Cloudflare → *DNS*): registro `A`, nombre `<dominio_api>` sin el dominio raíz, contenido `<ip_del_droplet>`, proxy activado (nube naranja).

**11.2 Certificado de borde** (Cloudflare → *SSL/TLS* → *Edge Certificates*). El Universal SSL cubre el dominio raíz y **un** nivel de subdominio (`*.dominio`). Si `<dominio_api>` tiene dos niveles o más (`api.leasing.dev.dominio`), hace falta *Order Advanced Certificate* con ese nombre exacto (requiere Advanced Certificate Manager). Se espera a que figure *Active*.

**11.3 Certificado de origen** (Cloudflare → *SSL/TLS* → *Origin Server* → *Create Certificate*, con el hostname `<dominio_api>`, formato PEM). Cloudflare muestra el certificado y su clave **una sola vez**:

```bash
install -d -m 755 /etc/ssl/<app>
nano /etc/ssl/<app>/origin.pem     # pegar el certificado
nano /etc/ssl/<app>/origin.key     # pegar la clave privada
chmod 644 /etc/ssl/<app>/origin.pem
chmod 600 /etc/ssl/<app>/origin.key
```

**11.4 Nginx con HTTPS.** Reemplaza el sitio de la fase 10:

```bash
nano /etc/nginx/sites-available/<dominio_api>
```

```nginx
server {
    listen 80;
    server_name <dominio_api>;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    http2 on;
    server_name <dominio_api>;

    ssl_certificate     /etc/ssl/<app>/origin.pem;
    ssl_certificate_key /etc/ssl/<app>/origin.key;

    access_log /var/log/nginx/<app>-access.log;
    error_log  /var/log/nginx/<app>-error.log warn;

    client_max_body_size 20M;

    location / {
        proxy_pass http://127.0.0.1:<puerto_backend>;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 10s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

```bash
nginx -t && systemctl reload nginx
ss -ltnp | grep -E ':80 |:443 '
```

**11.5 Modo Full (strict)** (Cloudflare → *SSL/TLS* → *Overview*). El modo vale para toda la zona: antes de cambiarlo se revisa que los demás subdominios con proxy tengan un certificado válido en su origen; si alguno no, se fija solo para `<dominio_api>` con una *Configuration Rule*. Nunca *Flexible*.

**Comprobar** (*en la laptop*):

```bat
curl -sI https://<dominio_api>/actuator/health
```

Sin backend da `502` **con el TLS correcto**; con backend, `200`.

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| DNS | `A` con proxy | Así | Igual |
| Certificado de borde | Activo para `<dominio_api>` | No hay: el saludo TLS falla | **Falta** |
| Certificado de origen | En `/etc/ssl/<app>/` | No hay | **Falta** |
| Nginx en `:443` | Como arriba | Solo `:80` | **Falta** |
| Modo de Cloudflare | Full (strict) | Sin comprobar | **Falta comprobar** |

---

## 12. GitHub

**12.1 Environment y secretos** (`<repo_backend>` → *Settings* → *Environments* → `<environment_github>`, regla de despliegue: `<rama>`):

| Secreto | Valor |
|---|---|
| `DROPLET_HOST` | `<ip_del_droplet>` |
| `DROPLET_USER` | `<usuario_deploy>` |
| `DROPLET_SSH_KEY` | La privada de 9.1, completa: `cat /root/github-deploy-setup/id_ed25519`, de `-----BEGIN` a `-----END` |
| `DROPLET_KNOWN_HOSTS` | La línea de abajo |

La línea de `DROPLET_KNOWN_HOSTS` se genera **desde fuera** del Droplet y se acepta solo si su huella es `<huella_host>` (*en la laptop*):

```bat
ssh-keyscan -t ed25519 <ip_del_droplet> > droplet_known_hosts.txt
ssh-keygen -lf droplet_known_hosts.txt
```

Se pega una sola línea, que empieza con `<ip_del_droplet> ssh-ed25519 AAAA`.

**12.2 `deploy-dev.yml`.** Workflow manual en `<repo_backend>`, con lo que pide [droplet-digitalocean.md §4](droplet-digitalocean.md#4-lo-que-tiene-que-hacer-deploy-devyml-propuesto). Su primera corrida solo prueba la conexión (`whoami && hostname`).

**12.3 Limpieza.** Cuando Actions ya entró bien, se borra la copia de la llave privada:

```bash
shred -u /root/github-deploy-setup/id_ed25519 && rm -rf /root/github-deploy-setup
```

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| Environment y regla de rama | `<environment_github>`, `<rama>` | `development`, `develop` | Igual |
| `DROPLET_HOST`, `DROPLET_USER`, `DROPLET_SSH_KEY` | Cargados | Cargados; sin probar desde Actions | Igual, sin probar |
| `DROPLET_KNOWN_HOSTS` | La línea de `<ip_del_droplet>` comprobada | En el Droplet solo hay una de `127.0.0.1` y otra vacía | **Probablemente distinto**: volver a cargarlo |
| `deploy-dev.yml` | En `.github/workflows/` | No existe | **Falta** |
| `/root/github-deploy-setup` | Borrada tras probar Actions | Existe, con la llave privada | **Falta** limpiar (después de 12.2) |

---

## 13. Primer despliegue

Antes:

- Un respaldo restaurable de la base.
- `ci.yml` verde para el commit que se va a desplegar.
- Si otro despliegue usa la misma base, decidir qué pasa con él ([droplet-digitalocean.md §9](droplet-digitalocean.md#9-decisiones-pendientes)).

Pasos:

1. Correr `deploy-dev.yml` desde *Actions* → *Run workflow* → `<rama>`.
2. En el Droplet:

   ```bash
   readlink -f /opt/apps/backends/<app>/current
   systemctl status <app> --no-pager -l
   journalctl -u <app> -n 100 --no-pager
   curl -i --max-time 8 http://127.0.0.1:<puerto_backend>/actuator/health      # 200 {"status":"UP"}
   ```

3. Que arranque con el servidor: `systemctl enable <app>`.
4. Cloudflare → *Workers & Pages* → el proyecto del frontend → *Settings* → *Variables and Secrets* → `BACKEND_ORIGIN` = `https://<dominio_api>`, sin barra final, y volver a desplegar el frontend.
5. Entrar en `https://<dominio_frontend>` e iniciar sesión.
6. Anotar en [droplet-digitalocean.md](droplet-digitalocean.md) la fecha, el commit y lo comprobado.

| Punto | Ideal | En este Droplet | Estado |
|---|---|---|---|
| Versión desplegada y `current` | Existen | No | **Falta** |
| Servicio | `enabled`, `active`, health `UP` | `disabled`, `inactive` | **Falta** |
| `BACKEND_ORIGIN` | `https://<dominio_api>` | Sin cambiar (no se ha revisado a qué apunta hoy) | **Falta** |
