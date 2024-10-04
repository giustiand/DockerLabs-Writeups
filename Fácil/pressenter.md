# Pressenter
**Herramientas y recursos usados**  



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

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_1.jpg)   

Tenemos un solo puerto abierto, el 80.  
Miramos la web a ver si encontramos algo util.  
En la página como tal no veo nada interesante.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_2.jpg)   

Echamos un ojo al código fuente.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_3.jpg)    

Como podmeos ver hay un enlace que punta al dominio **pressenter.hl**.  
Añnadimos entonces esta entrada en el fichero /etc/hosts y miramos a ver que sale.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_4.jpg)    

Buenos, estamos frente a otra web hecha en Wordpress.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_5.jpg)      

Entonces prodamos a usar la herramiente **wpscan** para intentar enumerar usuarios, tema y plugin vulnerables.  
Para ello utilizamos el comando:  

`sudo wpscan --url http://pressenter.hl --enumerate u,p,t`  

Tenemos 2 usuarios, y lo que podemos hacer es un ataque de fuerza bruta para ver si logramos obtener una contraseña valida.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_6.jpg)      

Para ello damos el comando:  

`sudo wpscan --url http://pressenter.hl/ --passwords /usr/share/wordlists/rockyou.txt --usernames pressi,hacker`  

Y tras esperar unos minutos encontramos una contraseña.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_7.jpg)   

# Explotación

Entonces nos logeamos en el panel wp-login.php.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_8.jpg)   

En la scansión anterior de **wpscan** hemos podido apreciar que el directory listing está habilitado, es decir, podemos navegar a esta ruta.  

http://pressenter.hl/wp-content/themes/twentytwentyfour/  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_9.jpg)   

Por lo tanto lo que intentaremos hacer es modificar el fichero **funcion.php** para obtener una reverse shell.  
Para ello vamos al menu "Herramientas" luego a "Editor de archivos de tema" y por ultimo seleccionaremos el fichero **function.php**.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_10.jpg)     

Lo actualizamos, y ahora solo tendremos que ponernos a la escucha en la nuestra maquina kali y abrir el fichero function.php.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_11.jpg)       

Perfecto, ahora haremos un tratamiento de la shell para poder trabajar más comodamente.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_12.jpg)     

# Escalada de privilegios  

Probamos con los típicos comandos `sudo -l` y `find / -perm -4000 2>/dev/null` pero no vemos nada interesante.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_13.jpg)  

Probamos a subir a la maquina victima **pspy64** pero no obtenemos nada interesante.  
Entonces probamos a subir **linpeas** y aquí sí que encontramos información valiosa.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_14.jpg)    

Con las credenciales obtenidas nos logeamos a sql y vemos si podemos sacar algo util.  

`mysql -u admin -prooteable -h 127.0.0.1`  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_15.jpg)     

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_16.jpg)     

Bien!  
Hemos podido sacar la password de un usuario que se llama **enter**.  
Vamos a comprobar si en la maquina hay algun usuario con este nombre y sí que hay!  
Entonces probamos a ver si podemos cambiar de usuario.   

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_17.jpg)      

Perfecto!  
Hacemos un `sudo -l` y vemos cosas que nos vendrán bien para escalarnos a root.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_18.jpg)     
















