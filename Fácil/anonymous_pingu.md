# AnonymousPingu

# Enumeración

Empezamos con un escaneo de puertos.  

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN AnonymousPingu`  

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

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_1.jpg)    

Tenemos solamente 2 puertas abiertas, la 21 y la 80.   
Nos llama la atención que el servicio FTP tenga habilitada la autenticación anónima.  
Nos conectaremos con **FileZilla** para verificar si tenemos permisos para cargar archivos.  
Una vez dentro, vemos que existe una carpeta llamada **upload** con permisos de lectura y escritura para todos los usuarios.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_2.jpg)   

Lo que haremos entonces será cargar una reverse shell en PHP.  
Descargaremos entonces nuestra reverse shell, por ejemplo, desde el sitio de [pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell).  
Una vez que hayamos modificado los parámetros adecuados, la subiremos al servidor FTP.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_3.jpg)    

Ahora será suficiente acceder a la dirección http://172.17.0.2/upload y ejecutar nuestra reverse shell, una vez que estemos en modo de escucha en nuestra máquina atacante.    

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_4.jpg)  

Perfecto, estamos dentro.   
Ahora haremos el tratemiento de la shell para poder trabajar con mayor comodidad.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_5.jpg)    

# Escalada de privilegios  

Ahora ejecutaremos el comando `sudo -l` para ver si podemos abusar de algún binario y escalar privilegios.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_6.jpg)    

Bien, echemos un vistazo a [GTFOBins](https://gtfobins.github.io/).  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_7.jpg)   

Excelente, podemos ejecutar `man` para obtener una shell como el usuario **pingu**.    

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_8.jpg)   

¡Bien!  
Ahora volvemos a ejecutar el comando `sudo -l`.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_9.jpg)    

Al buscar nuevamente en el sitio de [GTFOBins](https://gtfobins.github.io/), vemos que con `dpkg` podemos ejecutar comandos, de manera similar a como lo hicimos con `man`.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_10.jpg)     

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_11.jpg)     

¡Perfecto!   
Ejecutemos nuevamente el comando `sudo -l` para ver si finalmente podemos convertirnos en root.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_12.jpg)     

Demos otro vistazo a [GTFOBins](https://gtfobins.github.io/) para ver lo que podemos hacer.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_13.jpg)     

Perfecto, lo que haremos será modificar los permisos del archivo `/etc/passwd` y luego usar **sed** para editar la cadena de **root** y eliminar la contraseña de inicio de sesión.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_14.jpg)     

Ahora que tenemos los permisos necesarios, ejecutaremos el siguiente comando:

`sed -i 's/\(^root\):x:/\1::/' /etc/passwd` y al ejecutar `su`, ¡listo! 
¡Ya somos root!    

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_15.jpg)     














