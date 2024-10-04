# Pressenter
**Herramientas y recursos usados**  
- zip2john
- john the ripper


# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Pressenter`  

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

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/Pressenter/P_1.jpg)   

Tenemos un solo puerto abierto, el 80.  
Miramos la web a ver si encontramos algo util.  
En la página como tal no veo nada interesante.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/Pressenter/P_2.jpg)   

Echamos un ojo al código fuente.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/Pressenter/P_3.jpg)    

Como podmeos ver hay un enlace que punta al dominio **pressenter.hl**.  
Añnadimos entonces esta entrada en el fichero /etc/hosts y miramos a ver que sale.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/Pressenter/P_4.jpg)    

Buenos, estamos frente a otra web hecha en Wordpress.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/Pressenter/P_5.jpg)      

Entonces prodamos a usar la herramiente **wpscan** para intentar enumerar usuarios, tema y plugin vulnerables.  
Para ello utilizamos el comando:  

`sudo wpscan --url http://pressenter.hl --enumerate u,p,t`  

Tenemos 2 usuarios, y lo que podemos hacer es un ataque de fuerza bruta para ver si logramos obtener una contraseña valida.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/Pressenter/P_6.jpg)      

Para ello damos el comando:  

`sudo wpscan --url http://pressenter.hl/ --passwords /usr/share/wordlists/rockyou.txt --usernames pressi,hacker`  

Y tras esperar unos minutos encontramos una contraseña.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/Pressenter/P_7.jpg)      

Entonces nos logeamos en el panel wp-login.php.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/Pressenter/P_8.jpg)  







