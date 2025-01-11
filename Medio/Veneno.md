# Veneno

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Veneno`  

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

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneo/V_1.png)  

Hay dos puertas abiertas, la 22 y la 80.  
Echemos un vistazo a la web.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneo/V_2.png)  

Podemos ver la página por defecto de apache2 y tampoco encontramos nada de interés en el código fuente de la página.  
Por lo tanto, vamos a hacer un poco de fuzzing.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneo/V_3.png)    

Descubrimos una nueva ruta, /uploads, a la que tenemos acceso, pero que no contiene nada.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneo/V_4.png)     
  
También vemos otra página, /problems.php, que, a primera vista, parecería ser la misma que la página de inicio.  

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneo/V_5.png)    

Intentemos ver si esta página es vulnerable a un ataque LFI (Local File Inclusion).  
Ejecutamos el comando:  

`sudo wfuzz -c -L -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt --hl=363 "http://172.17.0.2/problems.php?FUZZ=/../../../../../etc/passwd"`

![V](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Veneo/V_6.png)      

¡Así es!  




