# **PRÁCTICA 4: Asegurar la granja web**

Víctor Fernández Poyatos

## **Objetivos**
· Instalar un certificado SSL para configurar el acceso HTTPS a los servidores.

· Configurar las reglas del cortafuegos para proteger la granja web.

### I Crear e instalar en la máquina 1 un certificado SSL autofirmado para configurar el acceso HTTPS a los servidores. Una vez configurada la máquina 1, se debe copiar al resto de máquinas servidoras y al balanceador de carga. Se debe configurar nginx adecuadamente para aceptar y balancear correctamente tanto el tráfico HTTP como el HTTPS.

Partiendo de tener Apache en la máquina a instalar el certificado SSL, procedemos a restaurar el servicio con los siguientes comandos:

```
a2enmod ssl
service apache2 restart
```

Creamos la carpeta *ssl* en la ruta `etc/apache2/ssl` y generamos el certificado:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
```

Tras rellenar los datos pertinentes, añadimos las siguientes líneas en la ruta `/etc/apache2/sites-available/default-ssl.conf`:

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica4/img/default-ssl.PNG)

Activamos el sitio y reiniciamos Apache:

```
a2ensite default-ssl
service apache2 reload
```

Se repite este proceso para las máquinas restantes a excepción de la máquina con *nginx*, en la que además tendremos que añadir las siguientes líneas a su archivo de configuración en `etc/nginx/conf.d/default.conf`:

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica4/img/nginx-conf.PNG)

Reiniciamos el servicio de nginx y probamos ```curl –k``` para ver si todo funciona correctamente:

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica4/img/curl-https.PNG)

### II Configurar las reglas del cortafuegos con IPTABLES para asegurar el acceso a uno de los servidores web, permitiendo el acceso por los puertos de HTTP y HTTPS a dicho servidor. Esta configuración se hará en una de las máquinas servidoras finales (p.ej. en la máquina 1), y se debe poner en un script con las reglas del cortafuegos que se ejecute en el arranque del sistema (según la versión de Linux, se llevará a cabo de una forma u otra).

Para esta tarea se creará un script que automatice todo este proceso, eliminando reglas previas añadiendo las que necesitamos. En este caso el script lo he llamado **iptables.sh**:
 
```
#!/bin/sh
    # Se eliminan todas las reglas del cortafuegos
iptables -F
iptables -X
iptables -Z
iptables -t nat -Z
    # Se prohíbe la entrada o salida de todo
iptables -P INPUT   DROP
iptables -P OUTPUT  DROP
iptables -P FORWARD DROP
    # Añadimos la excepción del localhost
iptables -A INPUT  -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
    # Añadimos las excepciones de HTTP, HTTPS y SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT
```

A continuación, creamos un servicio que ejecute este script en cada inicio y ya lo tendríamos funcionando:

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica4/img/iptables.PNG)

