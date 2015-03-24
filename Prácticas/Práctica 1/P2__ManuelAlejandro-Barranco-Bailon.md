#Práctica 2. Clonar información de un sitio web

En esta práctica 2, nos centramos en los distintos procesos existentes para clonar información del servidor web de producción (principal) a un servidor web secundario de respaldo.

##Copia de archivos por SSH

Uno de los métodos que tenemos para copiar archivos de una máquina a la otra es mediante el protocolo y servicio **SSH**.

En nuestro caso, lo que se ha hecho es comprimir la carpeta */var/www* y por SSH, copiarla a la carpeta principal del usuario de la máquina secundaria. El comando utilizado ha sido: *tar czf - /var/www | ssh usuarioMaquinaDestino@IPDestino 'cat > ~/copia.tar.tg'*. El proceso podemos verlo en las siguientes capturas de pantalla:
![img]()
![img]()

##Clonado de una carpeta con la herramienta rsync
Esta herramienta de sincronización no fue necesario instalarla en mi caso puesto que ya lo estaba. Con esta herramienta lo que realicé fue clonar la carpeta */var/www* desde la máquina principal en la secundaria. Desde la máquina **secundaria** y como usuario **root** se ejecutó el siguiente comando: *rsync -avz -e ssh root@IPMaquinaPrincipal:/var/www/ /var *. Con esta pequeña variación respecto al comando que viene en el guión, se actualiza la carpeta */var/www* de la máquina secundaria con la */var/www* de la principal. (*Si hubiese puesto el comando que viene en el guión, rsync -avz -e ssh root@IPMaquinaPrincipal:/var/www/ /var/www/, se hubiera creado en la máquina secundaria dentro de /var/www/ una nueva carpeta www de la forma /var/www/www.*) Durante la ejecución del comando, nos pedirá la clave para realizar la conexión SSH.
El proceso podemos verlo en las siguientes capturas de pantalla:
![img]()
![img]()

También en el guión se da el comando que permite borrar aquellos ficheros que se han borrado en la máquina principal, así como excluir de la copia los que no estamos interesados en copiar.

##Configuración de SSH sin solicitud de contraseña
Para que el proceso de copia anterior con rsync se pueda automatizar, necesitaremos que no pida la contraseña para la conexión SSH. Para ello se utilizan las claves pública-privada. Mediante ssh-keygen podemos generar la clave. Con el comando *ssh-keygen -t dsa* en la máquina secundaria obtendremos la clave que necesitamos.(*Esta clave generada se asocia con el usuario remoto que queramos y con el usuario local que ha ejecutado la generación de clave. Es decir si la genero con el usuario local alex, posteriormente el comando rsync para que sea automático debería ejecutarlo alex*). Durante la ejecución del comando, nos pedirá un *passphrase*. Si queremos que se conecte sin contraseña, deberemos no poner nada y dejarlo en blanco.
![img]()

Ahora copiamos la clave generada al equipo remoto dónde nos conectaremos sin contraseña. Usamos el comando: *ssh-copy-id -i .ssh/id_dsa.pub root@IPMaquinaPrincipal*

En mi caso, ahora ya si que podrá conectarse el usuario alex de la máquina secundaria como usuario root en la máquina principal. Lo comprobamos:
![img]()
![img]()

Y ejecutar comandos:
![img]()

##Programar la copia con cron
Ahora ya podemos conectarnos via SSH sin contraseña para automatizar la tarea de rsync. Para ello editamos el fichero */etc/crontab/* (como root) que mediante cron ejecuta las tareas allí programadas automáticamente. En la siguiente imagen podemos ver la configuración para que como usuario alex (el que generó las claves pública-privada y que por tanto será el que puede establecer conexión como root via SSH sin contraseña) se ejecute la instrucción de rsync para que las carpetas */var/www/* de ambas máquinas se sincronicen cada comienzo de hora. Lo vemos en la siguiente imagen ( la última línea: 00 * * * * alex rsync -avz -e ssh root@172.16.125.130:/var/www /var/www/ ):
![img]()

Y en la siguiente imagen vemos como están actualizadas ambas carpetas (para la comprobación en el momento de que funcionaba la orden del crontab, puse que se ejecutara cada minuto, con la línea * * * * * alex rsync -avz -e ssh root@172.16.125.130:/var/www /var/www/ simplemente cambio 00 del primer campo por *):
![img]()


