![image](https://github.com/user-attachments/assets/52f5c54e-e135-4d0e-afb6-e4757153ad6f)# Vacaciones
**Herramientas y recursos usados**
- nmap
- searchsploit
- msfconsole

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Vacaciones`

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
![V_1](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Vacaciones/V_1.jpg)  

Como podemos ver tenemos 2 puertos abiertos, el puerto 22 que corresponde al servicio SSH y el puerto 80 que corresponde a HTTP.  
Probamos a ver que aparece si abrimos un explorador a la dirección http://172.17.0.2  

![V_2](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Vacaciones/V_2.jpg) 

Como podemos ver nos sale una pagina en blanco.
Probamos a ver si hay algo "escondido" en el código de la pagina ( `Ctrl + U` ).  

![V_3](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Vacaciones/V_3.jpg)    


Tomamos nota de esta información y procedemos con la explotación.

# Explotación
Vamos a crear un fichero de testo con nombre `users.txt` donde pondremos juan y camilo.  
Ahora utilizaremos hydra para ver si logramos sacar una contraseña valida para podernos logear en SSH.  
Tras esperar un buen rato encontramos una contraseña valida para el usuario camilo y podemos asý conectarnos a la maquina victima por SSH.  

![V_4](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Vacaciones/V_4.jpg)  

![V_5](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Vacaciones/V_5.jpg)  

# Escalada de privilegios
La prima cosa que me ocurre hacer es ir a revisar los correos.  
![V_6](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Vacaciones/V_6.jpg)  

Probamos esta nueva contraseña que hemos descubierto para hacer login con el usuario juan.  

![V_7](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Vacaciones/V_7.jpg)  

Ahora provamos a escalar los privilegios a root.  
Damos el comando `sudo -l`.  

![V_8](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Vacaciones/V_8.jpg)   

Lo que podemos hacer ahora es mirar en la web de GTFOBins si encontramos alguna manera para poder escalar de privilegios.  

![V_9](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Vacaciones/V_9.jpg)   

Lanzamos el comando y listo, ya somos root!  
![V_10](https://github.com/giustiand/DockerLabs-Writeups/blob/main/MuyF%C3%A1cil/.images/Vacaciones/V_10.jpg) 


