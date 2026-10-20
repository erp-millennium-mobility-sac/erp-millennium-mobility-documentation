# Linux básico para administrar el Droplet

**Actualizado:** 20/10/2026 · **Acompaña a:** [droplet-paso-a-paso.md](droplet-paso-a-paso.md) y [droplet-digitalocean.md](droplet-digitalocean.md)

Lo mínimo que hay que saber de Linux (Ubuntu 24.04) para configurar, revisar y reparar el Droplet sin romperlo. Los ejemplos usan las rutas reales del servidor.

---

## 1. Las carpetas de un servidor Linux

Linux no tiene `C:\` ni `D:\`: todo cuelga de una sola raíz, `/`. Cada carpeta de primer nivel tiene un uso fijo, el mismo en cualquier servidor (*Filesystem Hierarchy Standard*). Saberlo dice dónde buscar algo y dónde poner lo propio.

| Carpeta | Qué guarda | Qué se suele configurar o mirar ahí | En este Droplet |
|---|---|---|---|
| `/etc` | La configuración de todo el sistema, en archivos de texto | Servicios, sitios web, SSH, usuarios, sudo, firewall, tareas programadas | `apps/…/backend.env`, `nginx/`, `systemd/system/`, `ssh/`, `sudoers.d/` |
| `/opt` | Programas que no vienen de `apt`, cada uno en su carpeta | Aplicaciones propias, instaladas a mano | `apps/backends/millennium-erp/` (los JAR) |
| `/var` | Lo que cambia mientras el sistema corre | Logs, datos de servicios, colas, cachés | `log/nginx/`, `lib/apps/…` |
| `/var/log` | Los logs en archivo | Leer errores; rotación con `logrotate` | Los de Nginx. Los de systemd van al journal |
| `/var/lib` | Datos persistentes de cada servicio | Lo que una aplicación guarda entre reinicios | `apps/backends/millennium-erp/` |
| `/var/www` | Sitios web estáticos, por costumbre | Archivos HTML que sirve Nginx | No se usa: el frontend está en Cloudflare Pages |
| `/home` | Una carpeta por usuario humano o de servicio con login | Llaves SSH (`~/.ssh`), archivos personales | `deploy-millennium-erp/.ssh/` |
| `/root` | La carpeta personal de root (no está en `/home`) | Llaves de root, scripts del administrador | `.ssh/`, `github-deploy-setup/` (temporal) |
| `/usr` | Los programas instalados por el sistema y sus bibliotecas | Casi nunca se toca a mano | `/usr/bin/java`, `/usr/sbin/nginx` |
| `/usr/local` | Programas compilados o instalados a mano por el administrador | Herramientas propias fuera de `apt` | Vacío |
| `/bin`, `/sbin`, `/lib` | Hoy son enlaces a `/usr/bin`, `/usr/sbin`, `/usr/lib` | Nada | — |
| `/srv` | Datos que el servidor ofrece (sitios, FTP) | Alternativa a `/var/www` | No se usa |
| `/tmp` | Archivos temporales; se borra al reiniciar | Nada importante, nunca | Lo que dejan pruebas sueltas |
| `/boot` | El kernel y el arranque | Nada | — |
| `/dev` | Los dispositivos como archivos (discos, terminal) | Nada a mano | `/dev/vda1`, el disco |
| `/proc`, `/sys` | Vista en vivo del kernel y los procesos; no ocupan disco | Leer información (`/proc/meminfo`) | — |
| `/run` | Estado de lo que corre ahora (PIDs, sockets); se borra al reiniciar | Nada | — |
| `/mnt`, `/media` | Puntos de montaje de discos extra | Montar un volumen de DigitalOcean | No se usa |
| `/snap` | Paquetes instalados con `snap` | Nada | — |

### Dentro de `/etc`

| Ruta | Qué se configura |
|---|---|
| `/etc/nginx/nginx.conf` | Nginx en general (rara vez se toca) |
| `/etc/nginx/sites-available/` | Un archivo por sitio; existir ahí no lo activa |
| `/etc/nginx/sites-enabled/` | Enlaces a los sitios activos (`ln -s`) |
| `/etc/systemd/system/` | Las unidades propias: cómo arranca cada servicio |
| `/etc/ssh/sshd_config` y `sshd_config.d/` | El servidor SSH: puerto, contraseñas, root |
| `/etc/sudoers.d/` | Qué puede hacer cada usuario como root; se edita con `visudo -f` |
| `/etc/ufw/` | El firewall; se maneja con el comando `ufw`, no editando archivos |
| `/etc/fstab` | Qué se monta al arrancar (discos, swap) |
| `/etc/hosts` y `/etc/hostname` | El nombre del servidor y nombres locales |
| `/etc/apt/sources.list.d/` | Repositorios extra de `apt` (el de Microsoft para Java) |
| `/etc/ssl/` | Certificados |
| `/etc/logrotate.d/` | Cada cuánto se rotan y borran los logs |
| `/etc/cron.d/` | Tareas programadas del sistema |
| `/etc/passwd`, `/etc/group` | Usuarios y grupos (se leen con `getent`, no se editan a mano) |
| `/etc/os-release` | La versión de Ubuntu |

**Regla práctica:** el programa va en `/opt`, su configuración en `/etc`, sus datos en `/var/lib` y sus logs en `/var/log` o en el journal. Así se puede reemplazar el programa sin tocar la configuración ni los datos.

---

## 2. Comandos básicos

### Moverse y mirar

| Comando | Qué hace |
|---|---|
| `pwd` | Dice en qué carpeta se está |
| `cd /opt/apps` · `cd ..` · `cd -` · `cd ~` | Ir a una carpeta · subir una · volver a la anterior · ir a la personal |
| `ls -lah` | Lista con permisos, dueño, tamaño legible y archivos ocultos (los que empiezan con `.`) |
| `ls -ld /opt/apps` | Los datos de la carpeta misma, no su contenido |
| `tree -L 2 /etc/nginx` | Árbol hasta 2 niveles |
| `tree -d -L 4 /opt/apps` | Solo carpetas |
| `tree -a -pug /opt/apps` | Con ocultos, permisos (`-p`), usuario (`-u`) y grupo (`-g`) |
| `tree -h --du /var/log` | Con tamaños |
| `stat archivo` | Todo de un archivo: permisos en número, dueño, fechas |
| `namei -l /ruta/completa` | Permisos de cada carpeta del camino: dice dónde se corta el acceso |
| `readlink -f current` | A dónde apunta de verdad un enlace |
| `file app.jar` | Qué tipo de archivo es |

### Leer archivos

| Comando | Qué hace |
|---|---|
| `cat archivo` | Lo muestra entero |
| `less archivo` | Lo muestra por páginas: flechas para moverse, `/texto` para buscar, `q` para salir |
| `head -n 20` · `tail -n 50` | Las primeras o últimas líneas |
| `tail -f archivo.log` | Sigue un log en vivo (`Ctrl+C` para salir) |
| `grep -n 'texto' archivo` | Las líneas que contienen el texto, con su número |
| `grep -rn 'proxy_pass' /etc/nginx` | Busca en todos los archivos de una carpeta |
| `grep -E '^(PORT\|SERVER_ADDRESS)=' archivo` | Busca varias claves a la vez (sin mostrar las demás) |
| `wc -l archivo` | Cuántas líneas tiene |

### Crear, copiar, mover, borrar

| Comando | Qué hace |
|---|---|
| `mkdir -p a/b/c` | Crea la carpeta y las intermedias; no falla si existe |
| `touch archivo` | Crea un archivo vacío (o actualiza su fecha) |
| `nano archivo` | Editor simple: `Ctrl+O` `Enter` guarda, `Ctrl+X` sale, `Ctrl+W` busca, `Ctrl+K` corta una línea |
| `cp archivo archivo.bak-$(date +%F)` | Copia de seguridad con la fecha, **antes** de editar |
| `cp -a origen destino` | Copia conservando dueño, permisos y fechas |
| `mv viejo nuevo` | Mueve o renombra |
| `ln -s destino enlace` | Crea un enlace simbólico (como un acceso directo) |
| `ln -sfn destino enlace` | Lo reemplaza si existe (`-n`: si es un enlace a carpeta, cambia el enlace y no entra en él) |
| `rm archivo` · `rm -r carpeta` | Borra. **No hay papelera**: lo borrado no vuelve |
| `rmdir carpeta` | Borra una carpeta solo si está vacía: la forma segura |
| `install -d -m 750 -o usuario -g grupo carpeta` | Crea carpeta con dueño y permisos en un paso |
| `install -m 600 origen destino` | Copia un archivo y le pone permisos en un paso |

### Usuarios y permisos

| Comando | Qué hace |
|---|---|
| `whoami` · `id` · `id usuario` | Quién soy · mis grupos · los de otro |
| `getent passwd usuario` | Su carpeta personal y su shell |
| `useradd --system …` · `adduser …` | Crea un usuario de servicio · uno con carpeta personal |
| `usermod -aG grupo usuario` | Lo agrega a un grupo (**sin `-a` lo saca de todos los demás**) |
| `chown usuario:grupo ruta` · `chown -R …` | Cambia el dueño · también de todo lo de adentro |
| `chmod 640 archivo` · `chmod -R …` | Cambia los permisos ([§3](#3-permisos)) |
| `sudo comando` | Ejecuta como root |
| `sudo -u usuario comando` | Ejecuta como otro usuario: sirve para probar permisos |
| `sudo -u deploy-millennium-erp test -w /ruta && echo SI` | ¿Puede ese usuario escribir ahí? (`-r` leer, `-x` entrar) |
| `sudo -l -U usuario` | Qué puede hacer ese usuario con sudo |
| `umask` | Los permisos que se le quitan a lo que se crea (`0022` → archivos `644`) |

### Paquetes

| Comando | Qué hace |
|---|---|
| `apt update` | Actualiza la lista de paquetes (no instala nada) |
| `apt upgrade` | Instala las actualizaciones |
| `apt install paquete` · `apt remove paquete` | Instala · desinstala |
| `apt-cache search texto` | Busca paquetes |
| `apt list --installed \| grep nginx` | ¿Está instalado? |
| `command -v java` | Dónde está un programa (vacío si no existe) |

### Servicios (systemd)

| Comando | Qué hace |
|---|---|
| `systemctl status millennium-erp` | Estado, PID, memoria y las últimas líneas del log |
| `systemctl start` · `stop` · `restart` | Arranca · para · reinicia |
| `systemctl reload nginx` | Relee la configuración sin cortar conexiones (si el servicio lo soporta) |
| `systemctl enable` · `disable` | Arranca o no al encender el servidor (no lo arranca ahora; `enable --now` sí) |
| `systemctl is-active` · `is-enabled` | Respuesta corta: `active`/`inactive`, `enabled`/`disabled` |
| `systemctl cat millennium-erp` | La unidad tal como systemd la tiene cargada |
| `systemctl daemon-reload` | Obligatorio después de editar una unidad |
| `systemctl list-units --failed` | Los servicios que fallaron |
| `systemd-analyze verify archivo.service` | Revisa una unidad sin arrancarla |

### Logs

| Comando | Qué hace |
|---|---|
| `journalctl -u millennium-erp -n 100 --no-pager` | Las últimas 100 líneas del backend |
| `journalctl -u millennium-erp -f` | En vivo |
| `journalctl -u millennium-erp --since '10 min ago'` | Desde hace un rato |
| `journalctl -u millennium-erp -b` | Desde el último arranque del servidor |
| `journalctl -p err -b` | Solo errores de todo el sistema |
| `tail -f /var/log/nginx/millennium-erp-leasing-dev-error.log` | Errores del proxy en vivo |

### Red

| Comando | Qué hace |
|---|---|
| `ip -4 -brief address` | Las IPs del servidor |
| `ss -ltnp` | Qué programas escuchan en qué puertos (`127.0.0.1:8080` = solo local; `0.0.0.0:80` = todas las interfaces) |
| `curl -i http://127.0.0.1:8080/actuator/health` | Hace una petición y muestra la respuesta con cabeceras |
| `curl -I -H 'Host: api.leasing.dev.millennium.ky' http://127.0.0.1/` | Prueba un sitio de Nginx sin pasar por el DNS |
| `curl -Iv https://…` | Muestra además el saludo TLS y el certificado |
| `getent hosts nombre` · `dig +short nombre` | A qué IP resuelve un nombre |
| `nc -vz -w 10 host 25060` | ¿Está abierto ese puerto? |
| `ping -c 3 host` | ¿Responde? (muchos servidores no responden ping) |
| `ufw status verbose` | Estado del firewall |

