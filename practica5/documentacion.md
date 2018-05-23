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
create database contactos;   
```
*Selección de la base de datos creada*
```
use contactos;
```
*Creación de campos en la tabla*
```
create table datos(nombre varchar(100),tlf int);
```
*Inserción de datos para esos campos*
```
insert into datos(nombre,tlf) values ("pepe", 95834987);
insert into datos(nombre,tlf) values ("swap", 101010101);
```
*Comprobamos si se han introducido bien los datos mostrando la tabla*
```
select * from datos;
```


![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/tabla1.PNG)


### II Realizar la copia de seguridad de la BD completa usando mysqldump en la máquina principal y copiar el archivo de copia de seguridad a la máquina secundaria y III Restaurar dicha copia de seguridad en la segunda máquina (clonado manual de la BD), de forma que en ambas máquina esté esa BD de forma idéntica
Para realizar una copia de seguridad, es tan sencillo como usar el comando ```mysqldump```, pero antes de eso debemos asegurarnos de que la BD que vamos a copiar, no se va a modificar. Para ello, desde el menú de MySQL introducimos el siguiente comando:
```
FLUSH TABLES WITH READ LOCK;
quit
```
A continuación, procedemos a realizar la copia de la BD en el directorio que queramos. En mi caso, usaré `/tmp/`:
```
mysqldump contactos -u root -p > /tmp/contactos.sql
```
Ya que hemos realizado la copia, podemos volver a desbloquear las tablas. Una vez más, desde MySQL:
```
UNLOCK TABLES;
quit
```
Ahora ya podemos usar *scp* desde la otra máquina para descargar la copia de la BD. Introducimos en el terminal como root:
```
scp máquina1:/tmp/contactos.sql /tmp/
```
Con esto descargamos el archivo de la máquina 1 y lo ponemos en el directorio /tmp/ de la máquina 2. Ahora sólo tenemos que crear una BD en nuestra máquina 2 y asignarle a ésta nuestro archivo *.sql* procediente de la máquina 1, desde MySQL:
```
CREATE DATABASE `contactos`;
quit

mysql -u root -p contactos < /tmp/contactos.sql
```
Con esto ya tendríamos la BD procediente de máquina 1 perfectamente restaurada en nuestra máquina 2. Adjunto captura de pantalla de cómo fue en mi caso para evitar errores a la hora de usar los parámetros:


![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/restauracion.PNG)

**Nota: aunque en la captura aparezca como *ejemplodb*, posteriormente lo volví a copiar como *contactos* para evitar confusiones.**


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

/etc/init.d/mysql restart
```

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/maestro2.PNG)







