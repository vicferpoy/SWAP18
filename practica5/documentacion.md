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


