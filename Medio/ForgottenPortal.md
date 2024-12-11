# ForgottenPortal

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN FP`  

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

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_1.png)     

Tenemos dos puertos abiertos, el 22 y el 80.  
Echemos un vistazo para ver qué está corriendo en el puerto 80.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_2.png)    

Aparentemente no encontramos nada interesante, pero si analizamos el código fuente, nos llama la atención este comentario.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_3.png)    

Echemos entonces un vistazo a la ruta **m4ch1n3_upload.html**.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_4.png)   

Bien, por lo que podemos observar, somos capaces de cargar archivos con extensión **.php**.  
Por lo tanto, lo que haremos será cargar una reverse shell para ver si podemos obtener acceso a la máquina víctima.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_5.png)   

Perfecto, ahora nos pondremos en escucha en el puerto 6969 (que es el que hemos configurado en nuestra reverse shell) y ejecutaremos el archivo para ver si obtenemos una shell.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_6.png)  

¡Perfecto!  
¡Estamos dentro!  

# Escalada de privilegios  

Una vez dentro, como siempre, lo primero que haremos será estabilizar la shell para trabajar de manera más cómoda.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_7.png)   

Ejecutamos los comandos clásicos `sudo -l` y `find / -perm -4000 2>/dev/null`, pero no obtenemos nada interesante.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_8.png)   

Intentamos, después de haber montado un servidor HTTP con Python, transferir **linpeas** y **pspy64** a la máquina víctima, pero no obtenemos nada.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_9.png)   

Por lo tanto, lo que haremos será analizar dentro de la carpeta html para ver si encontramos algo interesante.  
Vemos que hay un archivo llamado **access.log**.   

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_10.png)  

Echemos un vistazo.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_11.png)    

Nos llama la atención esta línea, donde aparece una cadena que parece estar codificada en base64.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_12.png)  

¡Excelente! Hemos obtenido un nombre de usuario y una contraseña.  
Ahora intentemos escalar de usuario.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_13.png)    

¡Perfecto!  
Nos movemos a la carpeta home para ver si encontramos algún archivo interesante.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_14.png)    

Vemos que hay una carpeta llamada **incidents** que contiene un archivo llamado **report**.   
Echemos un vistazo y descubrimos algo muy interesante.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_15.png)    

Por lo que podemos entender, descubrimos que dentro de nuestra carpeta **/home/alice/.ssh** hay una clave **id_rsa** que podemos usar para conectarnos a través de SSH con el usuario **bob**, utilizando como contraseña **cyb3r_s3curity**.   
Veamos si funciona.  
Lo primero que haremos es verificar que efectivamente exista esta clave.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_16.png)     

Bien, ahora la copiaremos a nuestra máquina Kali, le asignaremos los permisos correctos y probaremos conectarnos mediante SSH.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_17.png)       

¡Excelente, ha funcionado! Ahora ejecutaremos el comando `sudo -l` para ver si somos capaces de escalar a root.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_18.png)   

Como de costumbre, recurrimos a la web de GTFOBins (https://gtfobins.github.io/gtfobins/tar/#sudo) para ver, en este caso, cómo podemos abusar del binario **tar**.  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_19.png)     

Ejecutamos el comando sugerido y...  

![F](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/ForgottenPortal/FP_20.png)     

¡Listo! 
¡Ya somos root!



