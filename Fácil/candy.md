# Candy

# Enumeración

Empezamos con un escaneo de los puertos.  

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Candy`

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

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_1.jpg)   

Solo tenemos un puerto abierto: el 80. Como podemos ver, nmap nos reporta que hay 17 entradas que están prohibidas.    
Una de estas, llamada **un_caramelo**, debería llamarnos la atención.   
Echamos un vistazo y vemos que la página en sí no contiene nada interesante.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_2.jpg)     

Pero si miramos el código fuente, podemos obtener información valiosa.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_3.jpg)   

Intentamos iniciar sesión en la página principal, pero nos da un error.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_4.jpg)    

Por lo tanto, intentamos ver si la contraseña está codificada en base64.  
Ejecutamos el comando:  

`sudo echo 'c2FubHVpczEyMzQ1' | base64 -d` y perfecto, ¡parece que tenemos una contraseña!   

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_5.jpg)    

Intentamos iniciar sesión de nuevo y ahora podemos entrar.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_6.jpg)     

Vemos que no podemos hacer muchas cosas, por lo tanto, realizaremos un poco de fuzzing web para ver si hay otro panel de inicio de sesión donde podamos hacer algo más.    
Ejecutamos el comando:    

`sudo gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php` y vemos que encontramos un recurso llamado **/administrator**.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_7.jpg)     

Si lo abrimos, nos encontramos frente a otro panel de inicio de sesión.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_8.jpg)      

Utilizamos la misma contraseña descubierta anteriormente y ahora estamos dentro.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_9.jpg)  

Ahora intentaremos ver si podemos modificar algún template .php para enviarnos una reverse shell a nuestra máquina Kali.    
Para ello, iremos a **System** - **Site template**.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_10.jpg)   

Seleccionamos el tema.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_11.jpg)  

Y ahora intentaremos modificar un archivo .php insertando una reverse shell.   
Modificaremos el archivo **error.php**.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_12.jpg)   

Una vez hecho, solo tendremos que ponernos en escucha en el puerto 6969 y abrir la ruta **http://172.17.0.2/templates/cassiopeia/error.php**.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_13.jpg)   

¡Perfecto!  
¡Ya estamos dentro!  
Ahora, si ejecutamos los comandos `sudo -l` o `find / -perm -4000 2>/dev/null`, vemos que no podemos hacer muchas cosas.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_14.jpg)   

Una cosa que se me ocurre es buscar entre todos los archivos .txt si hay alguno que nos pueda servir.    
Ejecutamos el siguiente comando:    

`find / -type f -name *.txt 2>/dev/null`  

Y entre los resultados, hay algo que nos llama la atención.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_15.jpg)    

Lo abrimos y vemos que hay información interesante.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_16.jpg)    

Intentamos conectarnos a MySQL con el usuario **luisillo**, pero no nos deja.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_17.jpg)   

Entonces, probamos a ver si en la máquina existe un usuario llamado **luisillo**.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_18.jpg)  

¡Pues sí que hay!    
Lo que haremos entonces será probar a usar la contraseña que hemos descubierto para convertirnos en **luisillo**.   

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_19.jpg)  

Ha funcionado.   
Ahora ejecutaremos el comando `sudo -l`.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_20.jpg)  

Bien, consultamos la web de GTFOBins para ver cómo podemos utilizar este binario e intentar convertirnos en root.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_21.jpg)    

Buscando por internet, vemos que podríamos abusar de este binario para modificar el archivo **/etc/sudoers**.    
Si abrimos este archivo en nuestra máquina Kali como root, podemos ver su estructura.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_22.jpg)    

La idea entonces sería modificar la entrada de manera que **luisillo** pueda ejecutar todos los comandos con su contraseña.    
Por lo tanto, regresamos a la máquina víctima y, siguiendo lo sugerido en GTFOBins, hacemos la modificación al archivo **sudoers**.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_23.jpg)    

Por lo tanto, ahora será suficiente teclear el comando `sudo su`, ingresar la contraseña de **luisillo** y...  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_23.jpg)      

¡Listo!   
¡Somos root!  















