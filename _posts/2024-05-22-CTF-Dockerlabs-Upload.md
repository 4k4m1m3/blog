---
title: CTF-DockerLabs-Upload
published: true
---

## WriteUps maquina Upload de DockerLabs

---
tags:
  - CTF
  - estado/completado
  - plataforma: [DockerLabs]
  - dificultad: Muy fácil
  - autor: [ElPinguinoDeMario]

# Datos

[!INFO] Upload
  -  **Nombre:** Upload
  -  **SO:** Linux
  -  **Dificultad:** Muy fácil
  -  **Enlace:** [Dockerlabs](https://dockerlabs.es/)

[!TODO] Objetivo
  - 🚩Ingresar a la maquina como algún usuario.
  - 🚩Elevar privilegios una vez obtenido el acceso.

El primer paso consiste en iniciar la máquina, lo cual es tan sencillo como ejecutar el siguiente comando después de haber descargado la maquina:

```bash
└─$ sudo bash auto_deploy.sh upload.tar
```

# Reconocimiento

  - Una vez iniciada la maquina, el mismo script de inicio me da la dirección IP a lo cual procedo a realizar un escaneo de puertos de la maquina y el resultado es el siguiente:

```bash
└─# nmap 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-22 09:49 EDT
Nmap scan report for 172.17.0.2
Host is up (0.0000070s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.25 seconds
```

Al darme cuenta que existe el `puerto 80` abierto, procedo a visitar desde el navegador la dirección IP de la maquina y encuentro con una formulario para cargar archivos (tiene lógica con el nombre de la maquina), además para no dejar que se me pueda escapar algo, procedo a mirar su código fuente y no encuentro nada fuera de la común. 

# Upload file

  - Una de las primeras cosas que observo es que no existe un filtrado de la extensión del archivo a cargar, así que por lógica permite cargar cualquier archivo sin importar la extensión, podría subir por ejemplo el `pentestmonkey` en php y abrirlo desde el navegador para obtener una reverse shell.
  -  - 
  -  - #pentestmonkey
  -  - Esta herramienta es útil durante pruebas de seguridad en las que tienes acceso para subir archivos a un servidor web con PHP. Subes el script a la raíz del sitio web y luego lo ejecutas desde tu navegador. El script crea una conexión desde el servidor web hacia una dirección y puerto que elijas, permitiéndote controlar el servidor a través de una terminal.
  - 
  - Así que procedo a modificar y subir un archivo php de pentestmonkey, sin embargo necesito ahora conocer el directorio donde se suben estos archivos, para ello uso `dirbuster` y continuando con el nombre de la maquina, el directorio se llama: `uploads`

Así que ingresando desde al navegador a la dirección: `http://172.17.0.2/uploads/pentestmonkey.php` logro obtener una reverse shell, sin olvidar que tengo que ponerme a la escucha con netcat y el comando: `nc -lnvp 4444`

![UploadNCdockerlabs.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/UploadNCdockerlabs.png)

Con esto logramos acceder a la maquina con el usuario `www-data`, ahora tocar elevar privilegios o pivotear a otro usuario.

### Pivotear a otro usuario

Para conocer los usuarios que se encuentran en el sistema utilizo: `compgen -u` a lo que me doy cuenta que no existe otro usuario, por lo tanto directamente desde este usuario toca elevar privilegios.

# Escalada de privilegios

  - Ahora que tengo acceso con el usuario `www-data` y verificando que no existe otro usuario en el sistema para pivotear, procedo a escalar privilegios y para ello coloco primeramente: `sudo -l` lo que me da como resultado:

```bash
www-data@8792622e94b1:/$ sudo -l
sudo -l
Matching Defaults entries for www-data on 8792622e94b1:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User www-data may run the following commands on 8792622e94b1:
    (root) NOPASSWD: /usr/bin/env
```

Así que procedo ir a la web [GTFOBins](https://gtfobins.github.io/) para ver como elevar privilegios utilizando `env` con `sudo` y el resultado es que si coloco el siguiente comando `sudo env /bin/sh` con la palabra `sudo` lograría ingresar con el usuario root.

![pwnedCTFdockerlabsUpload.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/pwnedCTFdockerlabsUpload.png)

# Comandos

[!IMPORTANT] Resumen de comandos utilizados
  - `sudo bash auto_deploy.sh upload.tar`
  - `nmap 172.17.0.2`
  - `dirbuster`
  - `nc -lnvp 4444`
  - `sudo vim -c ':!/bin/sh'`


```
¡Que la fuerza del hacking ético nos acompañe! :)
```

|   |
|:--|
|   |