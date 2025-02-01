# DevTools
# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN DevTools`  

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

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_1.png)     

Solo hay dos puertas abiertas, la 22 y la 80.  
Echemos un vistazo a la web para ver qué contiene.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_2.png)  

Vemos que se nos solicita ingresar un nombre de usuario y una contraseña, y no podemos hacer absolutamente nada.  
Entonces, podemos intentar interceptar la solicitud con BurpSuite para ver qué está sucediendo.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_3.png)    

Vemos que al abrir la página se ejecuta un script .js llamado **backupp.js**.  
Al analizar su contenido, somos capaces de identificar un nombre de usuario y una contraseña.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_4.png)  

Intentemos entonces autenticarnos en la página con las credenciales recién obtenidas.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_5.png)    

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_6.png)  

Vemos una página que habla sobre los DevTools, pero no encontramos nada más interesante.  

Lo primero que podemos intentar es usar las credenciales recién obtenidas para intentar acceder por SSH, probaremos tanto con la contraseña **chocolate** como con la contraseña **baluleroh**.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_7.png)  

Ninguna de las dos funciona. Hagamos un poco de fuzzing en la página web para ver si hay otros recursos interesantes, usaremos la herramienta **feroxbuster**.   

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_8.png)    

No obtenemos nada en absoluto.  
En este punto, lo que podríamos intentar es realizar un ataque de fuerza bruta utilizando las dos credenciales descubiertas anteriormente, y probar con una lista de usuarios para ver si encontramos algún acceso válido.  
Entonces, creamos un archivo que contenga las 2 contraseñas encontradas anteriormente.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_9.png)   

Entonces, utilizaremos **hydra** para realizar un ataque de fuerza bruta contra el servicio SSH.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_10.png) 

¡Perfecto! ¡Hemos obtenido una combinación válida!  

# Escalada de privilegios  

Accedemos entonces con SSH y verificamos si encontramos algún archivo interesante en la carpeta del usuario.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_10.png)     

Echemos un vistazo al contenido del archivo **nota.txt**.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_11.png)   

Bien, ahora ejecutemos `sudo -l` para ver si podemos abusar de algún binario.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_12.png)   

¡Perfecto!  
Consultemos entonces [GTFOBins](https://gtfobins.github.io/) y busquemos como proceder.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_13.png)    

No nos queda más que aprovechar **xxd** para leer el archivo **data.bak** que se encuentra en el directorio raíz y ver su contenido.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_14.png)      

Ahora no nos queda más que cambiar de usuario y...  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/DevTools/D_15.png)   

¡Y listo!  
¡Ya somos root!  



