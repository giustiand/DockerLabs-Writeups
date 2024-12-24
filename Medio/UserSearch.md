# User Search

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.18.0.2 -oN UserSearch`  

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

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/UserSearch/U_1.jpg) 

Tenemos 2 puertos abiertos, el 22 y el 80.  
Echemos un vistazo a la web para ver si encontramos algo interesante.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/UserSearch/U_2.jpg)   

Intentemos escribir **admin**, por ejemplo, y veamos qué sucede.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/UserSearch/U_3.jpg)   

¡Bien!  
Veamos qué nos devuelve un nombre de usuario y una contraseña.  
Si intentamos usar estas credenciales para acceder a través de SSH, no funciona.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/UserSearch/U_4.jpg)   

Lo que podemos hacer es verificar si existe algún tipo de vulnerabilidad de SQL Injection en este panel web, y efectivamente, al ingresar la cadena **admin'--'**, nos son devueltos otros dos usuarios.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/UserSearch/U_5.jpg)    

¡Perfecto!  
Intentemos nuevamente autenticarnos a través de SSH y verifiquemos si podemos iniciar sesión con el usuario **kvzlx**.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/UserSearch/U_6.jpg)    

# Escalada de privilegios  

Ejecutemos, como de costumbre, el comando `sudo -l` para ver si podemos elevar nuestros privilegios a root.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/UserSearch/U_7.jpg)   

Podemos ejecutar el script **system_info.py** presente en el directorio home sin necesidad de especificar una contraseña.  
Vamos a ver su contenido.  

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/UserSearch/U_8.jpg)   

Lo que podríamos hacer sería crear en el mismo directorio que contiene el script, un archivo llamado `psutil.py` con el siguiente contenido:  

`import os
os.system('/bin/bash')
`

Así que ahora podemos ejecutar el comando `sudo /usr/bin/python3 /home/kvzlx/system_info.py` y...

![U](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/UserSearch/U_9.jpg)    

¡Listo!
¡Ya somos root!









