# Fase 1- Tanteo
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/ecd69669-9859-4e1e-9aa6-d9d01014c9da)
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
3. **80/tcp(HTTP):** Apache httpd 2.4.59

### Puerto 80
Al identificar que el puerto 80 estaba abierto durante el escaneo, se procedió a ingresar la dirección IP en el navegador. Se encontró una pagina de pizzas.
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/72f68755-c15a-458b-b971-3d7153111cad)

# Fase 2- Exploración

#### Codigo
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/5700a8c8-9b6c-4667-8ee0-f89f7c6e0e73)
Revisando el codigo encontramos que en la linea 85 tiene un mensaje que dice:
```
<!-- Puedes creer que hay fanáticos de la pizza de piña que se ponen de usuario pizzapiña -->
```
Tenemos usuario para realizarle fuerza bruta.

# Fase 3- Explotación

#### Hydra
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/e49c85a8-0cb0-4736-b44a-368455dc862a)
Realizamos fuerza bruta al protocolo `ssh` al usuario `pizzapiña` y encontramos su contraseña: `steven`. Comando utilizado:
```python
hydra -l pizzapiña -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.4
```

# Fase 4- Privilegios

#### Sudo -l
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/7bd5ff9a-ced7-4098-a256-054099983fd0)

Despues de realizar el tratamiento de la TTY, vemos que tenemos permisos de ejecutar el binario `gcc` como usuario el `pizzasinpiña`.

#### Gcc
Para escalar utilizando este binario tenemos que ejecutar el siguiente comando:
```python
sudo -u pizzasinpiña gcc -wrapper /bin/sh,-s .
```

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/e10e3455-028a-4812-b2d3-73b2d1f214f8)

Y ya somos el usuario `pizzasinpiña`.

#### Pizzassinpiña
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/7b09f32e-02cb-4e30-ad99-0c9ef3c265a7)

Como el usuario `pizzasinpiña` podemos ejecutar el binario `man` como el usuario `root`.

#### Man
Para escalar tenemos que ejecutar el comando:
```
> sudo man man
```

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/a77635b2-7075-4d14-a11f-54d5394d54be)

Una vez aquí, tenemos que ejecutar el comando:
```
!/bin/sh
```

Así debería de quedar:
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/3d93396e-d658-49d3-a2fb-55e114a70439)

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/0b484a06-9131-48ed-a967-ffad0b4feb69)

Y listo! somos root!.
