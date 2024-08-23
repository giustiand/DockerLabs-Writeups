# Injection
**Herramientas y recursos usados**
- nmap
- gobuster

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

Perfecto, ya tenemos un usuario "dylan" y una contraseña.  

# Explotación
Probamos a utilizarlos para acceder a SSH.  

![I_5](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Injection/I_5.jpg)  

Ya hemos tenido acceso, ahora tenemos que intentar que escalar privilegios.  

# Escalada de privilegios
Probamos con el comando `sudo -l` para ver que podemos hacer.  
 
![I_6](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Injection/I_6.jpg)   

Parece que por ahí no van los tiros.  
Buscamos a ver si podemos aprovechar de algun binario para elevar nuestros privilegios.

![I_7](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Injection/I_7.jpg)  

Nos llama la atención el binario `env`, buscamos entonces en GTFOBins si hay alguna manera para poder escalar a root.  

![I_8](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Injection/I_8.jpg)   

Perfecto, parece que sí!  
Entonces digitamos el comando y  

![I_9](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Injection/I_9.jpg)   

Listo! Ya somos root!







