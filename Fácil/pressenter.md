# Pressenter

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

Solo tenemos un puerto abierto: el 80.  
Revisamos el sitio web para ver si encontramos algo útil.  
En la página en sí no veo nada interesante.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_2.jpg)   

Examinamos el código fuente.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_3.jpg)    

Como podemos ver, hay un enlace que apunta al dominio **pressenter.hl**.  
Añadimos, entonces, esta entrada en el archivo /etc/hosts y revisamos qué sucede.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_4.jpg)    

Bien, estamos frente a otro sitio web creado en WordPress.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_5.jpg)      

Entonces, procedemos a utilizar la herramienta wpscan para intentar enumerar usuarios, temas y plugins vulnerables.  
Para ello, utilizamos el siguiente comando:  

`sudo wpscan --url http://pressenter.hl --enumerate u,p,t`  

Tenemos dos usuarios, y lo que podemos hacer es un ataque de fuerza bruta para ver si logramos obtener una contraseña válida.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_6.jpg)      

Para ello, introducimos el siguiente comando:  

`sudo wpscan --url http://pressenter.hl/ --passwords /usr/share/wordlists/rockyou.txt --usernames pressi,hacker`  

Y después de esperar unos minutos, encontramos una contraseña.   

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_7.jpg)   

# Explotación

Entonces, iniciamos sesión en el panel wp-login.php.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_8.jpg)   

En el escaneo anterior de wpscan, hemos podido observar que el directorio está habilitado, es decir, podemos navegar a esta ruta:  

http://pressenter.hl/wp-content/themes/twentytwentyfour/  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_9.jpg)   

Por lo tanto, lo que intentaremos hacer es modificar el archivo **functions.php** para obtener una reverse shell.   
Para ello, vamos al menú 'Herramientas', luego a 'Editor de archivos del tema' y, por último, seleccionaremos el archivo **functions.php**.   

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_10.jpg)     

Lo actualizamos, y ahora solo tendremos que ponernos a la escucha en nuestra máquina Kali y abrir el archivo **functions.php**.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_11.jpg)       

Perfecto, ahora realizaremos un tratamiento de la shell para poder trabajar más cómodamente.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_12.jpg)     

# Escalada de privilegios  

Probamos con los típicos comandos `sudo -l` y `find / -perm -4000 2>/dev/null` pero no encontramos nada interesante.   

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_13.jpg)  

Intentamos subir a la máquina víctima pspy64, pero no obtuvimos nada interesante.   
Entonces, probamos a subir linpeas y aquí sí encontramos información valiosa.   

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_14.jpg)    

Con las credenciales obtenidas, iniciamos sesión en SQL y verificamos si podemos obtener algo útil.  

`mysql -u admin -prooteable -h 127.0.0.1`  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_15.jpg)     

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_16.jpg)     

¡Bien!  
Hemos logrado obtener la contraseña de un usuario llamado **enter**.  
Vamos a comprobar si en la máquina hay algún usuario con este nombre, ¡y efectivamente hay!  
Entonces, probamos a ver si podemos cambiar de usuario.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_17.jpg)      

¡Perfecto!  
Ejecutamos `sudo -l` y encontramos información que nos será útil para escalar a root.   

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_18.jpg)  

Si ejecutamos `ls` como usuario enter, vemos que hay un archivo llamado **user.txt**.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_19.jpg)    

Ahora que sabemos que podemos utilizar cat con privilegios de root, lo que podemos intentar hacer es verificar si existe un archivo llamado **root.txt** en el directorio home de root y leer su contenido.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_20.jpg)     

Esto no nos sirve.  
Después de reflexionar y no encontrar realmente ninguna forma de elevar privilegios, como última opción, casi sin creerlo, pruebo a utilizar la contraseña de enter... ¡y listo!  
¡Somos root!  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/pressenter/P_21.jpg)      

















