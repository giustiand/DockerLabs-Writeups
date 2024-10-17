# Escolares

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Escolares`  

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

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_1.jpg)     

Tenemos dos puertos abiertos: el 22 y el 80.    
Veamos qué hay en el puerto 80.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_2.jpg)     

A primera vista, no hay nada interesante.    
Echamos un vistazo al código fuente.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_3.jpg)  

Hay un comentario que debería llamar nuestra atención.    
Miramos la página **profesores.html** y vemos que hay un usuario, que al parecer se llama Luis, que es admin de WordPress.   

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_4.jpg)    

Hacemos entonces un poco de fuzzing web para ver si podemos obtener algo útil.  

`sudo gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x bak,html,php,txt`  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_5.jpg)   

Ahora miramos la ruta http://172.17.0.2/wordpress.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_6.jpg)    

Efectivamente, nos encontramos frente a un WordPress.    
Al mirar el código fuente, vemos que hay varios enlaces a un dominio llamado **escolares.dl**.   

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_7.jpg)    

Lo que haremos ahora será modificar el archivo **hosts** añadiendo este nuevo dominio.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_8.jpg)    

Ahora volvemos a hacer fuzzing web para ver si obtenemos más recursos.    

`sudo gobuster dir -u http://escolares.dl/wordpress -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x bak,html,php,txt`

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_9.jpg)   

Ahora deberíamos intentar enumerar los usuarios para intentar hacer un ataque de fuerza bruta.    
Para ello, usaremos la herramienta **wpscan**.   
Ejecutaremos el comando:   

`sudo wpscan --url http://escolares.dl/wordpress -e u,p,t`  

Tras esperar un rato, vemos que nos proporciona un nombre de usuario: **luisillo**.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_10.jpg)   

Ahora intentaremos realizar un ataque de fuerza bruta utilizando el diccionario **rockyou.txt**.  

`sudo wpscan --url http://escolares.dl/wordpress --usernames luisillo --passwords /usr/share/wordlists/rockyou.txt`  

Esperamos un largo rato, pero no logramos obtener una combinación válida.    
Entonces, lo que haremos será crear nuestro propio diccionario personalizado utilizando la información que podemos ver en la página de profesores.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_11.jpg)     

Ejecutarmos el comando `cupp -i`  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_12.jpg)      

Ahora volveremos a utilizar **wpscan** para realizar un ataque de fuerza bruta con nuestro diccionario recién creado.   

`sudo wpscan --url http://escolares.dl/wordpress --usernames luisillo --passwords luis.txt`  

¡Perfecto!    
Tenemos una combinación válida.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_13.jpg)      

Por lo tanto, iniciamos sesión.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_14.jpg)     

Una una vez dentro, buscamos alguna manera de enviarnos una reverse shell a nuestra máquina Kali.    
Vemos que hay la posibilidad de subir archivos a través del **WP File Manager**, por lo tanto, lo que haremos será subir nuestra reverse shell.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_15.jpg)     

Una vez hecho, nos pondremos a la escucha en nuestra máquina atacante y ¡ya estaremos dentro!  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_16.jpg)   

Bien, ahora intentamos escalar privilegios.    
Vemos que con los comandos típicos no obtenemos nada interesante.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_17.jpg)   

Entonces, probamos a buscar algún archivo .txt con el comando:  

`find / -type f -name *.txt 2>/dev/null`  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_18.jpg)  

Tenemos 2 archivos llamados **secret.txt**, uno en **/tmp** y el otro en **/home**.    
Leemos el archivo que se encuentra en **/home** y vemos que nos devuelve una probable contraseña.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_19.jpg)     

Ahora probamos a entrar como **luisillo** con la contraseña que hemos encontrado.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_20.jpg)    

¡Bien!   
Ahora intentamos escalar a root.   
Ejecutamos el comando: 

`sudo -l`  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_21.jpg)      

Ahora miraremos en **GTFOBins** cómo aprovechar este binario.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_22.jpg)      

Perfecto, entonces ejecutamos el comando:  

`sudo awk 'BEGIN {system("/bin/sh")}'`  

¡Y listo!   
¡Ya somos root!  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_23.jpg)      









