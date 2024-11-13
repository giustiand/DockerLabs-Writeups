# Picadilly

# Enumeración

Empezamos con un escaneo de puertos.  

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Picadilly`  

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

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_1.jpg)     

Tenemos dos puertos abierto, el 80 y el 443.  
Echemos un vistazo al puerto 80 para ver si podemos extraer alguna información interesante.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_2.jpg)  

Veamos qué contiene el archivo llamado **backup.txt**.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_3.jpg)    

Bien, podemos deducir que hay un usuario llamado **mateo** y que su contraseña está cifrada con un método que no sabemos exactamente cuál es.  
Al buscar en internet, podemos constatar que la contraseña está cifrada con el método de César: [https://es.wikipedia.org/wiki/Cifrado_C%C3%A9sar](https://es.wikipedia.org/wiki/Cifrado_C%C3%A9sar).  
Entonces, utilizaremos un descifrador en línea para decodificar la contraseña.  
Usaremos el sitio web [https://www.dcode.fr/cifrado-cesar](https://www.dcode.fr/cifrado-cesar).  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_3.jpg)   

Tenemos una lista de posibles contraseñas, aunque la única que parece tener sentido es la primera.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_4.jpg)   

Ahora echemos un vistazo para ver qué hay en el puerto 443.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_5.jpg)   

Nos llama la atención la posibilidad de cargar archivos.  
Lo que haremos será intentar cargar un archivo .php que contenga una reverse shell.  
Una vez cargado, realizaremos fuzzing para ver si existe alguna carpeta como **upload**, **uploads** o similares donde pueda haberse cargado el archivo.  
Usaremos la herramienta **wfuzz** para realizar el fuzzing.  

`sudo wfuzz -c -L --hl=9 --hc=400 -u https://172.17.0.2/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_6.jpg)   

Perfecto, hemos encontrado lo que buscábamos. Ahora nos pondremos a escuchar en nuestra máquina Kali y ejecutaremos el archivo .php, que ahora sabemos que se encuentra en la carpeta **uploads**.

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_7.jpg)   

¡Excelente! 
¡Ya estamos dentro!  
Ahora realizaremos el tratamiento de la **TTY** para poder trabajar más cómodamente.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_8.jpg)    

Perfecto, intentemos iniciar sesión con el usuario **mateo** y la contraseña que descubrimos anteriormente, **easycrxazy**. 

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_9.jpg)  

Recibimos un mensaje de error. 
Probamos con las otras contraseñas que habíamos descifrado, pero no tuvimos éxito. 
Dado que la única que parece tener sentido es **easycrxazy**, intentaremos con **easycrazy**.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_10.jpg)    

¡Genial! ¡Ha funcionado! Ahora que tenemos acceso, podemos continuar con los siguientes pasos.  

# Escalada de privilegios  

Ejecutamos el comando `sudo -l`.   

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_11.jpg)    

Perfecto, ahora consultaremos la web de **GTFOBins** [https://gtfobins.github.io/](https://gtfobins.github.io/) para ver cómo podemos abusar de este binario.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/picadilly/P_12.jpg)   

Perfecto, ejecutar los siguientes comandos nos permitirá aprovechar el binario para obtener una shell de **root**. 

```bash
CMD="/bin/sh"
sudo php -r "system('$CMD');"
```

Esto debería abrir una nueva shell con privilegios elevados (si el binario `php` tiene permisos de sudo sin contraseña).