### Sistema

| Comando | Qué hace |
|---|---|
| `df -h` | Espacio en disco |
| `du -sh /var/log/*` | Cuánto ocupa cada cosa |
| `free -h` | Memoria y swap |
| `top` · `htop` | Procesos en vivo (`q` sale) |
| `ps aux \| grep java` | ¿Está corriendo Java? |
| `uptime` | Cuánto lleva encendido y su carga |
| `timedatectl` | Hora, zona y sincronización |
| `reboot` | Reinicia el servidor (cierra la sesión) |

### SSH y copias, desde la laptop

| Comando | Qué hace |
|---|---|
| `ssh root@137.184.231.160` | Entra |
| `ssh -i llave usuario@host 'comando'` | Ejecuta un comando y vuelve |
| `scp archivo root@137.184.231.160:/tmp/` | Copia hacia el servidor |
| `scp root@137.184.231.160:/ruta/archivo .` | Copia desde el servidor |
| `ssh-keygen -t ed25519` | Crea un par de llaves |
| `ssh-keygen -lf archivo.pub` | La huella de una llave pública |
| `ssh-keyscan -t ed25519 host` | La llave de host de un servidor, para `known_hosts` |
| `exit` o `Ctrl+D` | Sale de la sesión |

### Atajos de la terminal

