# Chocolate lovers

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Chocolate_lovers`

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

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_1.jpg)   

Tenemos solamente un puerto abierto, el 80.  
Echamos un vistazo a ver que hay.  
Vemos que hay la pagina por defecto de Apache.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_2.jpg)     

Miramos a ver si hay algo de interesante en el código fuente.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_3.jpg)     

Abrimos el recurso /nibbleblog para ver lo que hay.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_4.jpg)      

Ahora haremos un poco de fuzzing web.  
Ejecutamos el comando:  

`sudo gobuster dir -u http://172.17.0.2/nibbleblog -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php`  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_5.jpg)      

Hay una ruta, **admin.php** que nos debería llamar la atención.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_6.jpg)       

Como podemos ver nos encontramos en frente a un panal de login. 
Probamos a poner admin/admin y por suerte ya estamos dentro.  

![C](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/chocolate_lovers/C_7.jpg)       

Mirando el la página vemos que es la versión 4.0.3 "Coffee".  
Lo que podemos hacer es buscar en Google si encontramos algún exploit.  









