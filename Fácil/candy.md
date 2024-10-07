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

Tenemos solamente un puerto abierto, el 80 y como podemos ver nmap nos reporta que hay 17 entradas que están no permitida.  
Una de esta nombrada **un_caramelo** debería llamarnos la atención.  
Echamos un ojo y vemos que la página en sí no contiene nada de interesante  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_2.jpg)     

pero si miramos el código fuente podemos sacar información valiosa.   

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_3.jpg)   

Probamos a logeranos en la pagina principal pero nos da error.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_4.jpg)    

Por lo tanto intentamos a ver si la contraseña está condificada en base64.  
Damos al comando:  

`sudo echo 'c2FubHVpczEyMzQ1' | base64 -d ` y perfecto, parece que tenemos una contraseña!  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_5.jpg)    

Intentamos a logearnos otra vez y ya podemos entrar.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_6.jpg)     

Vemos que no podemos hacer muchas cosas, por lo tanto haremos un poco de fuzzing web para ver si hay otro panel de login donde podamos hacer algo más.  
Ejecutamos el comando:  

`sudo gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php`  y vemos que encontramos un recurso nombredo **/administrator**.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_7.jpg)     

Si lo abrimos nos encontramos frente a otro panel de login.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_8.jpg)      

Utilizamos la misma contraseña descubierta anteriormente y ya estamos dentro.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_9.jpg)  

Ahora intentaremos ver si podemos modificar algun template .php para que podamos enviarnos una reverse shell a nuestra máquina kali.  
Para ello iremos a System - Site template  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_10.jpg)   

Seleccionamos el tema.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_11.jpg)  

y ahora intentaremos modificar un fichero .php insertanto una reverse shell.  
Modificaremos el fichero **error.php**.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_12.jpg)   

Una vez hecho solo tendremos que ponernos en escucha en el puerto 6969 y abrir la ruta http://172.17.0.2/templates/cassiopeia/error.php   

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/candy/C_13.jpg)   

Perfecto!  
Ya estamos dentro!  








