# **PRÁCTICA 3: Balanceo de carga**

Víctor Fernández Poyatos

## **Objetivos**

### I Configurar una máquina e instalar nginx como balanceador de carga

Para realizar esta práctica, consto de dos máquinas finales (*vfernandez1* y *vfernandez2*), dos máquinas configuradas como balanceadores para cada caso (*balanceador*, para nginx y *balanceador2*, para haproxy) y una última máquina con la que realizaré las peticiones y haré los benchmarks (*vfernandez3*).

**vfernandez1** - 192.168.198.128

**vfernandez2** - 192.168.198.129

**vfernandez3** - 192.168.198.131

**balanceador** - 192.168.198.130

**balanceador2** - 192.168.198.132

Instalamos nginx usando las instrucciones que se describen en el pdf de la *Práctica 3* en la máquina *balanceador* y configuramos el archivo */etc/nginx/conf.d/default.conf* de forma básica para probar si funciona, utilizando una configuración basada en ponderación, asumiendo que la máquina *vfernandez1* recibirá el doble de peticiones que *vfernandez2*.

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica3/img/nginx_pond.PNG)

En la imagen se aprecia la configuración usada para nginx y qué debería mostrar cada máquina. En la máquina 3 podemos observar cómo se comporta exactamente como se esperaba.


### II Configurar una máquina e instalar haproxy como balanceador de cargar

Para esta parte vamos a usar la otra máquina que hace de balanceador e instalar haproxy, además de configurarlo tal y como dice el guión.
El archivo de configuración de haproxy se encuentra en */etc/haproxy/haproxy.cfg*.

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica3/img/haproxy1.PNG)

De nuevo en la máquina 3 se aprecia como al realizar el *curl* sobre el balanceador con haproxy, va devolviendo diferentes mensajes dependiendo de la máquina final a la que llega.


### III Someter a la granja web a una alta carga, generada con la herramienta Apache Benchmark, teniendo primero nginx y después haproxy

Para esta parte hace falta tener instalado Apache Benchmark, disponible en el pack de apache, en la máquina vfernandez3, ya que es la que usamos para realizar las peticiones a cada balanceador. Una vez instalado, procedemos a usar el siguiente comando para realizar el benchmark sobre cada uno de los balanceadores.

*ab -n 1000 -c 10 http://IP_DEL_BALANCEADOR/index.html*

**Con nginx**

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica3/img/nginx_bench.PNG)

**Con haproxy**

![img](https://github.com/vicferpoy/SWAP18/blob/master/practica3/img/haproxy_bench.PNG)


### Conclusión

Observando los resultados obtenidos podemos ver que *haproxy* ha tardado 0.1 segundos menos que *nginx* en realizar el test al completo, usando los mismos parámetros. Siendo tan sólo 1000 peticiones, es un tiempo notable puesto que en servidores con mucha mayor concurrencia de peticiones, esa décima va a ser sustancial, por lo que yo personalmente me quedaría con *haproxy* si tuviera que elegir entre estos dos.