| Atajo | Qué hace |
|---|---|
| `Tab` | Completa nombres de archivos y comandos (dos veces: muestra opciones) |
| `↑` · `Ctrl+R` | Comando anterior · buscar en el historial |
| `Ctrl+C` | Corta el comando que está corriendo |
| `Ctrl+L` | Limpia la pantalla |
| `history \| tail -30` | Los últimos comandos escritos |
| `comando --help` · `man comando` | Ayuda corta · manual completo (`q` sale) |

---

## 3. Permisos

`ls -l` muestra algo como `-rw-r----- 1 root app-millennium-erp … backend.env`:

```text
 -    rw-     r--     ---
tipo  dueño   grupo   los demás
```

- **Tipo:** `-` archivo, `d` carpeta, `l` enlace.
- **r** leer (4), **w** escribir (2), **x** ejecutar un archivo o **entrar** en una carpeta (1). Se suman por grupo de tres: `rw-` = 6, `r-x` = 5, `rwx` = 7.

| Número | Letras | Uso típico | En este Droplet |
|---|---|---|---|
| `600` | `rw-------` | Secretos: solo el dueño lee y escribe | `backend.env`, `authorized_keys`, llaves privadas, `origin.key`, `/swapfile` |
| `440` | `r--r-----` | Solo lectura para dueño y grupo | Archivos de `sudoers.d` |
| `644` | `rw-r--r--` | Archivo que todos leen y solo el dueño cambia | `app.jar`, la unidad, los sitios de Nginx, `origin.pem` |
| `700` | `rwx------` | Carpeta privada | `.ssh`, `github-deploy-setup` |
| `750` | `rwxr-x---` | Carpeta del dueño; su grupo entra y lee | `/var/lib/apps/…` |
| `755` | `rwxr-xr-x` | Carpeta o programa que todos usan | `/opt/apps`, `/etc/apps/…` |
| `2775` | `rwxrwsr-x` | Carpeta compartida por un grupo | `/opt/apps/backends/millennium-erp` y `releases` |
| `777` | `rwxrwxrwx` | **Nunca.** Cualquiera cambia cualquier cosa | — |

