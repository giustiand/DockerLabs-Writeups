# MyBB

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN MyBB`  

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

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_1.png)     

Tenemos solo un puerto abierto, el 80.  
Echemos un vistazo para ver qué corre en ese puerto.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_2.png)      

Veamos esta página y si intentamos hacer clic en "Ir al foro", veremos que somos redirigidos a la dirección panel.mybb.dl.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_3.png)   

Modifiquemos entonces nuestro archivo de hosts y agreguemos esta entrada, y volvemos a ver que corre a esta dirección.  

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_4.png)   

![M](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/MyBB/M_5.png)   








