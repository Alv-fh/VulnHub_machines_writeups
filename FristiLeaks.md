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

Ahora hacemos





![alt text](image.png)
