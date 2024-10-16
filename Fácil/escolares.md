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

Tenemos dos puertos abiertos, el 22 y el 80.  
Miramos a ver que hay en el puerto 80.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_2.jpg)     

A primera vista nada de interesante.  
Echamos un ojo al código fuente.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_3.jpg)  

Hay un comentario que nos debería llamar a la atención.  
Miramps la página profesores.html y vemos que hay un usaurio, que al parecer se llama Luis, que es admin de wordpress.    

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_4.jpg)    

Hacemos entonces un poco de fuzzing web para ver si podemos sacar algo util.  

`sudo gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x bak,html,php,txt`  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_5.jpg)   

Ahora miramos la ruta http://172.17.0.2/wordpress.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_6.jpg)    

Efectivamente nos encontramos frente a un wordpress.  
Mirando el código fuente vemos que hay varios enlaces a un dominion nombrado **escolares.dl**.   

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_7.jpg)    

Lo que haremos ahora será modificar el fichero **hosts** añadiendo este nuevo dominio.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_8.jpg)    

Ahora volvemos a hacer fuzzing web para ver si sacamos más recursos.  

`sudo gobuster dir -u http://escolares.dl/wordpress -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x bak,html,php,txt`

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_9.jpg)   

Ahora deberíamos intentar enumerar los usarios para intentar hacer un ataque de fuerza bruta.  
Para ello usaremos la herramienta **wpscan**.  
Ejecutaremos el comando:  

`sudo wpscan --url http://escolares.dl/wordpress -e u,p,t`  

Trás esperar un rato vemos que nos saca un nombre de usuario, **luisillo**.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_10.jpg)   

Ahora intentaremos hacer un ataque de fuerza bruta utilizando el diccionario **rockyou.txt**.  

`sudo wpscan --url http://escolares.dl/wordpress --usernames luisillo --passwords /usr/share/wordlists/rockyou.txt`  

Esperamos un largo rato pero no logramos obtener una combinación valida.  
Entonces lo que haremos será crear un nuestro diccionario personalizado utilizando las informaciones que podemos ver en la página profesores.  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_11.jpg)     

Ejecutarmos el comando `cupp -i`  

![E](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/escolares/E_12.jpg)      

Ahora volveremos a utilizar wpscan para realizar un ataque de fuerza bruta con nuestro diccionario recien creado.  

`sudo wpscan --url http://escolares.dl/wordpress --usernames luisillo --passwords luis.txt`  

Perfecto!  
Tenemos una combinación valida.  

