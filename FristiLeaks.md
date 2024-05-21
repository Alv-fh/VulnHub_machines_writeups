# Writeup

## Conocimiento de la red

Mediante el comando ´arp-scan´ hacemos un escaneo de la red para conocer la IP de la máquina.

![alt text](image.png)

## Conectividad con la máquina objetivo

Hacemos un ping para comprobar que tenemos conectividad con la máquina y vemos que tiene un **TTL** de 64, por lo que se trata de una máquina Linux.

![alt text](image.png)

## Escaneo de puertos y servicios

Hacemos un escaneo con nmap y utilizamos la siguiente sintaxis:

`sudo nmap -p- -sS -sC -sV --min-rate=5000 -n -vvv -Pn 192.168.1.137 -oN puertos`

![alt text](image.png)

Vemos que tiene el puerto 80 abierto y que existe el fichero **robots.txt** con posibles rutas.

Comprobamos la IP en el navegador y efectivamente hay una página web.

![alt text](image-1.png)

Ahora buscamos el archivo **robots.txt** que nos ha reportado nmap.

![alt text](image.png)