# BuscaLove

# Enumeración

Empezamos con un escaneo de puertos.  

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.18.0.2 -oN BuscaLove`  

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

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_1.jpg)    

Tenemos dos puertos abiertos, el 22 y el 80.  
Echemos un vistazo a ver qué corre en el puerto 80.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_2.jpg)   

A primera vista, nada interesante.   
Por lo tanto, lo que haremos será hacer un poco de fuzzing para ver si descubrimos algunos recursos más.  
Para ello, ejecutamos el comando:  

`sudo gobuster dir -u http://172.18.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_3.jpg)    

Bien, descubrimos un nuevo recurso llamado **wordpress**.  
Echamos un vistazo para ver qué hay.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_4.jpg)    

La página en sí no contiene nada interesante, pero si miramos el código fuente, podemos encontrar una pista.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_5.jpg)     

Volvemos a hacer un poco de fuzzing sobre este nuevo recurso.  
Ejecutamos el comando:  

`sudo gobuster dir -u http://172.18.0.2/wordpress -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x htm,html,php,txt`  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_6.jpg)      

Bueno, encontramos solo el archivo **index.php**.  
Después de reflexionar un rato, lo que se me ocurre es intentar hacer fuzzing con la herramienta **wfuzz** en busca de una posible vulnerabilidad de LFI (Local File Inclusion).  
Para ello ejecutamos el comando:  

`wfuzz -c --hl=40 -w /usr/share/wordlists/dirb/common.txt http://172.18.0.2/wordpress/index.php?FUZZ=../../../../../../../etc/passwd`  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_7.jpg)    

¡Excelente!   
Estamos frente a un sitio vulnerable a LFI.  
Entonces, si ahora digitamos http://172.18.0.2/wordpress/index.php?love=../../../../../../etc/passwd, deberíamos ver los usuarios que están en el sistema.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_8.jpg)   

¡Bien!
Ahora, lo que podemos hacer es crear un archivo llamado **users.txt** que contenga los nombres de usuario *pedro* y *rosa*, que acabamos de descubrir, y utilizar la herramienta **hydra** para realizar un ataque de fuerza bruta a SSH.  
Ejecutaremos el comando:  

`sudo hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2 -t 64`  

Después de esperar varios minutos, hemos logrado descubrir la contraseña de *rosa*.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_9.jpg)    

Nos conectamos por SSH y estamos dentro de la máquina víctima.  

# Escalada de privilegios  

Una vez dentro, como de costumbre, ejecutamos el comando `sudo -l` para ver si podemos abusar de algún binario.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_10.jpg)    

Bien, podemos ejecutar **ls** y **cat** como usuario root. Probemos a ver el contenido de la carpeta *home* de **pedro**.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_11.jpg)  

Nada interesante.  
Hagamos lo mismo, pero esta vez en la carpeta *home* de root.    

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_12.jpg)    

Aquí sí que nos llama la atención un archivo llamado **secret.txt**.  
Lo abrimos con *cat*, ya que también podemos ejecutarlo como root.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_13.jpg)    

Nos encontramos frente a una cadena hexadecimal.  
Decodificamos la cadena utilizando la web [https://www.convertstring.com/es/EncodeDecode/HexDecode](https://www.convertstring.com/es/EncodeDecode/HexDecode).  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_14.jpg)  

Intentamos iniciar sesión con esta cadena tanto como el usuario *pedro* como el usuario *root*, pero no funciona.  
Después de intentar, descubrimos que esta cadena está en base32.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_15.jpg)    

Ahora sí que podemos iniciar sesión con el usuario *pedro*.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_16.jpg)    

Una vez dentro, ejecutamos el comando `sudo -l`.  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_17.jpg)     

Consultamos la web de GTFOBins [https://gtfobins.github.io/gtfobins/env/#sudo](https://gtfobins.github.io/gtfobins/env/#sudo) para ver cómo podemos abusar de este binario. 

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_18.jpg)      

¡Simplemente ejecutamos el comando `sudo env /bin/bash` y listo!  
¡Somos root!  

![B](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/buscalove/B_19.jpg)    






