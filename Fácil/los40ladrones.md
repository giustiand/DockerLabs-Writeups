# Los40ladrones
**Herramientas y recursos usados**
- nmap
- gobuster
- knock
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
![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/los40ladrones/L40_1.jpg)   

Tenemos solamente un puerto abierto, el 80.  
Abrimos un explorador para ver que corre en esto puerto.  

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/los40ladrones/L40_2.jpg)   

Se nos abre la pagina por defecto de Apache.  
Mirando el código fuente no podemos sacar nada de interesante, por lo tanto haremos fuzzing web para ver que podemos encontrar.  

# Explotación

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/los40ladrones/L40_3.jpg)  

Hay un fichero interesante, qdefense.txt, vamos a ver el contenido.  

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/los40ladrones/L40_4.jpg)   

Por lo que podemos intuir "toctoc" podría ser un nombre de usuario y 7000, 8000 y 9000 posibles puerto abiertos.  
Vamos a escanear los puertos.  

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/los40ladrones/L40_5.jpg)  

Por lo que podemos ver no podemos sacar nada de interesante.  
Lo que me ocurre es "port knocking" que es una de las técnicas que se puede utilizar para securizar un servidor Linux.
Como funciona?  
Junto al uso de un servicio de cortafuegos por software, su principal función es evitar que los puertos queden expuestos, tanto a personas como a bots o escáneres automatizados de puertos.  
El usuario llama a una serie de puertos en un orden, por ejemplo al 2023, 2000, 9999 y el servicio de knockd habilitaría una regla en el firewall del sistema para permitir a la dirección IP solicitante el acceso al puerto o los puertos que hayamos configurado.

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/los40ladrones/L40_6.jpg)   

Ahora volvemos a hacer uno escaneos de los puertos para ver que sale.  

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/los40ladrones/L40_7.jpg)   

Ahora sí que aparece el puerto 22, así que podemos probar a hacer un ataque de fuerza bruta con el usuario "toctoc".  


Perfecto! Hemos encontrado la contraseña y podemos acceder por SSH.  

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/los40ladrones/L40_8.jpg)  

# Escalada de privilegios  

Una vez dentro probamos a ver si podemos abusar de algun binario para escalar a root.  
Para ello damos el comando `sudo -l` y vemos que hay un binario interesante en /opt.  

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/los40ladrones/L40_9.jpg)  

Así que si ejecutamos el comando `sudo /opt/bash`  

![L_40](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/los40ladrones/L40_10.jpg)   

Listo! Ya somos root!














