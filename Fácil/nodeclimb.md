# NodeClimb  
**Herramientas y recursos usados**  
- zip2john
- john the ripper


# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN NodeClimb`  

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

![NC](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/NodeClimb/NC_1.jpg)       

Tenemos 2 puertos abiertos y, como podemos apreciar, el puerto 21 permite hacer login de manera anónima.    
Nos conectamos y descargamos el archivo **.zip** que encontramos.  

![NC](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/NodeClimb/NC_2.jpg)     

Si intentamos extraerlo, nos pide una contraseña.  

![NC](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/NodeClimb/NC_3.jpg)       

Para intentar extraer la contraseña, utilizaremos la herramienta **zip2john**.    
Crearemos un archivo llamado **hash** donde guardaremos el hash de la contraseña.  

![NC](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/NodeClimb/NC_4.jpg)        

Ahora utilizaremos la herramienta **John the Ripper** para intentar descifrar la contraseña.  

![NC](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/NodeClimb/NC_5.jpg)       

¡Perfecto!  
Ahora extraemos el archivo zip y miramos el contenido del archivo **password.txt**.  

![NC](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/NodeClimb/NC_6.jpg)       

¡Listo!   
Ya tenemos el usuario y la contraseña, y podemos conectarnos por SSH.  

# Explotación y escalada de privilegios   

Una vez dentro vemos que si le damos el comando `sudo -l` podemos ejecutar el fichero node con permisos de root.  *

![NC](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/NodeClimb/NC_7.jpg)     

Miramos en GTOBins y vemos que podemos explotarlo añadiendo esta stringa:  
`'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'`  
en un fichero .js, en este caso utilizaremos el fichero que se encuentra en /home/mario/script.js.   

![NC](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/NodeClimb/NC_8.jpg)          

Una vez modificado el fichero script.js solo tendremos que dar el comando `sudo /usr/bin/node /home/mario/script.js`  y listo, somos root!  

![NC](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/NodeClimb/NC_9.jpg)      