**Bits especiales** (el primer dígito de cuatro):

- **setgid (`2`) en una carpeta:** lo que se crea dentro hereda el **grupo** de la carpeta. Aparece como `s` en el lugar de la `x` del grupo (`rwxrwsr-x`). No decide los permisos del archivo nuevo: eso lo decide la `umask` o el `-m` de `install`.
- **sticky (`1`) en una carpeta:** cada uno solo borra lo suyo. Es el de `/tmp` (`drwxrwxrwt`).
- **setuid (`4`) en un programa:** corre como su dueño. Es el de `sudo` y `passwd`; no se pone en nada propio.

**Para llegar a un archivo** hace falta `x` en **todas** las carpetas del camino, además del permiso sobre el archivo. Por eso `namei -l` es el primer comando cuando algo da `Permission denied`.

---

## 4. Hábitos que evitan romper el servidor

1. **Copia antes de editar:** `cp archivo archivo.bak-$(date +%F)`.
2. **Probar antes de aplicar:** `nginx -t` antes de `reload`; `visudo -c` después de tocar sudoers; `sshd -t` antes de recargar SSH; `systemd-analyze verify` antes de arrancar una unidad.
3. **No cerrar la sesión abierta** cuando se cambia SSH o el firewall: se prueba desde una ventana nueva. La consola web de DigitalOcean entra aunque SSH falle.
4. **Sudoers solo con `visudo -f`**: un error de sintaxis en sudoers deja sin `sudo` a todos.
5. **Nunca `chmod 777`** ni `chmod -R` sobre carpetas del sistema. Si algo no tiene permiso, se busca cuál falta con `namei -l` y se da ese.
6. **Cuidado con `rm -rf`:** se escribe la ruta completa, se revisa con `ls` antes, y nunca con una variable que podría estar vacía (`rm -rf $X/` con `X` vacía es `rm -rf /`).
7. **Secretos fuera de la pantalla y del historial:** se escriben dentro de `nano`, no en la línea de comandos (quedan en `~/.bash_history`). Para revisar un archivo de entorno se muestran solo los nombres: `grep -oE '^[A-Z_]+=' archivo`.
8. **Archivos pegados desde Windows** pueden traer `\r` al final de cada línea y romper scripts y variables: `grep -c $'\r' archivo` lo detecta y `sed -i 's/\r$//' archivo` lo quita.
9. **Las directivas no son comandos:** `server_name` o `proxy_pass` escritos en la terminal dan `command not found`; van dentro del archivo del sitio. Un archivo de configuración no se ejecuta, se edita.
10. **Ver antes de concluir:** que un archivo esté bien escrito no prueba que el servicio lo use. Se mira lo cargado (`nginx -T`, `systemctl cat`, `sshd -T`) y lo que escucha (`ss -ltnp`).

