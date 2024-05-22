# Writeup

## Conocimiento de la red

Mediante el comando `arp-scan` hacemos un escaneo de la red para conocer la IP de la máquina.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/e8fa4348-9c34-49ef-b72c-222270aa546f)


## Conectividad con la máquina objetivo

Hacemos un ping para comprobar que tenemos conectividad con la máquina y vemos que tiene un **TTL** de 64, por lo que se trata de una máquina Linux.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/5df92dd9-f7db-4ecd-b2c2-0ec97d9590d3)

## Escaneo de puertos y servicios

Hacemos un escaneo con nmap y utilizamos la siguiente sintaxis:

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 192.168.1.137 -oN puertos`

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/37e5edae-571a-46aa-bdc9-5d6edbd2b0af)

Vemos que tiene el puerto 80 abierto y que existe el fichero **robots.txt** con posibles rutas.

Comprobamos la IP en el navegador y el archivo **robots.txt**.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/0fd01645-ef64-4981-888f-9e2db52219a7)

Compruebo que en las tres rutas sale esto, por lo que se me ocurre hacer fuzzing web.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/1239d99a-adfa-45f1-88bb-90a4feb4d34e)

Hacemos uso de **gobuster** pero no me reporta nada. Por lo que empiezo a probar posibles rutas hasta que doy con una llamada **fristi**.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/ebac3e97-3d11-46ee-a398-4b423b102201)

Probamos credenciales típicas como **admin** **admin**, pero después de darle vueltas, decido hacer fuzzing web otra vez pero a la ruta **/fristi**.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/d0975690-da6b-4046-a896-cedc59f2f660)

Pruebo a ver que hay en el uploads.php y me hace una redirección al portal de admin en el que hay un login.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/86335577-3326-4a4b-a943-2854440374f6)

Me pongo a revisar el contenido de la página y veo un posible usuario llamado **eezeepz**.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/5b9b0d15-0492-498b-9b64-596bd67dc701)

Veo que en la imagen hay un comentario que dice que está en Base 64. Por lo que lo decodifico y dice esto.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/0a21fe2a-1afd-4ae9-8fd4-12538c5ec7e2)

Por lo que pruebo a decodificar el texto en PNG y me sale esta imagen.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/7993f881-b0df-4235-b33d-c9956969d8a7)

Pruebo con el usuario nombrado antes y con esto que parece una clave y funciona.

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/7f327683-b9c6-4cfd-bf3b-f499e24853b7)

Veo que puedo subir archivos, por lo que se me ocurre subir una reverse shell en php pero veo que solo puedo subir archivos con extensión .jpg,.png y .gif

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/76dd492d-9c50-4cd1-aa04-2313906334b0)

Por lo que pruebo y pruebo extensiones hasta que pienso en poner .png delante de la extensión .php haciendo uso del comando `mv`.

Veo que se sube correctamente y dice que se ha subido a **/uploads**.

Veo que no puedo ver lo que hay en **uploads** pero se que está subido por lo que pongo la ruta pero sin antes hacer un `nc -nlvp 443`

![image](https://github.com/Alv-fh/VulnHub_machines_writeups/assets/109484163/c085a55a-fc15-4e01-9cb0-5a907ec13f17)

