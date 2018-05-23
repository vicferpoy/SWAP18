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


### II Realizar la copia de seguridad de la BD completa usando mysqldump en la máquina principal y copiar el archivo de copia de seguridad a la máquina secundaria
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
Con esto descargamos el archivo de la máquina 1 y lo ponemos en el directorio /tmp/ de la máquina 2. Para evitar dudas con las rutas o direcciones, adjunto imagen de cómo se hace en mi caso:


![img](https://github.com/vicferpoy/SWAP18/blob/master/practica5/img/restauracion.PNG)


