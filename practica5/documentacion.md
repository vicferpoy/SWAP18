# **PRÁCTICA 5: Replicación de bases de datos MySQL**

Víctor Fernández Poyatos

## **Objetivos**
· Copiar archivos de copia de seguridad mediante ssh.

· Clonar manualmente BD entre máquinas.

· Configurar la estructura maestro-esclavo entre dos máquinas pare realizar el clonado automático de la información.

### I Crear una BD con al menos una tabla y algunos datos
Para crear una BD, lo primero que necesitamos es tener instalado MySQL, paso que daremos por hecho puesto que por lo general se instala al instalar el sistema. Lo siguiente es entrar en modo root dentro de MySQL, usando el comando:

```
mysql –uroot –p
```

Insertamos la contraseña que tengamos puesta para MySQL y procedemos a la creación de la base de datos:

*Creación de la base de datos*
```
mysql> create database contactos;   
```
*Selección de la base de datos creada*
```
mysql> use contactos;
```
*Creación de campos en la tabla*
```
mysql> create table datos(nombre varchar(100),tlf int);
```
*Inserción de datos para esos campos*
```
mysql> insert into datos(nombre,tlf) values ("pepe", 95834987);
mysql> insert into datos(nombre,tlf) values ("swap", 101010101);
```
*Comprobamos si se han introducido bien los datos mostrando la tabla*
```
mysql> select * from datos;
```


![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/tabla1.PNG)


### II Realizar la copia de seguridad de la BD completa usando mysqldump en la máquina principal y copiar el archivo de copia de seguridad a la máquina secundaria y III Restaurar dicha copia de seguridad en la segunda máquina (clonado manual de la BD), de forma que en ambas máquina esté esa BD de forma idéntica
Para realizar una copia de seguridad, es tan sencillo como usar el comando ```mysqldump```, pero antes de eso debemos asegurarnos de que la BD que vamos a copiar, no se va a modificar. Para ello, desde el menú de MySQL introducimos el siguiente comando:
```
mysql> FLUSH TABLES WITH READ LOCK;
mysql> quit
```
A continuación, procedemos a realizar la copia de la BD en el directorio que queramos. En mi caso, usaré `/tmp/`:
```
mysqldump contactos -u root -p > /tmp/contactos.sql
```
Ya que hemos realizado la copia, podemos volver a desbloquear las tablas. Una vez más, desde MySQL:
```
mysql> UNLOCK TABLES;
mysql> quit
```
Ahora ya podemos usar *scp* desde la otra máquina para descargar la copia de la BD. Introducimos en el terminal como root:
```
scp máquina1:/tmp/contactos.sql /tmp/
```
Con esto descargamos el archivo de la máquina 1 y lo ponemos en el directorio /tmp/ de la máquina 2. Ahora sólo tenemos que crear una BD en nuestra máquina 2 y asignarle a ésta nuestro archivo *.sql* procediente de la máquina 1, desde MySQL:
```
mysql> CREATE DATABASE `contactos`;
mysql> quit

mysql -u root -p contactos < /tmp/contactos.sql
```
Con esto ya tendríamos la BD procediente de máquina 1 perfectamente restaurada en nuestra máquina 2. Adjunto captura de pantalla de cómo fue en mi caso para evitar errores a la hora de usar los parámetros:


![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/restauracion.PNG)

**Nota:** aunque en la captura aparezca como *ejemplodb*, posteriormente lo volví a copiar como *contactos* para evitar confusiones.


### IV Realizar la configuración maestro-esclavo de los servidores MySQL para qu ela replicación de datos se realice automáticamente
El primero paso para llevar a cabo esta configuración es elegir una máquina como maestro. Supongamos que será la máquina 1, de manera que debemos editar el archivo de configuración en `/etc/mysql/mysql.conf.d/mysqld.cnf`:

*Comentamos el parámetro bind-address*
```
#bind-address 127.0.0.1
```

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/maestro1.PNG)


*Indicamos los siguientes parámetros de la siguiente manera. Guardamos el archivo y reiniciamos servicio*
```
log_error = /var/log/mysql/error.log
server-id = 1
log_bin = /var/log/mysql/bin.log
```
```
/etc/init.d/mysql restart
```

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/maestro2.PNG)


Para el esclavo *(máquina 2)*, la configuración es exactamente la misma excepto por el parámetro *server_id*, que será 2. Reiniciamos servicio también en esta máquina al terminar.

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/esclavo1.PNG)


El siguiente paso es crear un usuario en nuestra máquina maestro con el que pueda conectarse nuestro esclavo. Para ello usaremos los siguientes comandos en MySQL:
```
mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%'
IDENTIFIED BY 'esclavo';
mysql> FLUSH PRIVILEGES;
mysql> FLUSH TABLES;
mysql> FLUSH TABLES WITH READ LOCK;


mysql> SHOW MASTER STATUS;
```

El último comando debería darnos este resultado si todo ha ido bien. Es muy importante guardar estos datos porque nos harán falta para la configuración del esclavo:

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/maestro3.PNG)


De nuevo en la máquina esclava, procedemos a usar el siguiente comando con los parámetros referentes a nuestra máquina maestro y los datos obtenidos en la captura anterior para configurar el esclavo y ponerlo a funcionar:

``` 
mysql> CHANGE MASTER TO MASTER_HOST='192.168.88.132', 
MASTER_USER='esclavo', MASTER_PASSWORD='esclavo',
MASTER_LOG_FILE='mysql-bin.000012', MASTER_LOG_POS=154,
MASTER_PORT=3306;


mysql> START SLAVE;
```

Ahora procedemos a desbloquear las tablas en el maestro con ``` mysql> UNLOCK TABLES;``` y ya podemos realizar los cambios que queramos en el maestro, actualizándose en tiempo real el esclavo. Para comprobar que todo ha ido bien podemos usar en la máquina esclavo ```mysql> SHOW SLAVE STATUS\G```. Si falla algo, podemos ver qué es exactamente gracias a los logs. En mi caso fallaba la conexión porque el puerto 3306 no estaba abierto, así que lo solucioné con ```ufw allow 3306``` o ```ufw disable```, en ambas máquinas.

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/esclavo-status.PNG)


Como se observa en la siguiente captura, todo funciona correctamente:


![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/sincronizacion.PNG)










