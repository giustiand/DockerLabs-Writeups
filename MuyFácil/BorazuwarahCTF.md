# BorazuwarahCTF
**Herramientas y recursos usados**
- nmap
- steghide
- exiftool
- hydra

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN BorazuwarahCTF`

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
![B_1](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BorazuwarahCTF/B_1.jpg)  

Como podemos ver tenemos 2 puertos abiertos, el puerto 22 que corresponde al servicio SSH y el puerto 80 que corresponde a HTTP.  
Probamos a ver que aparece si abrimos un explorador a la dirección http://172.17.0.2  

![B_2](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BorazuwarahCTF/B_2.jpg)  

Aparece esta imagen y al parecer nada más, tampoco en el código fuente de la página.  
Descargamos la imagen y probamos a ver si tiene alguna información util al suo interno.  
Para ello utilizaremos la herramienta `steghide`.  

![B_3](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BorazuwarahCTF/B_3.jpg)  

Vemos que nos extrae un fichero nombrado "secreto.txt".  
Vamos a ver su contenido.  

![B_4](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BorazuwarahCTF/B_3.jpg)   

Tenemos la pista de "seguir buscando en la imagen".  
Podemos utilizar otra herramienta nombrada `exiftool`.  

![B_5](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BorazuwarahCTF/B_5.jpg)   

Aquí si que tenemos una información interesante, o sea el nombre de usuario "borazuwarah".

# Explotación
Utilizaremos la herramienta `hydra` para hacer una ataque de fuerza bruta contra SSH.  

![B_6](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BorazuwarahCTF/B_6.jpg)  

Perfecto!  
Ahora podemos logearnos e intentar escalar los privilegios.  

# Escalada de privilegios
Vamos a dar el comando `sudo -l` para ver si podemos hacer algo.  

![B_7](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BorazuwarahCTF/B_7.jpg)   

Miramos en la web de GTFOBins y encontramos este comando.

![B_8](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BorazuwarahCTF/B_8.jpg) 

Lo ejecutamos  

![B_9](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/BorazuwarahCTF/B_9.jpg)   

Y listo! Ya somos root!





 
