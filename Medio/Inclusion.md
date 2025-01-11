# Inclusion

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Inclusion`  

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

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_1.png)

Hay dos puertas abiertas, la 22 y la 80.  
Echemos un vistazo a la web.  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_2.png)

A primera vista, no notamos nada interesante.  
Procederemos entonces con un poco de fuzzing web en busca de algo útil.  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_3.png)

Descubrimos un nuevo recurso, **shop**, echemos un vistazo.  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_4.png)  

Nos llama la atención este estraño mensaje que dice **Error de Sistema: ($_GET['archivo']");**.  
Intentemos ver si esta página es vulnerable a un ataque LFI (Local File Inclusion) utilizando el parámetro **archive** mostrado en el error.  
Ejecutamos el comando:  

`wfuzz -c --hc=404 --hl=44 --hw=90 -t 200 -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -u http://172.17.0.2/shop/index.php?archivo=FUZZ`  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_5.png)   

¡Eso parece!  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_6.png)   

Ahora que hemos descubierto dos usuarios lo que podemos hacer es realizar un ataque de fuerza bruta contra el servicio ssh.  
Creamos un fichero llamado **users.txt** con los 2 nombres de usuario que acabamos de descubrir e intentamos ver si conseguimos un login válido.  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_7.png)   

Probamos a conectarnos.  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_8.png)   

¡Perfecto!  
¡Estamos dentro!  

# Explotación  

Probamos los comandos clásicos pero no obtenemos nada.  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_9.png)  

También cargamos linpeas y linenum pero no conseguimos nada.  
Lo que podemos hacer es cargar una utilidad en la máquina víctima para intentar forzar el usuario **seller** que sabemos que está presente en la máquina.  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_10.png)  

Ahora todo lo que tenemos que hacer es cargar el diccionario rockyou.txt (`scp /usr/share/wordlists/rockyou.txt ssh manchi@172.17.0.2:/tmp`) en la máquina víctima y ejecutar el script.   

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_11.png)  
![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_12.png)  

¡Excelente!  
A continuación, iniciamos sesión como usuario vendedor para ver si podemos escalar a root.  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_13.png)    

Ejecutamos `sudo -l` y vemos que podemos abusar del binario **php**.  
Consultemos el sitio web [GTFOBins] (https://gtfobins.github.io/gtfobins/php/#sudo) para ver cómo abusar de este binario.  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_14.png)    

Ejecutamos los comandos como se sugiere y...  

![I](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Inclusion/I_15.png)   

¡Listo!  
¡Ya somos root!  














