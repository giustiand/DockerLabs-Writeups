# AguaDeMayo

# Enumeración

Empezamos con un escaneo de puertos.  

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN AguaDeMayo`  

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

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_1.jpg)     

Tenemos solamente 2 puertas abiertas, la 22 y la 80.  
Echemos un vistazo para ver qué hay en la puerta 80.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_2.jpg)   

A simple vista no hay nada interesante.    
Analicemos el código fuente para ver si podemos extraer alguna información útil.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_3.jpg)     

Como podemos notar, aparece un extraño comentario al final de la página.  
Al buscar más información en internet, podemos constatar que se trata de un tipo particular de lenguaje denominado **Brainfuck**.  
Podemos descifrar el mensaje directamente en línea accediendo a uno de los numerosos sitios web que permiten realizar esta operación, como por ejemplo [https://md5decrypt.net/en/Brainfuck-translator/](https://md5decrypt.net/en/Brainfuck-translator/).  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_4.jpg)  

Obtenemos esta cadena, **bebeaguaqueessano**.  

Ahora podríamos realizar un poco de fuzzing web para ver si podemos obtener más información.  
Por lo tanto, digitaremos el comando:  

`sudo gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt`  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_5.jpg)    

Echemos un vistazo al recurso que hemos descubierto.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_6.jpg)      

Descarguemos la imagen para ver si con herramientas como **exiftool** o **steghide** podemos descubrir alguna información interesante.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_7.jpg)     

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_8.jpg)   

No somos capaces de obtener nada interesante.  
Analizando el nombre del archivo, **agua_ssh**, podemos intuir que **agua** podría ser el nombre del usuario y que la cadena que descubrimos anteriormente, **bebeaguaqueessano**, podría ser la contraseña.  
Intentemos entonces conectarnos por SSH.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_9.jpg)    

¡Perfecto, estamos dentro!  

Ahora ejecutaremos el comando `sudo -l` para ver si podemos escalar privilegios.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_10.jpg)     

Bien, vemos que podemos ejecutar la herramienta **bettercap** como usuario root.  
Analizamos la web de GTFOBins para ver si podemos abusar de este binario, pero no encontramos nada.  
Entonces, intentemos ejecutar **bettercap** para ver qué podemos obtener.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_11.jpg)      

Escribimos el comando `help` y analizamos las opciones.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_12.jpg)   

Nos llama la atención la línea donde vemos que podemos ejecutar comandos.  
Intentemos escribir **whoami** para ver si realmente somos root.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_13.jpg)     

¡Perfecto! Lo que podemos hacer ahora es eliminar la contraseña del usuario root y luego acceder mediante el comando `su`.  
Entonces, escribimos `passwd -d root` para eliminar la contraseña del usuario root.  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_14.jpg)     

¡Ahora será suficiente salir de la utilidad **bettercap**, escribir el comando `su` y...  
¡Listo! ¡Somos root!  

![A](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/aguademayo/A_15.jpg)    









