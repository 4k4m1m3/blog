---
title: CTF-DockerLabs-Vacaciones
published: true
---

## WriteUps maquina Vacaciones de DockerLabs

---
tags:
  - CTF
  - estado/completado
  - plataforma: [DockerLabs]
  - dificultad: Muy fácil
  - autor: [Romabri]

# Datos

[!INFO] Vacaciones
>  **Nombre:** Vacaciones
>  **SO:** Linux
>  **Dificultad:** Muy fácil
>  **Enlace:** [Dockerlabs](https://dockerlabs.es/)

[!TODO] Objetivo
> 🚩Ingresar a la maquina como algún usuario.
> 🚩Elevar privilegios una vez obtenido el acceso.

El primer paso consiste en iniciar la máquina, lo cual es tan sencillo como ejecutar el siguiente comando después de haber descargado la maquina:

```bash
└─$ sudo bash auto_deploy.sh vacaciones.tar
```

# Reconocimiento

> Una vez iniciada la maquina, el mismo script de inicio me da la dirección IP a lo cual procedo a realizar un escaneo de puertos de la maquina y el resultado es el siguiente:

```bash
└─# nmap 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-22 11:46 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000010s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.26 seconds
```

Al darme cuenta que existe el `puerto 80` abierto, procedo a visitar desde el navegador la dirección IP de la maquina y encuentro con una pagina totalmente en blanco, pero que al visitar su código fuente me aparece el siguiente mensaje:

![CodigoFuenteVacaciones.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/CodigoFuenteVacaciones.png)

De este mensaje se pueden sacar la siguiente información, existen dos posibles usuarios `juan` y `camilo` y adicional que debo buscar algún directorio donde pueda existir un correo electrónico, que posiblemente la ruta sea: `/var/mail/`

# Fuerza bruta SSH

> Ya que el código fuente, me proporciona 2 usuarios procedo a utilizar `hydra` para hacer un ataque de fuerza bruta, creo un archivo llamado `users.txt` en donde colocare los posibles usuarios.

>> ![CatUserVacaciones.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/CatUserVacaciones.png)

Y listo, luego de pasar bastante rato, al final logro encontrar la contraseña para el usuario `camilo` con esto he logrado acceder por `ssh`, con el usuario `camilo` y la contraseña encontrada.

`ssh camilo@172.17.0.2`

# Acceso por SSH

> Ahora que tengo acceso con el usuario `camilo`, lo primero que hago para elevar privilegios es colocar: `sudo -l` y resuelta que este usuario no puede ejecutar `sudo`, también intento con el comando `compgen -u` ver si existen otros usuarios, pero e indica que `compgen: not found`, así que ingreso al directorio `/home` y allí veo que existen los directorios: `camilo juan pedro`

Al ver que existe el directorio `/home/juan`, recordé que el código fuente menciona que `juan` le ha dejado un correo a `camilo` así que de inmediato visito el directorio: `/var/mail` y allí estaba el directorio `camilo` con un archivo de texto llamado: `correo.txt` que contiene la clave de SSH para el usuario `juan`

![ClaveCamiloVacaciones.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/ClaveCamiloVacaciones.png)

# Escalada de privilegios

Ya estando conectado a la cuenta de usuario de `juan` luego de pivotear desde el usuario `camilo` hago un `sudo -l` para ver algún script vulnerable y efectivamente me encuentro con `(ALL) NOPASSWD: /usr/bin/ruby` y buscando en [GTFOBins](https://gtfobins.github.io/) me indica que si coloco el siguiente comando: `sudo ruby -e 'exec "/bin/sh"'` puedo acceder con el usuario `root`

# pwned

![pwnedVacaciones.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/pwnedVacaciones.png)

# Comandos

[!IMPORTANT] Resumen de comandos utilizados
> `sudo bash auto_deploy.sh vacaciones.tar`
> `nmap 172.17.0.2`
> `cd /var/mail/camilo`
> `sudo -l`
> `sudo ruby -e 'exec "/bin/sh"'`


```
¡Que la fuerza del hacking ético nos acompañe! :)
```

|   |
|:--|
|   |