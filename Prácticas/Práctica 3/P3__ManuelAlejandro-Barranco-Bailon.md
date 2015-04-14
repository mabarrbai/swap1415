#Práctica 3. Balanceo de carga

En esta práctica 3, nos centramos en la configuración de un balanceador que reparta la carga de trabajo (HTTP) entre los distintos servidores de una granja web.

El escenario sobre el cual se han realizado los siguientes ejercicios consiste en dos servidores web finales con Apache (forman la granja web), y delante de estos se coloca un servidor (sin Apache, con SSH, nginx y haproxy) que será el que atienda las peticiones externas de clientes web y las balancee en los servidores finales.

##Balanceo con nginx

El primer balanceador que vamos a instalar, configurar y poner en funcionamiento será **nginx**.

Para su instalación, primero importamos la clave del repositorio que lo contiene, y a continuación añadimos el repositorio (editando el fichero /etc/apt/sources.list).

Ahora ya sí podemos instalar nginx con *apt-get update* y *apt-get install nginx*.

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-03-24%2018:46:18.png)

Una vez instalado, vamos a configurarlo. Para ello, una de las tareas que habrá que hacer será especificarle a nginx los servidores finales y englobarlos en un grupo. 
Para realizar la configuración editamos el fichero */etc/nginx/conf.d/default.conf*.
Con la directiva upstream, le indicamos el grupo de servidores web finales:

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-03-24%2019:08:09.png)

Otros aspectos a destacar en la configuración son indicar que la versión de HTTP es la 1.1 y eliminar la cabecera Connection para que no lleguen a los servidores las cabeceras provenientes de los clientes.

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-03-24%2019:08:39.png)

Una vez realizada la configuración, guardamos los cambios en el fichero y reiniciamos el servicio ngninx con *sudo service nginx restart*.

Ya podemos comprobar si se está realizando el balanceo correctamente. Para diferenciar que nos contestan ambas máquinas, el index.html de cada una de ella especifica desde que máquina se está contestando.

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-03-24%2019:18:05.png)

Le enviamos peticiones desde el cliente externo con *curl*(en mi caso, la máquina anfitriona), al servidor que actua de balanceador y vemos cómo nos redirige la petición hacia uno u otro servidor final. En esta primera configuración se realiza el balanceo mediante **round-robin** como se puede comprobar en la siguiente imagen.

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-03-24%2019:20:01.png)

Podemos hacer que se balancee según una asignación de pesos a los distintos servidores. Esto lo indicamos con el modificador *weight* al que asignamos un valor numérico que indica la carga que le asignamos.

En mi ejemplo, asigné un peso de 2 a la máquina principal *Granada*, lo que indica que de 3 peticiones, Granada atenderá 2, y Milan(la secundaria) atenderá 1.

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-03-24%2019:21:53.png)

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-03-24%2019:23:19.png)

A continuación se describirá el proceso de balanceo con haproxy. Para ello, deberemos apagar nginx con *service nginx stop*.


##Balanceo con haproxy

El primer paso será instalar haproxy con el comando *sudo apt-get install haproxy*.

Ahora vamos a configurarlo mediante el fichero */etc/haproxy/haproxy.cfg*. Además de otros ajustes(especificados en el guión, los más importantes será especificar los servidores finales con sus respectivas IP's escuchando por el puerto 80. Tambien mencionar que la máquina balanceadora escuchará por el puerto 80. Lo vemos en el parametro *bind* en *frontend http-in*.

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-04-07%2018:53:59.png)

Guardamos los cambios y arrancamos el servicio haproxy con la orden **sudo /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg**.

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-04-07%2018:58:10.png)

De nuevo, nos disponemos desde la máquina cliente ( en mi caso la máquina anfitriona, a realizar peticiones web con el comando *curl* a la IP de la máquina balanceadora, y ésta nos redirigirá a alguno de los servidores finales. En este primer ejemplo, se sigue el algoritmo de reparto **round-robin**.

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-04-07%2019:02:45.png)

Al igual que hicimos en nginx, le vamos a asignar distintos pesos a los servidores finales con el modificador *weight*. La máquina Granada recibirá 2 de cada 3 peticiones hechas al balanceador, y Milan 1.

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-04-07%2019:12:00.png)

![img](https://github.com/mabarrbai/swap1415/blob/master/Pr%C3%A1cticas/Pr%C3%A1ctica%203/imagen/Captura%20de%20pantalla%20de%202015-04-07%2019:12:51.png)








