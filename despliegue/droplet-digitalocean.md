# Despliegue del backend en un Droplet de DigitalOcean

**Actualizado:** 20/10/2026 · **Evidencias al:** 20/10/2026 (auditoría de solo lectura en el Droplet) · **Estado:** servidor preparado, sin ningún despliegue · **Fuente:** el manual técnico v5.0 del 18/10/2026 (capturas y salidas de terminal del operador) contrastado con el código de `develop` del backend (`6d73031`). Si este documento y el código no coinciden, manda el código; si el README del backend y este documento no coinciden en cómo se despliega, ver la [§1](#1-qué-se-está-montando-y-qué-ya-existe).

Este documento junta tres cosas: cómo quedó configurado el Droplet, lo que hay que saber para configurarlo (o repetirlo en otro servidor) y las lecciones de la configuración. Los comandos en orden, fase por fase y con su comprobación, están en [droplet-paso-a-paso.md](droplet-paso-a-paso.md). No dice que algo funcione si no se vio funcionar: cada estado lleva su evidencia.

- **Verificado**: se vio en una captura o en la salida de un comando.
- **Informado**: lo dijo el operador, sin una salida adjunta.
- **Pendiente**: no se hizo, o se hizo sin poder comprobarlo.
- **Propuesto**: referencia para construir, nunca ejecutada.

Nunca se escriben aquí secretos, claves privadas, contraseñas, tokens ni el contenido de `backend.env`. Tampoco el hostname real de la base: se escribe `<DATABASE_HOST>`.

---

## 1. Qué se está montando y qué ya existe

```text
Navegador ── https://dev.erp.millennium.ky ──► Cloudflare Pages (frontend)
                                                 │  la Pages Function reenvía /api/ a BACKEND_ORIGIN
                                                 ▼
                         https://api.leasing.dev.millennium.ky (Cloudflare, Full strict)
                                                 ▼
Droplet Ubuntu 24.04 ── Nginx :443 ──► 127.0.0.1:8080 Spring Boot (systemd, app-millennium-erp)
                                                 ▼
                         PostgreSQL 18 administrado de DigitalOcean (puerto 25060, TLS)

GitHub develop ── Actions: ci.yml (pruebas) ── deploy-dev.yml (por crear) ── SCP + SSH ──► Droplet
```

Tres hechos del código que fijan esta forma:

1. **El navegador nunca llama al Droplet.** La cookie de sesión es `HttpOnly`, `Secure` y `SameSite=Strict`, y el frontend tiene que estar en el mismo origen que la API (`application.yaml`, `WebSecurityConfiguration`). Por eso el frontend se publica en Cloudflare Pages y su *Pages Function* reenvía `/api/` al backend (README del frontend, "Despliegue en Cloudflare Pages"). `api.leasing.dev.millennium.ky` es el `BACKEND_ORIGIN` de esa función, no una URL para el navegador, y no hace falta CORS.
2. **El repositorio ya tiene otro despliegue.** `.do/app.yaml` y el `Dockerfile` publican `develop` en DigitalOcean App Platform en cada push (README del backend, "Despliegue en DigitalOcean"), y `orden-de-implementacion` registra despliegues de `develop` sin errores el 18/10/2026. El Droplet es un segundo camino, no el primero.
3. **Los dos apuntan a la misma base** si `DATABASE_NAME` es la de App Platform. Antes del primer arranque en el Droplet hay que decidir qué pasa con App Platform ([§9](#9-decisiones-pendientes)).

---

## 2. Estado al 20/10/2026

Auditoría de solo lectura corrida en el Droplet el 20/10/2026 a las 04:00 UTC, más pruebas de DNS y HTTPS desde fuera.

| Pieza                                                                                                                                                 | Evidencia                                                                                                                                                                                                       | Estado                                                                                              |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Ubuntu 24.04.5 LTS, Microsoft OpenJDK 25.0.4.1 en `/usr/bin/java`; 1 vCPU, 1,9 GiB de RAM, **sin swap**, 67 GB de disco; UTC con NTP sincronizado     | Auditoría                                                                                                                                                                                                       | Verificado                                                                                          |
| Usuarios `app-millennium-erp` (UID 999, shell `nologin`) y `deploy-millennium-erp` (UID 1000, en el grupo `app-millennium-erp`)                       | Auditoría                                                                                                                                                                                                       | Verificado                                                                                          |
| Raíz y `releases` en `2775` de `app-millennium-erp`; deploy escribe en las dos (umask `0002`), la app las atraviesa y escribe en `/var/lib/…` (`750`) | Auditoría, probado como cada usuario                                                                                                                                                                            | Verificado                                                                                          |
| `current` y `current/app.jar`                                                                                                                         | `releases` vacío                                                                                                                                                                                                | No existen                                                                                          |
| `millennium-erp.service`                                                                                                                              | Igual a la [§3.4](#34-systemd); `systemd-analyze verify` sin avisos; `disabled`, `inactive`, sin entradas en el journal                                                                                         | Verificado                                                                                          |
| `backend.env`                                                                                                                                         | `root:root` `600`, sin CRLF ni claves repetidas; `prod`, `PORT=8080`, `SERVER_ADDRESS=127.0.0.1`, `require`, `erp_mm_app` y `erp_mm_owner`, base `millennium_erp_dev`, registro público apagado                 | Verificado; **sobran `DO_ADMIN_USER` y `DO_ADMIN_PASSWORD`** (la clave ya tiene valor)              |
| Sudoers del usuario deploy                                                                                                                            | Las dos reglas exactas, en vigor (`sudo -l`)                                                                                                                                                                    | Verificado; **el archivo tiene modo `640` y `visudo -c` pide `440`**                                |
| SSH del servidor                                                                                                                                      | Puerto 22, `PermitRootLogin yes`, `PasswordAuthentication yes`                                                                                                                                                  | **Root entra con contraseña desde Internet**                                                        |
| Llave de host ED25519                                                                                                                                 | `SHA256:eWy2BzWHuFSaHGFO8ABXcX/0qIp0YJ1N1pKek1vCmXw`, la misma dentro del Droplet y por `ssh-keyscan` desde fuera                                                                                               | Verificado                                                                                          |
| Llaves autorizadas para deploy                                                                                                                        | Una sola: `github-actions-millennium-dev` (ED25519), `.ssh` `700`, `authorized_keys` `600`                                                                                                                      | Verificado                                                                                          |
| `/root/github-deploy-setup`                                                                                                                           | Conserva la llave privada; `known_hosts_test` (la línea de `127.0.0.1`) y `known_hosts_public` **vacío**                                                                                                        | Pendiente de limpiar; ver la [§3.3](#33-ssh)                                                        |
| Nginx                                                                                                                                                 | `nginx -t` correcto; sitio `api.leasing.dev.millennium.ky` (`:80` → `127.0.0.1:8080`, con las cabeceras `X-Forwarded-*`) y `default` habilitados; `millennium-erp-api` sigue en `sites-available` sin habilitar | Verificado: con ese `Host` responde `502` (no hay backend); sin `Host`, el `default` responde `200` |
| Nginx en `:443` y certificados de origen                                                                                                              | No escucha `:443`; no hay `/etc/ssl/millennium-erp` ni `/etc/letsencrypt`                                                                                                                                       | Pendiente                                                                                           |
| Firewall                                                                                                                                              | `ufw` inactivo; escuchan `:22` y `:80` en todas las interfaces. El firewall de nube de DigitalOcean no se ve desde dentro                                                                                       | **Sin firewall en el servidor**                                                                     |
| DNS en Cloudflare                                                                                                                                     | Resuelve a IPs de Cloudflare: el proxy está activo                                                                                                                                                              | Verificado                                                                                          |
| HTTPS público                                                                                                                                         | Falla el saludo TLS en el borde de Cloudflare, desde la laptop y desde el Droplet; `http://` responde `502` de Cloudflare                                                                                       | **Sin certificado de borde para el nombre de dos niveles**                                          |
| PostgreSQL administrado                                                                                                                               | El host resuelve a una IP pública; TCP a `25060` abierto (el clúster acepta al Droplet); TLS 1.3 con certificado de la CA propia del proyecto (`Verify return code: 19`, esperado sin esa CA)                   | Verificado                                                                                          |
| PostgreSQL: autenticación de las dos cuentas y permisos de Flyway                                                                                     | —                                                                                                                                                                                                               | Informado, sin salida adjunta                                                                       |
| GitHub: environment `development`, limitado a `develop`, con `DROPLET_HOST`, `DROPLET_USER`, `DROPLET_SSH_KEY` y `DROPLET_KNOWN_HOSTS`                | Capturas                                                                                                                                                                                                        | Registrados; los valores no se pueden ver                                                           |
| `ci.yml`                                                                                                                                              | Tres corridas verdes en `develop`                                                                                                                                                                               | Verificado; no despliega                                                                            |
| `deploy-dev.yml`                                                                                                                                      | Hay un borrador fuera del repositorio                                                                                                                                                                           | Pendiente: no existe en `.github/workflows/`                                                        |

---

## 3. Configuración del servidor, pieza por pieza

### 3.1 Rutas

| Ruta | Para qué |
|---|---|
| `/opt/apps/backends/millennium-erp/releases/<id>/app.jar` | Cada versión, inmutable; `<id>` es `<sha>-<run_id>` |
| `/opt/apps/backends/millennium-erp/current` | Enlace simbólico a la versión activa; lo crea solo el despliegue |
| `/opt/apps/backends/millennium-erp/shared/` | Recursos compartidos no sensibles, si algún día hacen falta |
| `/etc/apps/backends/millennium-erp/backend.env` | Variables y secretos de ejecución |
| `/var/lib/apps/backends/millennium-erp/` | Única carpeta en la que el servicio puede escribir |
| `/etc/systemd/system/millennium-erp.service` | La unidad |
| `/etc/nginx/sites-available/` y `sites-enabled/` | El sitio de Nginx y su enlace |
| `/home/deploy-millennium-erp/.ssh/authorized_keys` | Las llaves que pueden entrar como deploy |

### 3.2 Usuarios y sudoers

`app-millennium-erp` ejecuta Java y no tiene llaves. `deploy-millennium-erp` copia versiones, mueve `current` y solo puede reiniciar y consultar el servicio:

```text
deploy-millennium-erp ALL=(root) NOPASSWD: /usr/bin/systemctl restart millennium-erp.service, /usr/bin/systemctl is-active --quiet millennium-erp.service
```

Sudoers compara el comando entero: `systemctl restart millennium-erp` (sin `.service`) o `systemctl status …` piden contraseña y fallan en un job sin terminal. El despliegue tiene que llamar exactamente esas dos líneas. `enable` no está permitido: lo hace un administrador una vez, después del primer despliegue sano ([§6](#6-primer-despliegue-y-los-siguientes)).

El archivo de `/etc/sudoers.d/` va en modo `440` y de `root:root`; con otro modo `visudo -c` avisa (`bad permissions, should be mode 0440`) y versiones más estrictas de sudo lo ignoran.

```bash
sudo chmod 0440 /etc/sudoers.d/deploy-millennium-erp
sudo -l -U deploy-millennium-erp
sudo visudo -c
sudo -u deploy-millennium-erp test -w /opt/apps/backends/millennium-erp && echo OK
```

### 3.3 SSH

Hay dos pares de llaves y no se mezclan:

- **La llave de deploy** (ED25519) autentica al runner ante el Droplet. Su pública está en `authorized_keys` del usuario deploy (`.ssh` en `700`, `authorized_keys` en `600`); la privada vive solo en el secreto `DROPLET_SSH_KEY`.
- **La llave de host** autentica al Droplet ante el runner. Va en `DROPLET_KNOWN_HOSTS`, y su línea tiene que nombrar exactamente lo mismo que `DROPLET_HOST` (la IP pública, sin `http://`), y el puerto si no es el 22.

Se prueba así, sin desactivar nunca `StrictHostKeyChecking`:

```bash
sudo ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub     # la huella real del Droplet
ssh -i <llave> -o IdentitiesOnly=yes -o BatchMode=yes \
    -o StrictHostKeyChecking=yes -o UserKnownHostsFile=<known_hosts> \
    deploy-millennium-erp@<host> 'whoami'
```

La línea de `DROPLET_KNOWN_HOSTS` se genera desde fuera y se acepta solo si su huella es la que el Droplet muestra por dentro:

```bash
ssh-keyscan -t ed25519 137.184.231.160          # desde un equipo que no sea el Droplet
ssh-keygen -lf <archivo con esa línea>          # tiene que dar SHA256:eWy2BzWHuFSaHGFO8ABXcX/0qIp0YJ1N1pKek1vCmXw
```

El resultado es una sola línea, `137.184.231.160 ssh-ed25519 AAAA…`. La de `known_hosts_test` nombra `127.0.0.1` y no sirve para el runner, y `known_hosts_public` quedó vacío: si el secreto se cargó desde alguno de los dos, se vuelve a cargar.

Las copias de trabajo de la llave en `/root/github-deploy-setup` se borran después de la primera conexión correcta desde Actions.

**El acceso del administrador.** Hoy root entra por contraseña desde cualquier IP. Se cierra en este orden, sin cerrar la sesión abierta hasta probar la nueva: se crea una llave en el equipo del administrador (`ssh-keygen -t ed25519`), se agrega su pública a `/root/.ssh/authorized_keys` (o a un usuario administrador con sudo), se prueba entrar con ella desde **otra** ventana, y solo entonces se pone en `/etc/ssh/sshd_config.d/` `PasswordAuthentication no` y `PermitRootLogin prohibit-password` (o `no` si hay un usuario administrador), `sudo sshd -t` y `sudo systemctl reload ssh`. La consola web de DigitalOcean queda como acceso de emergencia.

El Droplet no necesita llegar a GitHub: Actions compila y copia el JAR. No se crea una llave Droplet → GitHub ni se instala Maven en el servidor.

### 3.4 systemd

La unidad instalada (verificada con `systemctl cat`):

```ini
[Unit]
Description=Millennium ERP Backend - Spring Boot
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=app-millennium-erp
Group=app-millennium-erp
WorkingDirectory=/opt/apps/backends/millennium-erp/current
EnvironmentFile=/etc/apps/backends/millennium-erp/backend.env
ExecStart=/usr/bin/java -jar /opt/apps/backends/millennium-erp/current/app.jar
Restart=on-failure
RestartSec=5
SuccessExitStatus=143
TimeoutStopSec=60
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/apps/backends/millennium-erp
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

- `EnvironmentFile` lo lee systemd como root al arrancar, así que `backend.env` puede quedar en `600` de root. Un cambio en el archivo se aplica con `restart`; un cambio en la unidad, con `daemon-reload` y después `restart`.
- `SuccessExitStatus=143` hace que la parada normal de la JVM (SIGTERM) no cuente como fallo.
- `ProtectSystem=strict` deja todo el disco de solo lectura salvo `ReadWritePaths`. El backend no escribe archivos propios, y Tomcat usa `/tmp`, que con `PrivateTmp` es privado del servicio.
- `Restart=on-failure` no levanta un servicio parado a mano, y `active` solo dice que el proceso existe, no que la API responda.

La unidad instalada es válida y arrancaría el backend. Le faltan dos ajustes de estabilidad (propuesto, antes del primer arranque; la unidad completa con ellos está en [droplet-paso-a-paso.md §8](droplet-paso-a-paso.md#8-unidad-de-systemd)):

```ini
[Unit]
# Si arranca y cae 5 veces en 10 minutos, se queda en failed en vez de reintentar sin fin
# (cada intento vuelve a correr Flyway contra la base).
StartLimitIntervalSec=600
StartLimitBurst=5

[Service]
# Como el Dockerfile de App Platform: cuánta RAM toma Java para sus objetos, y salir ante un
# OutOfMemoryError para que systemd reinicie en vez de quedar vivo pero roto.
ExecStart=/usr/bin/java -XX:MaxRAMPercentage=50 -XX:+ExitOnOutOfMemoryError -jar /opt/apps/backends/millennium-erp/current/app.jar
```

Sin `-XX:MaxRAMPercentage`, Java toma como máximo el 25 % de la RAM: unos 490 MB en este Droplet, poco margen para exportaciones y picos. Con `50` llega a unos 950 MB y deja el resto para lo que Java usa fuera de ese espacio, Nginx y el sistema. El `Dockerfile` usa `75` porque en un contenedor no corre nada más.

El Droplet tiene 1,9 GiB y ningún swap. Un swap de 2 GiB evita que el kernel mate a Java en un pico, sobre todo en el primer arranque (propuesto; qué es el swap, en [linux-basico.md §5](linux-basico.md#5-memoria-y-swap)):

```bash
sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 3.5 `backend.env`

Las claves son las de `.env.example` del backend, que es la lista de referencia:

| Variable | Valor en el Droplet | Lo que dice el código |
|---|---|---|
| `SPRING_PROFILES_ACTIVE` | `prod` | `application.yaml` no trae perfil: sin esta variable la aplicación no arranca. `prod` es el único perfil sin credenciales de desarrollo; no significa que el servidor sea de producción |
| `PORT` | `8080` | `application-prod.yaml`: `server.port: ${PORT:8080}`. No se agrega `SERVER_PORT`: también funciona y gana sobre `PORT`, y con dos fuentes el puerto real deja de ser evidente |
| `SERVER_ADDRESS` | `127.0.0.1` | No está en los YAML; Spring la toma del entorno. Hace que Java solo escuche en loopback, detrás de Nginx |
| `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_NAME` | El clúster, `25060` y la base | Puerto directo del clúster, nunca un *connection pool* de PgBouncer: Row Level Security se fija por transacción (README, "Despliegue en DigitalOcean") |
| `DATABASE_SSL_MODE` | `require` | Cifra, pero no comprueba la identidad del servidor ([§9](#9-decisiones-pendientes)) |
| `DATABASE_USER`, `DATABASE_PASSWORD` | `erp_mm_app` | La cuenta de la aplicación, sin dueño de nada |
| `FLYWAY_USER`, `FLYWAY_PASSWORD` | `erp_mm_owner` | En `prod` son obligatorias y tienen que ser otra cuenta que `DATABASE_USER`: si no, la aplicación no arranca |
| `IAM_BOOTSTRAP_ADMIN_EMAIL`, `IAM_BOOTSTRAP_ADMIN_PASSWORD` | Definidas | Solo crean una cuenta si no hay ningún SUPERADMIN activo. Contra una base que ya tiene uno no hacen nada |
| `IAM_PUBLIC_SIGN_UP_ENABLED` | `false` | Registro público apagado |
| `DO_ADMIN_USER`, `DO_ADMIN_PASSWORD` | Presentes, con la clave definida | `.env.example` las lista, pero ningún YAML ni clase las lee, y `.do/app.yaml` no las define: son para correr una vez `create-roles.sql` y `prepare-managed-database.sql` con `psql`. No van en `backend.env`, porque `EnvironmentFile` se lo pasa entero al proceso Java y `doadmin` administra el clúster completo. Si se quieren en el Droplet para mantenimiento, van en un archivo aparte de root que systemd no carga (abajo) |

`DATABASE_NAME` conserva el nombre histórico `millennium_erp_dev` por decisión del operador. Las contraseñas que se vieron en capturas son ficticias según el operador; si alguna fue real, se rota.

Para sacar `DO_ADMIN_*` del entorno del servicio sin perderlas:

```bash
sudo install -m 600 -o root -g root /dev/null /etc/apps/backends/millennium-erp/db-admin.env
sudo grep -E '^DO_ADMIN_' /etc/apps/backends/millennium-erp/backend.env | sudo tee /etc/apps/backends/millennium-erp/db-admin.env >/dev/null
sudo sed -i '/^DO_ADMIN_/d' /etc/apps/backends/millennium-erp/backend.env
```

Para auditar sin mostrar secretos:

```bash
sudo grep -E '^(SPRING_PROFILES_ACTIVE|PORT|SERVER_ADDRESS)=' /etc/apps/backends/millennium-erp/backend.env
sudo grep -oE '^[A-Z_]+=' /etc/apps/backends/millennium-erp/backend.env    # solo los nombres
sudo systemctl show millennium-erp -p EnvironmentFiles
```

### 3.6 Nginx

El sitio `api.leasing.dev.millennium.ky` está habilitado y elegido por su `server_name`. El sitio viejo `millennium-erp-api` apuntaba a un dominio de ejemplo y al puerto `8081`; ningún YAML del backend usa `8081`.

Configuración objetivo (propuesto: hoy solo existe la parte de `:80`, y `:443` espera el certificado de origen):

```nginx
server {
    listen 80;
    server_name api.leasing.dev.millennium.ky;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    http2 on;
    server_name api.leasing.dev.millennium.ky;

    ssl_certificate     /etc/ssl/millennium-erp/origin.pem;   # certificado de origen del FQDN exacto
    ssl_certificate_key /etc/ssl/millennium-erp/origin.key;

    access_log /var/log/nginx/millennium-erp-leasing-dev-access.log;
    error_log  /var/log/nginx/millennium-erp-leasing-dev-error.log warn;

    client_max_body_size 20M;

    location / {
        proxy_pass http://127.0.0.1:8080;
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

`application-prod.yaml` tiene `forward-headers-strategy: native`: Tomcat acepta `X-Forwarded-*` solo de un proxy con dirección privada o de loopback, y Nginx en `127.0.0.1` lo es. Con Nginx en `:443`, `$scheme` es `https` y la aplicación sabe que la petición llegó cifrada. Si Cloudflare llegara al origen por `:80` (modo *Flexible*), la aplicación creería que todo es `http`.

Detrás de Cloudflare, `$remote_addr` es una IP de Cloudflare. Para registrar la del cliente se usa `real_ip_header CF-Connecting-IP` con `set_real_ip_from` limitado a los rangos publicados de Cloudflare, y se mantienen esos rangos al día.

### 3.7 Cloudflare

- Registro **A** (no CNAME: el destino es una IPv4) `api.leasing.dev` → IP pública del Droplet, proxy activado cuando el origen tenga `:443`.
- El Universal SSL cubre `millennium.ky` y `*.millennium.ky`, un solo nivel: **no cubre `api.leasing.dev.millennium.ky`**. Hace falta un certificado de borde que nombre ese FQDN (Advanced Certificate Manager o equivalente); sin él, el navegador y la *Pages Function* reciben un error de TLS.
- En el origen, un certificado válido para ese nombre (Cloudflare Origin CA o Let's Encrypt con desafío DNS) y el modo **Full (strict)**. Nunca *Flexible*.
- Después se cambia `BACKEND_ORIGIN` del proyecto de Pages a `https://api.leasing.dev.millennium.ky`, sin barra final.

### 3.8 PostgreSQL administrado

No se instala PostgreSQL en el Droplet: el controlador JDBC viaja dentro del JAR. `postgresql-client` solo sirve para diagnosticar. En el clúster, el Droplet tiene que estar en *Trusted sources* (por IP o por VPC).

```bash
getent hosts <DATABASE_HOST>
nc -vz -w 10 <DATABASE_HOST> 25060
psql "host=<DATABASE_HOST> port=25060 dbname=<DATABASE_NAME> user=erp_mm_app sslmode=require" -c 'select 1'   # pide la clave, no la escribe en la línea
```

La base se prepara una sola vez con `doadmin` y los scripts del README del backend (`create-roles.sql`, `prepare-managed-database.sql`); si ya la usa App Platform, ese paso está hecho.

### 3.9 GitHub

En el repositorio → *Settings* → *Environments* → `development`, con la regla de despliegue limitada a `develop` y los cuatro secretos de la [§3.3](#33-ssh). Un secreto no es un environment: no se crea un environment llamado `DROPLET_HOST`. La llave de deploy tampoco va en *SSH and GPG keys* de una cuenta personal.

`ci.yml` no se toca: corre la suite contra PostgreSQL 18 con las dos cuentas en cada push y PR a `develop` y `master`.

---

## 4. Lo que tiene que hacer `deploy-dev.yml` (propuesto)

Un workflow nuevo, a mano (`workflow_dispatch`), en el environment `development`:

1. **Exigir que `ci.yml` pasó para el mismo commit.** El despliegue no repite las pruebas: necesitan PostgreSQL con las dos cuentas, que solo `ci.yml` prepara. Un `mvn verify` en un job sin ese servicio falla. Compilar sin pruebas es lo mismo que hace el `Dockerfile`, y es aceptable solo porque el SHA ya pasó la suite.
2. Java 25 (Temurin) y `sh ./mvnw -B -DskipTests package`, con el Maven Wrapper como en `ci.yml`.
3. Comprobar que `target/erp-millennium-mobility-backend-0.0.1-SNAPSHOT.jar` existe y su manifiesto trae `Start-Class`, y calcular su `sha256`.
4. Escribir la llave y `known_hosts` desde los secretos (`install -d -m 700 ~/.ssh`, archivos en `600`) y conectar con `StrictHostKeyChecking=yes` y `BatchMode=yes`.
5. Copiar el JAR por SCP a `releases/.incoming-<RELEASE_ID>.jar` y correr en el Droplet el paso de activación de abajo, con el **mismo** `RELEASE_ID`.

El paso de activación, en el Droplet como `deploy-millennium-erp`:

```bash
#!/usr/bin/env bash
set -euo pipefail
APP=/opt/apps/backends/millennium-erp
RELEASE_ID="$1"            # <sha>-<run_id>, el mismo con el que se copió el JAR
EXPECTED_SHA256="$2"
INCOMING="$APP/releases/.incoming-$RELEASE_ID.jar"

echo "$EXPECTED_SHA256  $INCOMING" | sha256sum -c -
install -d -m 2775 "$APP/releases/$RELEASE_ID"
install -m 0644 "$INCOMING" "$APP/releases/$RELEASE_ID/app.jar"
rm -f "$INCOMING"

PREVIOUS=$(readlink "$APP/current" 2>/dev/null || true)   # vacío en el primer despliegue

activate() {
  ln -sfn "$1" "$APP/current.tmp"
  mv -T "$APP/current.tmp" "$APP/current"                  # cambio atómico del enlace
  sudo /usr/bin/systemctl restart millennium-erp.service
}

healthy() {
  # El primer arranque corre todas las migraciones, el ubigeo incluido: hasta 5 minutos.
  for _ in $(seq 1 60); do
    curl -fsS --max-time 5 http://127.0.0.1:8080/actuator/health >/dev/null && return 0
    sleep 5
  done
  return 1
}

activate "releases/$RELEASE_ID"
healthy && { echo "Desplegado $RELEASE_ID"; exit 0; }

echo "La versión $RELEASE_ID no respondió sana" >&2
if [ -n "$PREVIOUS" ]; then
  activate "$PREVIOUS"
  healthy && echo "Vuelta a $PREVIOUS: sana" >&2 || echo "Vuelta a $PREVIOUS: TAMPOCO responde" >&2
fi
exit 1
```

`/actuator/health` es público (`PUBLIC_PATHS` de `WebSecurityConfiguration`) y es el único endpoint de Actuator expuesto, así que no hace falta credencial; responde `200` con `UP` y `503` con `DOWN`, y `curl -f` falla en el segundo. Es la misma ruta que usa App Platform, que espera 120 s antes de la primera prueba.

---

## 5. Lo que hay que saber

**DNS no publica nada.** Un registro A solo lleva el nombre a una IP. Para que la API responda hacen falta Nginx, el certificado, el firewall y Spring escuchando.

**Nginx y Java son capas distintas, con logs distintos.** Nginx: `/var/log/nginx/…-access.log` y `…-error.log`. Spring: `journalctl -u millennium-erp`. Un `502` es de Nginx y dice que detrás no hay nadie.

**Un perfil de Spring no es un entorno.** `prod` elige propiedades. El `<profile>dev</profile>` del `spring-boot-maven-plugin` en el `pom.xml` solo vale para `./mvnw spring-boot:run`; un `java -jar` no tiene perfil y no arranca sin `SPRING_PROFILES_ACTIVE`.

**El puerto: `PORT` es del proyecto, `SERVER_PORT` es de Spring.** `PORT` funciona porque `application-prod.yaml` lo nombra. Se usa uno solo, y `proxy_pass` apunta al mismo número.

**El JAR se revierte, la base no.** Volver a la versión anterior no deshace una migración de Flyway. Si una migración no es compatible hacia atrás, el JAR anterior puede no arrancar (`ddl-auto: validate` compara las entidades con el esquema). Antes de un despliegue con migraciones: respaldo comprobado, y cambios en dos tiempos (*expand and contract*).

**Algunas migraciones se detienen a propósito.** V39, V40, V41 y V43 paran si una fila guardada rompe su regla; el README del backend trae los inventarios de solo lectura que se corren antes. Una migración nueva de ese tipo se trata igual.

**El primer arranque es lento.** Flyway corre todas las migraciones, el ubigeo de Perú incluido, antes de que el servidor escuche. Una sonda de salud que espera 30 s da un falso fallo.

**El cifrado de la base: `require` no es `verify-full`.** `sslmode=require` cifra pero acepta cualquier certificado del servidor. `verify-full` comprueba la CA y el nombre, pero necesita la CA del clúster: hoy la URL de `application-prod.yaml` no pasa `sslrootcert`, y el controlador la buscaría en `~/.postgresql/root.crt` del usuario que ejecuta Java, que con `ProtectHome=true` y un usuario de sistema no sirve. Pasar a `verify-full` es un cambio de código.

**Permisos de archivos.** `700`: solo el dueño (`.ssh`). `600`: solo el dueño lee y escribe (llave privada, `authorized_keys`, `backend.env`). `644`: todos leen (el JAR). `2775`: el setgid hace que lo nuevo herede el grupo de la carpeta, pero no decide los permisos del archivo: eso lo deciden la umask o el `install -m`.

**Lo que se escribe en Bash y lo que va en un archivo.** `server_name` o `proxy_pass` en la terminal dan `command not found`: son directivas de Nginx. Ejecutar `/etc/nginx/sites-available/<archivo>` da `Permission denied`: se lee con `cat` o se edita con `nano`. El nombre del archivo del sitio no importa; importa su `server_name`.

**Siempre `nginx -t` antes de `reload`**, y `daemon-reload` después de tocar una unidad.

**Comandos de auditoría que no muestran secretos:**

```bash
hostname; grep PRETTY_NAME /etc/os-release; /usr/bin/java -version
id app-millennium-erp; id deploy-millennium-erp
tree -a -pug -L 3 /opt/apps/backends/millennium-erp
readlink -f /opt/apps/backends/millennium-erp/current
systemctl is-enabled millennium-erp; systemctl is-active millennium-erp
sudo systemd-analyze verify /etc/systemd/system/millennium-erp.service
sudo journalctl -u millennium-erp -n 100 --no-pager
sudo nginx -t
sudo nginx -T 2>/dev/null | grep -nE 'server_name|listen|proxy_pass'
sudo ss -ltnp | grep -E ':80 |:443 |:8080 '
curl -i -H 'Host: api.leasing.dev.millennium.ky' http://127.0.0.1/
curl -i --max-time 8 http://127.0.0.1:8080/actuator/health
```

---

## 6. Primer despliegue y los siguientes

Antes del primero:

1. Decidir la convivencia con App Platform ([§9](#9-decisiones-pendientes)).
2. Confirmar que `backend.env` quedó guardado con `prod`, `PORT=8080` y `SERVER_ADDRESS=127.0.0.1`, y quitar `DO_ADMIN_*`.
3. Probar con `psql` la conexión, el TLS y las dos cuentas desde el Droplet.
4. Tener un respaldo restaurable de la base.
5. Ver verde `ci.yml` para el commit que se va a desplegar.
6. Crear `deploy-dev.yml` y probar primero solo la conexión SSH desde el runner (`whoami && hostname`).

El primero no tiene versión anterior: si falla, no hay vuelta automática y el servicio queda sin publicar hasta corregirlo. Después de un primer despliegue sano, un administrador corre una vez `sudo systemctl enable millennium-erp` para que arranque con el servidor.

Después de cada despliegue:

```bash
readlink -f /opt/apps/backends/millennium-erp/current
sudo systemctl status millennium-erp --no-pager -l
sudo journalctl -u millennium-erp -n 100 --no-pager
curl -i --max-time 8 http://127.0.0.1:8080/actuator/health
curl -i --max-time 8 -H 'Host: api.leasing.dev.millennium.ky' http://127.0.0.1/actuator/health
curl -Iv --max-time 15 https://api.leasing.dev.millennium.ky/actuator/health    # desde fuera, cuando haya :443
```

La vuelta atrás se prueba de verdad solo cuando haya dos versiones sanas. Las versiones viejas no se borran con un automatismo hasta tener una retención decidida.

---

## 7. Diagnóstico

| Síntoma | Qué mirar |
|---|---|
| "Welcome to nginx" con el `Host` de la API | Nginx eligió otro sitio: revisar `sites-enabled` y el `server_name`, y recargar |
| `502 Bad Gateway` | Java no escucha en el puerto de `proxy_pass`: `systemctl status`, `journalctl`, `ss -ltnp` |
| `404` en `/` | La API no tiene raíz: probar `/actuator/health` |
| `Connection refused` en `127.0.0.1:8080` | Spring no arrancó o escucha en otro puerto: `PORT` y el log |
| systemd `status=200/CHDIR` | `current` no existe o el usuario de la app no puede atravesarlo |
| La aplicación termina al arrancar sin perfil | Falta `SPRING_PROFILES_ACTIVE` |
| La aplicación termina diciendo que las cuentas son la misma | `FLYWAY_USER` falta o es igual a `DATABASE_USER` |
| `FlywayException` o una migración detenida | Parar el despliegue, leer qué fila la detuvo, decidir; el JAR anterior no lo arregla |
| Timeout a `25060` | *Trusted sources*, VPC, firewall; repetir antes de concluir (el primer `nc` falló y el segundo no) |
| `Host key verification failed` | `DROPLET_KNOWN_HOSTS` no corresponde a `DROPLET_HOST`: comparar huellas, no apagar la verificación |
| `Permission denied (publickey)` | `authorized_keys`, sus permisos, el usuario y la llave usada |
| `sudo: a password is required` en Actions | El comando no coincide letra por letra con sudoers |
| Error de TLS en `https://api.leasing.dev…` | Certificado de borde para el FQDN de dos niveles, certificado de origen y Full (strict) |
| La sesión no se mantiene en el navegador | Se está llamando al backend directo y no por `dev.erp.millennium.ky/api/` |

---

## 8. Lecciones de la configuración

| Lo que pasó | Lo que queda |
|---|---|
| `BatchMode=yes` respondió `Host key verification failed` y parecía un problema de la llave de deploy | Era la identidad del servidor. Se resolvió comparando huellas y con `UserKnownHostsFile`, sin apagar `StrictHostKeyChecking` |
| La prueba SSH a `127.0.0.1` funcionó | Prueba la llave y el usuario, no el acceso desde un runner, ni la IP de `DROPLET_HOST`, ni `DROPLET_KNOWN_HOSTS` |
| El sitio viejo apuntaba a un dominio de ejemplo y al `8081` | Revisar lo que Nginx tiene cargado (`nginx -T`) antes de editar; el puerto real sale del YAML |
| Con el sitio correcto habilitado, `curl` local dio `502` | Es el resultado esperado sin backend: dice que el proxy está bien, no que la API funcione |
| `nc` a la base: timeout y luego correcto | Repetir y diagnosticar antes de culpar a *Trusted sources* |
| `nc` con `HOST_POSTGRESQL` literal, y otro con host vacío por un `sed` mal armado | Los marcadores se reemplazan, y el host se lee con `grep '^DATABASE_HOST='` sin imprimir el resto |
| `git ls-remote https://…` desde el Droplet pidió credenciales | GitHub no acepta la contraseña de la cuenta por HTTPS, y además no hace falta: Actions compila y copia el JAR |
| `SPRING_PROFILES_ACTIVE=prod` se editó en nano y la captura mostraba el asterisco de sin guardar | Una captura del editor no prueba nada: se confirma con `grep` |
| Contraseñas visibles en capturas, ficticias según el operador | Ninguna clave en capturas, chats ni documentos; si fue real, se rota |
| Cuatro secretos creados en GitHub | Existen; si sus valores son correctos lo dice solo una corrida |
| `ci.yml` verde | Pasaron las pruebas; no hubo despliegue |
| El primer borrador de despliegue derivaba el nombre del archivo temporal de un identificador distinto del de la copia | Un solo `RELEASE_ID` para copiar, verificar e instalar |
| El manual anterior proponía un workflow con `mvn clean verify` sin base | Las pruebas necesitan PostgreSQL con las dos cuentas: se corren en `ci.yml` y el despliegue exige su verde |
| El manual anterior recomendaba agregar `SERVER_PORT` y temía un `401` en `/actuator/health` | El código usa `PORT`, y el health es público: leer el código antes de configurar alrededor de él |

---

## 9. Decisiones pendientes

Las decide el responsable; este documento no las toma.

1. **App Platform y el Droplet.** `.do/app.yaml` despliega `develop` en cada push contra la base de `DATABASE_NAME`. Si el Droplet arranca contra la misma base mientras App Platform sigue activo, dos versiones distintas migran y leen el mismo esquema, y el proceso programado de `iam` corre dos veces. Opciones: apagar App Platform al pasar al Droplet (y entonces actualizar el README del backend, `.do/app.yaml` y el `Dockerfile`), o darle al Droplet otra base.
2. **La verificación completa (`verify-full`) en la conexión a PostgreSQL**, que pide un cambio en `application-prod.yaml` ([§5](#5-lo-que-hay-que-saber)).
3. **Despliegue automático en cada push a `develop`**, solo después de varios despliegues manuales sanos y una vuelta atrás probada.
4. **Retención de versiones y de logs** (`logrotate`), respaldos de la base con prueba de restauración, y alertas.
5. **Firewall**, hoy inexistente en el servidor: SSH limitado por origen, `8080` nunca expuesto (lo asegura `SERVER_ADDRESS`), y `:80`/`:443` solo desde los rangos de Cloudflare. Puede ser `ufw` o un *Cloud Firewall* de DigitalOcean; con `ufw` se permite el 22 **antes** de `ufw enable`, para no cortar la sesión.
6. **Red privada hacia la base.** El host de la base resuelve a una IP pública: el tráfico va cifrado por Internet. Si el Droplet y el clúster están en la misma región y VPC, el host privado (`private-…`) lo deja dentro de la red de DigitalOcean.
7. **El sitio `default` de Nginx** responde `200` a cualquier petición por IP o con otro `Host`. Se puede dejar en `return 444;` para que el servidor no conteste nada que no sea la API, y borrar `sites-available/millennium-erp-api`, que ya no se usa.

---

## 10. Siguiente sesión

- [x] Confirmar `backend.env` (perfil, puerto, dirección): verificado el 20/10/2026.
- [ ] Pasar `DO_ADMIN_USER` y `DO_ADMIN_PASSWORD` de `backend.env` a `db-admin.env` (o borrarlas si no se usan en el Droplet).
- [ ] `chmod 0440 /etc/sudoers.d/deploy-millennium-erp`.
- [ ] Llave de administrador, y después SSH sin contraseña y sin root por contraseña.
- [ ] Firewall (`ufw` o Cloud Firewall) y swap.
- [ ] Volver a cargar `DROPLET_KNOWN_HOSTS` con la línea de `137.184.231.160` verificada por huella.
- [ ] Decidir la convivencia con App Platform.
- [ ] Probar con `psql` desde el Droplet la conexión con TLS y las dos cuentas.
- [ ] Respaldo restaurable de la base.
- [ ] Agregar a la unidad `StartLimit*` y las opciones de memoria de la JVM.
- [ ] Crear `deploy-dev.yml`; primera corrida solo con la conexión SSH.
- [ ] Primer despliegue manual, con `journalctl` y el health local y por Nginx.
- [ ] `systemctl enable` después del primer despliegue sano.
- [ ] Certificado de borde para `api.leasing.dev.millennium.ky`, certificado de origen, Nginx `:443` y Full (strict).
- [ ] `BACKEND_ORIGIN` de Pages apuntando al Droplet, y la sonda del frontend por `dev.erp.millennium.ky`.
- [ ] Probar la vuelta atrás cuando haya dos versiones sanas.
- [ ] Actualizar este documento con la fecha, el commit y la evidencia de cada paso.
