# Amor

# Enumeración

Empezamos con un escaneo de puertos.  

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

Solo tenemos dos puertos abiertos, el 22 y el 80.  
Vamos a revisar qué encontramos en el puerto 80.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_2.jpg)   

Observamos que hay una usuaria de la empresa llamada carlota, por lo tanto, podemos intentar un ataque de fuerza bruta contra SSH.  
Ejecutaremos el siguiente comando:  

`sudo hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t64`  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_3.jpg)     

¡Perfecto!  
Ya tenemos una contraseña válida y hemos iniciado sesión por SSH.   
Una vez dentro, ejecutamos el comando `sudo -l`, pero vemos que no obtenemos ningún resultado útil.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_4.jpg)   

Lo mismo nos sucede al ejecutar el comando `find / -perm -4000 2>/dev/null`, ya que no obtenemos ningún resultado relevante.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_5.jpg)     

Al revisar un poco en el directorio del usuario, encontramos una carpeta llamada **fotos** que contiene una imagen.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_6.jpg)     

Nos descargamos la imagen con el siguiente comando:  

`scp -rpC carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg /home/kali/Downloads`  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_7.jpg)    

Abrimos la imagen, pero aparentemente no contiene nada interesante.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_8.jpg)     

Entonces utilizamos la herramienta **exiftool** para inspeccionar los metadatos de la imagen, pero no parece haber nada útil que podamos aprovechar.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_9.jpg)     

Entonces probamos con la herramienta steghide.  
Ejecutamos el siguiente comando:  

`sudo steghide extract -sf imagen.jpg`  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_10.jpg)   

Vemos que se ha extraído un archivo llamado **secret.txt**.   
Al abrirlo, parece que contiene una contraseña codificada en base64.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_11.jpg)     

Por lo tanto, la decodificamos con el siguiente comando:  

`echo 'ZXNsYWNhc2FkZXBpbnlwb24=' |base64 -d`  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_12.jpg)  

¡Perfecto!   
Ahora revisamos el archivo **/etc/passwd** para ver si hay algún otro usuario además de **carlota**.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_13.jpg)  

¡Bien!   
Probamos la contraseña recién descubierta con el usuario **oscar**, ¡y funciona!  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_14.jpg)    

Ahora ejecutamos el comando `sudo -l`.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_15.jpg)      

Bien, revisamos la web de GTFOBins para ver si y cómo podemos abusar de este binario para escalar privilegios.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_16.jpg)       

Entonces ejecutaremos el siguiente comando:  

`sudo ruby -e 'exec "/bin/sh"'` ¡y listo!  

¡Ya somos root!  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/amor/A_17.jpg)         













