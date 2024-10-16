# Find your style

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN FindYourStyle`  

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

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_1.jpg)     

Tenemos solamente un puerto abierto, el 80 y por lo que podemos ver estamos frente a un Drupal 8.   

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_2.jpg)   

Lo que podemos hacer es mirar si hay algun exploit que podemos lanzar.  
Abrimos Metasploit y buscaremos con el comando `search drupal`.   

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_3.jpg)     

Usaremos el exploit 16, el drupalgeddon.   
Vamos a cumplimenter las  opciones y ejecutaremos el exploit.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/findyourstyle/F_4.jpg)   







