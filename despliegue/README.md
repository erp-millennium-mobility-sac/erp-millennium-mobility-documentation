# Despliegue — ERP Millennium Mobility

**Actualizado:** 19/10/2026

Aquí se guarda cómo se configuran los servidores en los que corre el sistema, lo que hay que saber para repetirlo y lo que se aprendió montándolos. Cómo se compila, se prueba y qué variables lee la aplicación está en el README de cada repositorio; esta carpeta cubre lo que vive fuera del código: el servidor, la red, los certificados y los secretos.

| Documento | Qué cubre | Estado |
|---|---|---|
| [droplet-digitalocean.md](droplet-digitalocean.md) | El backend en un Droplet Ubuntu con Nginx, systemd, Cloudflare y PostgreSQL administrado, desplegado desde GitHub Actions: estado, porqué de cada pieza, lo que hay que saber, diagnóstico y lecciones | Servidor preparado, sin ningún despliegue |
| [linux-basico.md](linux-basico.md) | Las carpetas de un servidor Linux y qué se configura en cada una, los comandos básicos por tema, los permisos y los hábitos que evitan romper el servidor | Referencia |
| [droplet-paso-a-paso.md](droplet-paso-a-paso.md) | La configuración ideal como plantilla genérica, con `<marcadores>` para cualquier implementación: el mapa del servidor y los comandos en orden, de un Droplet nuevo al backend corriendo. Cada fase la compara con nuestro Droplet, y un cuadro inicial resume qué le falta y qué tan importante es | Plantilla; el contraste, al 20/10/2026 |

## Cómo se escribe

- Cada estado lleva su evidencia: verificado (se vio la salida), informado (lo dijo el operador), pendiente o propuesto. Nada se da por hecho sin haberlo visto funcionar.
- Lo que sale del código (puertos, perfiles, variables, rutas públicas) se comprueba en el código y se cita el archivo.
- Nunca se escriben secretos, claves privadas, contraseñas, tokens, el contenido de un archivo de entorno ni el hostname real de una base.
- Al terminar una sesión de trabajo en un servidor se actualiza su documento con la fecha, el commit desplegado y lo que se comprobó.
