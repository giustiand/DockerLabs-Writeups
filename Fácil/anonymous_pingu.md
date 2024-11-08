# AnonymousPingu

# Enumeración

Empezamos con un escaneo de puertos.  

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN AnonymousPingu`  

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

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_1.jpg)    

Tenemos solamente 2 puertas abiertas, la 21 y la 80.   
Nos llama la atención que el servicio FTP tenga habilitada la autenticación anónima.  
Nos conectaremos con **FileZilla** para verificar si tenemos permisos para cargar archivos.  
Una vez dentro, vemos que existe una carpeta llamada **upload** con permisos de lectura y escritura para todos los usuarios.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_2.jpg)   

Lo que haremos entonces será cargar una reverse shell en PHP.  
Descargaremos entonces nuestra reverse shell, por ejemplo, desde el sitio de [pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell).  
Una vez que hayamos modificado los parámetros adecuados, la subiremos al servidor FTP.  

![P](https://github.com/giustiand/DockerLabs-Writeups/blob/main/F%C3%A1cil/images/anonymous_pingu/P_3.jpg)     







