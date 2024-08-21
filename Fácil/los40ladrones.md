# Los40ladrones
**Herramientas y recursos usados**
- nmap
- searchsploit
- hydra

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Los40ladrones`

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
![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/L40_1.jpg)   

Tenemos solamente un puerto abierto, el 80.  
Abrimos un explorador para ver que corre en esto puerto.  

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/L40_2.jpg)   

Se nos abre la pagina por defecto de Apache.  
Mirando el código fuente no podemos sacar nada de interesante, por lo tanto haremos fuzzing web para que podemos encontrar.  

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/L40_3.jpg)   



