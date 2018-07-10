# **PRÁCTICA 1: Preparación de las herramientas**

Víctor Fernández Poyatos

### I Acceder por ssh de una máquina a otra
Para acceder por ssh de una máquina a otra, tan sólo necesitamos que se vean y conocer un usuario y contraseña dentro de la máquina a la que queremos acceder. En este caso, entraremos a la máquina 2 usando el propio usuario root, por lo que escribo:
```
ssh swap2@192.168.88.133
```
Después introduzco su contraseña y ya estaríamos dentro.
![img](https://github.com/vicferpoy/SWAP18/blob/master/practica1/img/ssh.PNG)

### II Acceder mediante la herramienta curl desde una máquina a la otra
Para este objetivo, necesitamos instalar un servidor Apache en la máquina a la que le haremos el curl para que muestre una página personalizada. Para instalar apache:
```
sudo apt-get install apache2 mysql-server mysql-client
```
Y para ver si está instalado y si está en ejecución, respectivamente:
```
apache2 –v
ps aux | grep apache
```
A su vez, para instalar curl, usamos el apt-get también:
```
sudo apt-get install curl
```
Cambiamos el contenido de la página *index.html* en la ruta `/var/www/html` y probamos a entrar usando ```curl http://direccionIPservidor/index.html```

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica1/img/curl.PNG)
