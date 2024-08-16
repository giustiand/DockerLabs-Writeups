# Injection
**Herramientas y recursos usados**
- nmap
- searchsploit
- hydra

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Injection`

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
![I_1](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Injection/I_1.jpg)   

Vemos que hay dos puertos abiertos, el 22 y el 80.  
Abrimos un explorador para ver que hay en el puerto 80.  

![I_2](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Injection/I_2.jpg)  

Tenemos este panel de login, y podemos probar con credeciales por defecto como admin, password etc. pero en este caso en concreto no nos sirven.  
Intentamos a ver si con el fuzzing web logramos descubrir algo más.  

![I_3](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Injection/I_3.jpg)  

No sacamos muchas cosas la verdad.  
Volvemos al panel de login e intentamos a bypassar la autenticación con la más clasica de la SQL injection, es decir digitando `' OR '1'='1`.  

![I_4](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Injection/I_4.jpg)  

Perfecto, ya tenemos un usuario "dylan" y una contraseña "KJSDFG789FGSDF78".  

# Explotación
Probamos a utilizarlos para acceder a SSH.  

![I_5](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Injection/I_5.jpg)  






