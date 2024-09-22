# VulnVault
**Herramientas y recursos usados**  
- nmap 
- gobuster  


# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN VulnVault`  

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
![VV_1](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_1.jpg)  

Tenemos 2 puertos abiertos, el 22 y el 80. 
Abrimos el explorador para ver que hay en el puerto 80. 

![VV_2](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_2.jpg)  

Puede llamarnos la atencion el menu que pone "subir archivos".   
Probamos a ver si podemos subir un archivo .php para obtener una reverse shell.   
Preparamos entonces un fichero .php y probamos a cargarlo.  

![VV_3](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_3.jpg)   

Perfecto!  
El archivo se ha cargado exitosamente.   
Ahora tenemos que hacer fuzzing web a la busqueda de una carpeta upload o algo así donde podría estar cargado nuestro fichero para poder usarlo para ganar una shell.   

![VV_4](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_4.jpg)     

Abrimos la página "upload.php" para ver si allí vemos el fichero, pero no hay nada.   

![VV_5](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_5.jpg)    

Volvemos entonces a la página de inicio para ver si podemos encontrar alguna pista.  
Vemos que hay una frase que habla de comandos maliciosos.  

![VV_6](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_6.jpg)     

Tras probar varias maneras para escapar los comandos podemos apreciar que si ponemos por ejemplo `; cat /etc/passwd` podemos obtener el listado de todos los usuarios de la maquina.  

![VV_7](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_7.jpg)    

![VV_8](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_8.jpg)   

Perfecto!  
Ahora sabeos que hay un usuario que se llama **samara** y podemos probar un ataque de fuerza bruta con hydra.  

![VV_9](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_9.jpg)     

Tras esperar un largo rato vemos que no podemos sacar la password.  
Lo que podemos probar a hacer es ver si hay la llave privada del usuario en la carpeta .ssh.  

![VV_10](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_10.jpg)      

![VV_11](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_11.jpg)    

Ahora abrimos el fichero **id_rsa** y lo copiamos.  

![VV_12](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_12.jpg)     

![VV_13](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_13.jpg)      

# Explotación  

Una vez copiado el contenido del fichero id_rsa en nuestra maquina local le vamos a dar permiso 600 y nos conectamos a la maquina victima.  

![VV_14](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_14.jpg)      

![VV_15](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_15.jpg)    

# Escalada de privilegios    

Vemos que si le damos al comando `sudo -l` nos dice que no encuentra el comando sudo.  

![VV_16](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_16.jpg)   

Revisamos permisos SUID  y podemos observar que no tenemos nada que nos pueda ser útil.  

![VV_17](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_17.jpg)     

Entonces lo que haremos será subir **pspy** a la máquina victima para ver si podemos obtener algo util.  
Creamos una carpeta `tmp` en nuestra máquina Kali y descargamos el fichero en la maquina victima.  

![VV_18](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_18.jpg)    

![VV_19](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_19.jpg)  

Ahora le damos los permisos de ejecución y lo ejecutamos.  

![VV_20](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_20.jpg)   

Notamos que hay un fichero nombrado "echo.sh" que se ejecuta ad intervalos regulares.  
Modificamos su contendio y le añadimos este comando para que nos ejecute una bash con permisos de root.  

![VV_21](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_21.jpg)     

Ahora solo tendremos que esperar un istante y si le damos al comando `bash -p` ya podremos ver que seremos root!  

![VV_22](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/vulnvault/VV_22.jpg)      













































