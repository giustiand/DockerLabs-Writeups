# Trust
**Herramientas y recursos usados**
- nmap
- gobuster
- hydra

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.19.0.2 -oN Trust`

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
![T_1](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Trust/T_1.jpg)  

Como podemos ver tenemos 2 puertos abiertos, el puerto 22 que corresponde al servicio SSH y el puerto 80 que corresponde a HTTP.  
Probamos a ver que aparece si abrimos un explorador a la dirección http://172.19.0.2  

![T_2](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Trust/T_2.jpg)  

Se nos abre la pagina por defecto de Apache.  
Vamos a ver con el comando `Ctrl + U` si podemos sacar algunas informaciones más pero no aparece nada interesante por aquí.  
Intentaremos por lo tanto hacer fuzzing web con la herramienta `gobuster`.  

![T_3](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Trust/T_3.jpg)  

Como podemos ver hay algo interesante por aqui, la pagina `secret.php`.  

![T_4](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Trust/T_4.jpg) 

Se abre esta web que a primera vista no contiene nada interesante, a parte el nombre "Mario". 
Vamos a ver el codigo fuente para ver si podemos sacar algo más.  

![T_5](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Trust/T_5.jpg)

Tampoco nada interesante por aqui.  
Lo que podemos hacer es utilizar `hydra` para hacer un ataque de fuerza bruta al puerto SSH con el usuario Mario.  

# Explotación
![T_6](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Trust/T_6.jpg)  

Perfecto! Ha funcionado!  
Ahora accedemos a SSH con el usario Mario e intentaremos escalar privilegios.  

# Escalada de privilegios

Una vez dentro primero de todo lanzaremos el comando `sudo -l` para ver si podemos aprovechar algun binario para escalar a root.  

![T_7](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Trust/T_7.jpg)  

Como podemos ver podemos aprovechar el file vim y en la web GTFOBins encontraremos el comando que podemos utilizar.

![T_8](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Trust/T_8.jpg)   

TIPS
Es conveniente usar siempre el percorso absoluto del binario, es decir, lanzar el comando poniendo /usr/bin/vim/

![T_9](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Trust/T_9.jpg)  

Y listo! Ya somos root!

