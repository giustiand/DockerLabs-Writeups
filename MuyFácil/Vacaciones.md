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

