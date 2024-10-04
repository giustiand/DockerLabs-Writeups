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

Si abrimos el fichero nombrado "EpisodiosT1" vemos un listado 

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/winterfell/W_5.jpg)   

Podemos intentar a copiar este listado en un fichero que llamaremos pass.txt para comprobar si una de estas podría ser una contraseña valida para el usuario jon.  

