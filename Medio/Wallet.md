# Wallet

# Enumeración

Empezamos con un escaneo de los puertos.

`sudo nmap -p- --open -sC -sS -sV --min-rate=5000 -n -Pn -vvv 172.17.0.2 -oN Wallos`  

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

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_1.jpg)    

Solo tenemos un puerto abierto, el 80.  
Echemos un vistazo a la web para ver qué hay.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_2.jpg)    

A primera vista, no vemos nada interesante.   
Sin embargo, notamos que si intentamos hacer clic en "Get A Quote", nos redirige a la dirección **panel.wallet.dl**.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_3.jpg)    

Por lo tanto, añadiremos panel.wallet.dl a nuestro archivo hosts y visitaremos el nuevo recurso encontrado.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_4.jpg)   

Bien, nos encontramos frente a un formulario de registro; por lo tanto, lo que haremos será registrarnos con el usuario "test".   
Una vez registrados, accedemos con el usuario recién creado.   

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_5.jpg)    

Se abre una página donde aparece un botón que dice "Añadir primera suscripción", veamos si encontramos algo interesante.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_6.jpg)   

Podemos ver que en la parte superior derecha hay un menú y, entre las diversas opciones, está "Acerca de".  
Veamos si podemos obtener información interesante sobre la versión de esta aplicación.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_7.jpg)    

Bien, podemos constatar que nos encontramos frente a la versión 1.11.0 de una aplicación llamada **wallos**.  

Lo que podríamos hacer ahora es buscar con herramientas como searchexploit si existen vulnerabilidades conocidas para esta versión.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_8.jpg)    

¡Perfecto!  
Podemos ver cómo explotar esta vulnerabilidad.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_9.jpg)    

Entonces, lo que podemos hacer es acceder al menú "Añadir primera suscripción" y cargar una reverse shell cuando nos pida cargar un logo.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_10.jpg)   

Ahora vamos a interceptar la petición con Burp Suite y modificamos el tipo de contenido.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_11.jpg)   

¡Perfecto!  
Nuestro archivo debería haber sido cargado correctamente.  


![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_12.jpg)    

Mirando las instrucciones de Exploit, nos dice que nuestro archivo debería ser cargado en la ruta /images/uploads/logos/.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_13.jpg)    

¡Perfecto, ahí la tenemos!  
Ahora nos pondremos en escucha en nuestra máquina y ejecutaremos el archivo recién cargado.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_14.jpg)  

¡Estamos dentro!  

# Escalada de privilegios  

Como es habitual, realizamos el tratamiento de la shell para poder trabajar más cómodamente.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_15.jpg)   

Entonces, ejecutamos el comando `sudo -l` para ver si podemos escalar privilegios.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_16.jpg)    

Bien, ahora consultamos la web de GTFOBins [https://gtfobins.github.io/gtfobins/awk/#sudo](https://gtfobins.github.io/gtfobins/awk/#sudo) para ver cómo podemos abusar de este binario.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_17.jpg)     

Ejecutamos el comando sugerido y nos convertimos en el usuario pylon.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_18.jpg)     

Ejecutamos el comando `ls` y vemos que hay una carpeta interesante llamada **secretitotraviesito.zip**.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_19.jpg)  

Intentamos extraer su contenido, pero se nos solicita una contraseña.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_20.jpg)  

Por lo tanto, debemos intentar transferir este archivo a nuestra máquina para intentar descifrar la contraseña.    
Lo que haremos será copiar el archivo a la carpeta `/tmp` y transferirlo con el usuario **www-data** a la carpeta `/`, donde previamente ejecutamos nuestra reverse shell.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_21.jpg)    

Entonces, si ahora visitamos la URL `http://panel.wallet.dl/images/uploads/logos/`, encontraremos nuestro archivo ZIP y podremos descargarlo en nuestra máquina.  
Una vez descargado, utilizaremos las utilidades **zip2john** y **john** para intentar descubrir la contraseña.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_22.jpg)     

Perfecto, ahora descomprimamos el archivo ZIP y veamos su contenido.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_23.jpg)   

Perfecto, hemos obtenido un nombre de usuario y una contraseña.  
Iniciemos sesión entonces como el usuario **pinguino**.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_24.jpg)   

Ahora ejecutemos el comando `sudo -l` para verificar si podemos escalar a root.  

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_25.jpg)  

Perfecto, podemos utilizar la utilidad `sed` para modificar el archivo `/etc/passwd` eliminando la contraseña de root.  
Podemos hacerlo ingresando el siguiente comando:  

`sudo sed -i 's/^root:x:/root::/' /etc/passwd`  y...

![W](https://github.com/giustiand/DockerLabs-Writeups/blob/main/Medio/images/Wallet/W_26.jpg)   

¡Listo!  
¡Ya somos root!












