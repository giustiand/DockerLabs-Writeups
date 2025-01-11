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

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/I_1.png)

Hay dos puertas abiertas, la 22 y la 80.  
Echemos un vistazo a la web.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/I_2.png)

A primera vista, no notamos nada interesante.  
Procederemos entonces con un poco de fuzzing web en busca de algo útil.  

![D](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Domain/I_3.png)

Descubrimos un nuevo recurso, **shop**.  
