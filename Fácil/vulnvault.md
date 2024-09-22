# VulnVault
**Herramientas y recursos usados**  
- nmap 
- gobuster  
- knock  
- hydra

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














