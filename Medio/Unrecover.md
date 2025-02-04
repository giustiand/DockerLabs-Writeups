# Unrecover  

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN UnRecover`  

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

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_1.png)     

Tenemos 4 puertos abiertos: el 21, el 22, el 80 y el 3306.

Echemos un vistazo a la web.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_2.png)    

A primera vista no vemos nada interesante, aparte de un posible nombre de usuario **capybara**.    
Lo que podemos hacer es intentar realizar un ataque de fuerza bruta contra FTP, SSH y MySQL con el usuario que acabamos de descubrir.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_3.png)    

¡Perfecto!   
Obtenemos credenciales válidas para el acceso a SQL.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_4.png)   

Ahora echamos un vistazo a las bases de datos para ver si hay información interesante.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_5.png)      

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_6.png)     

Bien, hemos obtenido otro usuario y otra contraseña.  

# Explotación  

Intentamos acceder a través de SSH con el usuario baluleroh y la contraseña que acabamos de descubrir, pero no podemos.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_7.png)      

¿Y si intentamos con la contraseña de capybara?  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_8.png)      

¡Perfecto!  
Estamos dentro.   

Vemos que en el directorio home del usuario, dentro de la carpeta *server*, hay un archivo denominado **backup.pdf**.

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_9.png)        

Lo transferimos a nuestra máquina para ver su contenido.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_10.png)    

Abrimos el archivo y vemos que contiene la contraseña de root un poco difuminada.   

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_11.png) 

Haciendo simplemente un poco de zoom, podemos ver que la clave es **passwordpepinaca**.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_12.png)   

Vamos a ver si funciona.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Unrecover/U_13.png)     

¡Y listo!  
¡ Ya somos root!  