---

## 5. Memoria y swap

La **RAM** es la memoria rápida donde corren los programas; cuando se apaga el servidor se vacía. El **swap** es un espacio del **disco** que Linux usa como RAM de reserva cuando la de verdad se llena: mueve ahí lo que menos se está usando y libera RAM para lo que la necesita ahora.

- **Sin swap**, cuando la RAM se acaba el kernel no tiene adónde mover nada y activa el *OOM killer* (*out of memory*): elige un proceso y lo mata. En un servidor como este casi siempre es Java, porque es el que más ocupa. El backend cae sin aviso, y en el journal aparece `Killed` o `oom-kill`.
- **Con swap**, el pico pasa: el servidor se pone más lento un momento, pero nada muere.
- **El swap no es más RAM.** Es mucho más lento. Si el servidor vive usando swap (`free -h` muestra el swap ocupado todo el tiempo), le falta RAM y hay que agrandar el Droplet. El swap es un seguro para los picos, no una solución para la falta de memoria.

Cuánto: igual a la RAM en servidores de hasta 2 GB; 2 GB en los más grandes.

| Comando | Qué hace |
|---|---|
| `free -h` | RAM y swap: total, usado y disponible (`available` es lo que importa) |
| `swapon --show` | Qué swap hay activo |
| `fallocate -l 2G /swapfile` | Reserva un archivo de 2 GB en el disco |
| `chmod 600 /swapfile` | Solo root lo lee: puede contener datos de la memoria de cualquier programa |
| `mkswap /swapfile` · `swapon /swapfile` | Lo prepara como swap · lo activa ahora |
| `echo '/swapfile none swap sw 0 0' >> /etc/fstab` | Lo activa en cada arranque |
| `journalctl -k \| grep -i oom` | ¿El kernel mató algo por falta de memoria? |

**Java y la memoria.** Java reserva un espacio propio (el *heap*) para sus objetos, con un tope. Si no se le dice, el tope es el 25 % de la RAM. `-XX:MaxRAMPercentage=50` lo sube a la mitad. Además del heap, Java usa memoria para su código, sus hilos y sus bibliotecas, así que el proceso completo ocupa bastante más que el heap: por eso no se le da el 100 %.
