
# Docker Basico

<!-- TOC -->

- [Docker Basico](#docker-basico)
    - [A. Que es Docker](#a-que-es-docker)
    - [B. Instalando Docker](#b-instalando-docker)
    - [C. Conceptos antes de arrancar](#c-conceptos-antes-de-arrancar)
    - [D. Imagenes](#d-imagenes)
    - [E. Ejecutando Imagenes (Containers)](#e-ejecutando-imagenes-containers)
    - [F. Ejecutando Imagenes (parte 2)](#f-ejecutando-imagenes-parte-2)
    - [G. Ejecutando imagenes](#g-ejecutando-imagenes)
    - [H. Ficheros para crear una Imagen](#h-ficheros-para-crear-una-imagen)
    - [I. Volumenes de datos](#i-volumenes-de-datos)
    - [J. Supervisor](#j-supervisor)
    - [K. Creando un servicio de SO](#k-creando-un-servicio-de-so)

<!-- /TOC -->

## A. Que es Docker


Docker es una herramienta open-source que nos permite realizar una ‘virtualización ligera’, con la que poder empaquetar entornos y aplicaciones que posteriormente podremos desplegar en cualquier sistema que disponga de esta tecnología.

La gran diferencia es que una máquina virtual necesita contener todo el sistema operativo mientras que un contenedor Docker aprovecha el sistema operativo sobre el cual se ejecuta, comparte el kernel del sistema operativo anfitrión e incluso parte de sus bibliotecas.

![Image](../resources/HJEThXHoz_rJ_evzcCf.png)



## B. Instalando Docker

En mi caso instalare docker en un Archlinux, por lo cual hare uso del comando "pacman" y el nombre de la paquete que se encuentra en el repositorio comunitario.

```bash
pacman -S docker
```

Para que el servicio se active cuando la maquina se inicie, habilitamos el servicio "docker"

```bash
systemctl enable docker
```

Iniciamos el servicio con el comando 
```bash
systemctl start docker
```

Validamos su estado
```bash
systemctl status docker
```

Para no ejecutar el comando docker como administradores, modificamos el usuario con el cual trabajemos y le colocamos el grupo "docker", donde $USER es el usuario con el cual voy a trabajar

```bash
usermod -aG docker $USER
```

## C. Conceptos antes de arrancar

Antes de iniciar a utilizar docker, es importante contar con dos conceptos claves los cuales son Imagenes (Image) y Contenedores (Container)

**Imagen**
Una imagen de Docker, es una estructura de directorios y paquetes mínima, creada para una función básica. Se trata de que sea una plantilla, que se puede modificar, pero que ha sido pensada para una uso básico y específico. Por ejemplo, una imagen Debian con un Apache. De esta forma, para tener múltiples servidores web apache, cada uno encapsulado y aislado de los demás, sólo tendremos que crear contenedores desde ella, y cada uno será un servidor web distinto, con su propia IP, al que se puede configurar de manera independiente y que a su vez de puede clonar o mover donde se quiera.


**Contenedor**
Un contenedor, es una instancia generada desde una imagen, que ya es en sí un pequeño OS funcionando, y que podemos usarlo para una función concreta (como por ejemplo correr un Apache, un servidor Java).

Por eso en Docker se trabamos primero con  imágenes (las plantillas) y luego con contenedores (los minisistemas operativos funcionando en nuestra maquina). 


**En otras palabras**
Puedo tener una imagen de "ubuntu", el cual es basicame un conjunto de paquetes y archivos pero al ejecutarse se inicia un contenedor. 


## D. Imagenes

Para ver el listado de las imagenes

```bash
[jgaviria@kike-labs]: ~>$ docker images
```

En este momento cuento con las imagenes

![Image](../resources/B17ns-7fX_H14Z2WXMX.png)

**Con las imagenes puedo** 

1. Sacar un backup de una imagen entera y llevarla a un TAR

Salve la imagen "openjdk" y llevela a un archivo con nombre "openjdk.tar"

```bash
docker image save hello-world > wello-world.tar
```

2. Si una imagen ya no la quermeos, podemos eliminarla con el comano 

```bash
docker rmi <image_id>
```

Por ejemplo eliminemos la imagen hello-world

```bash
docker rmi e38bc07ac18e
```

3. Importemos la imagen que salve, para lo cual ejecutamos 
```bash
docker image load < hello-world
```

## E. Ejecutando Imagenes (Containers)

Para ejecutar un comando de una imagen, por ejemplo 
echo "Hola mundo "

Ejecutamos
```bash
#Sintaxis : docker run <imagen> <comando>
docker run debian echo "Hola"
Hola
```

Para ejecutar y que se quede esperando linea de comando, utilizo los parametros -i (interactivo) -t (con tty)
```bash
docker run -i -t ubuntu --name "NombreContenedor"
```
<Para salir solo ejecuto el comando exit>

Cuando una imagen se esta ejecutando (container), veamos que se esta ejecutando con el comando "ps"
```bash
docker ps
```

Con un container en ejecución podemos ver que se esta ejecutando dentro de el, para lo cual ejecutamos
```bash
docker top <CONTAINER_ID>
```

Si deseo detener el container puedo ejecutar 
```bash
docker stop <CONTAINER_ID>
```


Tener en cuenta que el comando "ps " sin parametros solo me mostrara los containers que estan en ejecución, pero si deseo verlos todos debo ejecitar (Mostrandome todos los containers que he tenido en la historia)

```bash
docker ps -a
```

Cuando un container ya se encuentra abajo, podemos eliminarlo, con el comando 
```bash
docker rm <CONTAINER_ID>
```

Asi si hemos tenido mucho trabajo en nuestras ejecuciones de docker la lista de ps -a sera mas larga, para eliminarlas todas 
```bash
docker rm `docker ps -aq`
o 
docker rm $(docker ps -aq)
```

Ahora para ver las estadisticas de un container puedo ejecutar el comando 
```bash
docker stats <CONTAINER_ID>
```

Si ejecutar "docker stats " sin el id de container, me presentara las estadisticas de todos los containers en ejecución.

Por ejemplo es chevere ver cuanta memoria usa una imagen vs otra (por ejemplo la de busybox es una cosa de nada).

Dado que un container es la ejecución de una imagen tantas veces como sea necesaria y cada container usar su propia memoria/red/procesador, vamos a entender el ciclo de vida de un container

- Created : Cuando tengo una image y ejecuto esta imagen (run), el contenedor se crea y se le asocia internamente un CONTAINER_ID, vamos a realizarlo con alpine
```bash
docker run -it --name "prueba1" alpine
```

![Image](../resources/Hk7uPpQMQ_BJy9caXGQ.png)

- Vemos los procesos corriendo 
```bash
docker ps
```

![Image](../resources/Hk7uPpQMQ_HyVJsp7fm.png)

- Ahora lo pausaremos, con el comando (la ejecución de la maquina se detendra) 
```bash
#Sintaxis docker pause <CONTAINER_ID>
docker pause 6b791a31097c
```

- Ahora lo reanudamos
```bash
#Sintaxis docker unpause <CONTAINER_ID>
docker unpause 6b791a31097c
```

- stop / restart : Puedo pausar o reanudar un container para esto supongamos que ejecutamos el siguiente comando  en un container 

Puedo detener un container, mediante el comando stop, el cual tambien sale y detiene los proceso, con el comando 
```bash
#Sintaxis docker stop <CONTAINER_ID>
docker stop 6b791a31097c
```

Una vez detenido (stop) o matado (kill) Cual es su diferencia

STOP envia un "SIGKILL" que puede ser capturada e interpretada o ignorada por el proceso. Esto permite que el proceso realice la terminación agradable que libera recursos y que ahorra el estado si es apropiado.

KILL envia una señal SIGTERM se envía a un proceso para solicitar su terminación de forma forzosa.

Ahora para reiniciarlo ejecutamos
```bash
#Sintaxis docker restart <CONTAINER_ID>
docker restart 6b791a31097c
```

Para volver a entrar a la consola (si esta corriendo en modo interactivo), Si queremos acceder a un container que se encuentra en ejecución, lo hacemos con el comando "attach", por ejemplo acceder a la linea de comando 
```bash
#Sintaxis docker attach <CONTAINER_ID> 
docker attach 105c44970025
```

## F. Ejecutando Imagenes (parte 2) 

Y nos dejara nuevamente en la linea de comando del container en ejecución. Pero ojo utilizar el comando "exit", mataria nuestro container, para salir sin cerrar ejecutamos 

CTRL + P + CTRL + Q

Ahora bien, supongamos que nuestro container esta en ejecución pero no deseamos acceder a este para ejecutar un comando, podemos hacer uso del comando "exec" que es ejecutar algo en nuestro contenedor sin detenerlos

por ejemplo 
```bash
#Sintaxis docker exec <CONTAINER_ID> <COMANDO>
docker exec 105c44970025 ls
```

Pero si quermos como sea entrar en la linea de comando y ejecutar el "exit" sin necesidad de cerrarlo ejecutar "exec" con -i (interactivo) -t (tty) y el comando sera un "bash" 

```bash
[jgaviria@kike-labs]: ~>$ docker exec -i -t 105c44970025 bash
root@105c44970025:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@105c44970025:/# exit
exit
```


Y nuestro container aunque ejecutamos el "exit" sigue en ejecución.
```bash
[jgaviria@kike-labs]: ~>$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
105c44970025        ubuntu              "/bin/bash"         About an hour ago   Up 2 minutes                            cranky_kalam

```



Haciendo cambios en un container


Sobre esta ejecucion de container puedo hacer todos los cambios que desee pero estaran solo en el container pero no modificar la imagen, por ejemplo podemos hacer 

mkdir -p /opt/java /opt/server

Creando los directorios en el container, para ver que cambios se ha realizado desde el container a la imagen ejecuto el comando 

```bash
#Sintaxis sudo docker diff <ID_CONTAINER>
sudo docker diff 7e6929f9f19f
```

para este caso 


```bash
[jgaviria@argos ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
7e6929f9f19f        debian:latest       "/bin/bash"         About a minute ago   Up About a minute                       pensive_wright
```

```bash
[jgaviria@argos ~]$ sudo docker diff 7e6929f9f19f
C /opt
A /opt/java
A /opt/server
```

C (Modifica) A (Adicionar)

En caso que salga del container (exit) y vuelvo a ejecutar run , se crea un nuevo container id sin los cambios que habia hecho, si deseo mantenerlos tendria que construir una "imagen" (esta forma construye una imagen de una forma que Docker no lo hace, ya que se deberia hacer como un Dockerfile), pero vamos a hacerlo solo por probar.

```bash
docker commit 7e6929f9f19f mi-imagen
sha256:95e6620cea2f0082038aee6252a2ac00413345402490927a7a647d303e98ffb5
```

Ahora al ver mis imagenes 

```bash
jgaviria@kike-labs]: ~>$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mi-imagen           latest              95e6620cea2f        31 seconds ago      130MB

```

Y si llego a correrla ya puedo ver los cambios a los cuales he hecho commit

Si deseo eliminar esta imagen, siempre puedo hacer un "rmi" (eliminar imagen)

```bash
docker rmi mi-imagen
```


**OJO : Para crear imagenes de verdad es mejor hacerlo a base de un archivo para crear imagenes (dockerfile).**

Para correr el proceso en segundo plano podemos llamar el comando run con el argumento -d
```bash
docker run -d alpine
```

Para ver que se esta presentando o logs de este contenedor con el comando 
```bash
docker log <CONTAINER_ID>
```

## G. Ejecutando imagenes

En ciertas ocasiones necesitamos iniciar un servidor y permitir que nuestro servidor exponga un puerto, por ejemplo vamos a utilizar una imagen que cuenta con un servidor SSH (openssh-server), el cual para subirlo en la maquina se inicia con el comando 
```bash
#El parametro -D indica que se sube como demonio 
/usr/sbin/sshd -D 
```

Este proceso inicia el puerto 22 para recibir peticiones, ahora lo ejecutaremos con docker como un proceso demonio
```bash
#La opcion -d (Es de Docker para informar que es un demonio) 
#La opcion -p indica el puerto que sera expueto
docker run -d -p 22 cursodocker/alpine /usr/sbin/sshd -D 
```


Nuestro procesos se inicia y al validarlo con 
```bash
jgaviria@argos [~] $ docker ps
CONTAINER ID        IMAGE                COMMAND               CREATED             STATUS              PORTS                   NAMES
f5ff655b4160        cursodocker/alpine   "/usr/sbin/sshd -D"   7 seconds ago       Up 5 seconds        0.0.0.0:32768->22/tcp   keen_mcnulty
```

Nos indica que este imagen se encuentra corriendo y que en nuestra maquina el puerto "32768" sera enviado al puerto 22 mediante la tarjeta de red que docker define (comando ip -a)
```bash
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:dc:3d:98:a1 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:dcff:fe3d:98a1/64 scope link 
       valid_lft forever preferred_lft forever

```

La cual se conoce como "docker0"

Puedo iniciar tantas instancias como quiero, y veremos que utilizara puertos diferentes 

![Image](../resources/B1USvgEM7_HkqdaZ4f7.png)

Ojo en el parametro -p puedo indicar el puerto que yo quiero usar, por ejemplo 

```bash
#Sintaxis
docker run -d -p <MIPUERT>:<PUERTO_SERVICIO> cursodocker/alpine /usr/sbin/sshd -D
#Como quedario 
docker run -d -p 5000:22 cursodocker/alpine /usr/sbin/sshd -D
docker run -d -p 5001:22 cursodocker/alpine /usr/sbin/sshd -D
docker run -d -p 5002:22 cursodocker/alpine /usr/sbin/sshd -D
```

Para observar cual es la metadata de una imagen (como esta corriendo, direcciones IP, memoria.... ), usamos el comando 
```bash
#Sintaxis
docker inspect <ID_CONTAINER>
```

## H. Ficheros para crear una Imagen

Los ficheros nos permiten definir los pasos para realizar la creación de una imagen, es decir todos los pasos que hicimos mediante el modo interactivo, lo definiremos dentro de un archivo 

Para esto crearemos un archivo con el nombre "Dockerfile" el cual contendra

FROM : A partir de cual imagen vamos a trabajar
LABEL : Permite adicionar información a la metadata para informar cosas como el autor, la version ... lo que sea
RUN : Permite ejecutar comando dentro de la imagen
ENV : Define una variable de entorno para la imagen, estas variables quedan disponibles al ejecutar un comando linux "env"
EXPOSE : Le infoma a Docker que esta imagen tendra un puerto expuesto
CMD : Hace que se lance un programa, el cual seria el programa a exponer 
ADD : Adicionar un archivo a la imagen, para lo cual copia un archivo de la maquina en la que se hace la imagen al directorio destino

Por ejemplo, vamos a crear la imagen de openssh que teniamos antes pero con el archivo 
```bash
FROM alpine:latest
LABEL Autor Jaime Gaviria (jgavria@lucasian.com)
LABEL version 1.0
LABEL cualquier-llave valor

#Instalamos paquetes necesarios
RUN apk update
RUN apk upgrade
RUN apk add openssh 

#Creamos el usuario y el grupo 
RUN adduser student -D
RUN echo student:student | chpasswd

ADD /tmp/archivoLocal.txt /tmp/archivoEnImagen.txt

RUN mkdir /var/run/sshd

ENV KEYGEN="ssh-keygen -b 2047 -t rsa -f "
RUN ${KEYGEN} /etc/ssh/ssh_host_rsa_key -q -N ""
RUN ${KEYGEN} /etc/ssh/ssh_host_dsa_key -q -N ""
RUN ${KEYGEN} /etc/ssh/ssh_host_ecdsa_key -q -N ""
RUN ${KEYGEN} /etc/ssh/ssh_host_ed25519_key -q -N ""

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]



```

Para ejecutar esta imagen, ejecutamos el comando
```bash
#Sintaxis
docker build -t [NombreImage] .

docker build -t jgaviria/ssh .
```

Ahora ya veo mi imagen y la puedo lanzar
```bash
docker run -d -p 5001:22 jgaviria/ssh
```


Y validamos que esta corriendo 
```bash
jgaviria@argos [~] $ docker ps
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                  NAMES
f7e07ea781c2        jgaviria/ssh        "/usr/sbin/sshd -D"   2 minutes ago       Up 2 minutes        0.0.0.0:5001->22/tcp   keen_chaplygin

```

Cualquiera de las variables puedo modificarla, cuado ejecuto la imagen 
```bash
docker run -d -p 5001:22 -e NOMBRE_VARIABLE=NuevoValor   jgaviria/ssh
```
Puedo pasar un archivo con las variables, mediante el parametro --env-file
```bash
docker run -d -p 5001:22 --env-file NombreArchivo jgaviria/ssh
```

El uso de variables es importante, siempre y cuando el los archivos del aplicativo haga uso de estas variables, utilizando 
${NOMBRE_VARIABLE}


## I. Volumenes de datos

Los voumenes de datos son directorios que se pueden compartir entre contenedores, para esto utilizaremos el argumento -v cuando iniciemos. 

Para hacer uso de esto simplemente haremos uso de la imagen httpd (https://hub.docker.com/_/httpd/), la cual es un servidor http de Apache. 

Esta cuenta con un directorio en el cual estan todas las paginas web (htdocs) y estan en 
```bash
/usr/local/apache2/htdocs
```

Ahora lo que haremos es crear un directorio en nuestra maquina con las paginas que queremos presentar, por esto cree un directorio /webapps en mi maqina y en este coloque un archivo index.html

Para llamar la imagen simplemente ejecuto 

```bash
#Sintaxis 
#-v [Directorio de la maquina]:[Directorio en la imagen]:[:rw|ro]
#rw : Permisos de escritura/lectura
#ro : Read-only
#Tener en cuenta que la tuta del directorio de maquina me funciono con la ruta completa

docker run -P -d --name httpd-dir1 -v /data/jgaviria/estudio/docker/directorio-compartido/webapps:/usr/local/apache2/htdocs/:rw httpd
```

Ahora al accerder al navegador y habiendo validado el puerto entregado (-P entrega puerto dinamicamente).

```bash
jgaviria@argos [/data/jgaviria/estudio/docker/directorio-compartido] $ docker ps
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                   NAMES
88b4d7220a6f        httpd               "httpd-foreground"   11 seconds ago      Up 9 seconds        0.0.0.0:32774->80/tcp   http-1

```
vemso la pagina 

![Image](../resources/rJYCAGYGX_ryExYHW7m.png)

La mejor parte es que si llegamos a realizar cambiso sobre el contendio de la carpeta, esto se veran reflejados ya que el servidor httpd esta viendo esta carpeta 

Ahora dado que ya cuento con un volumen montado para el contenedor "http-1", puedo inciar otro contenedor y decirle que los volumenes se monten igual que otro contenedor, esto mediante el argumento "--volumes-from"

```bash
docker run -d -P --volumes-from http-1 --name http-2 httpd
```

Ahora ya tengo dos contenedores con los mismos volumenes
```bash
jgaviria@argos [/data/jgaviria/estudio/docker/directorio-compartido] $ docker ps 
CONTAINER ID        IMAGE               COMMAND              CREATED              STATUS              PORTS                   NAMES
1fec2c06e100        httpd               "httpd-foreground"   40 seconds ago       Up 38 seconds       0.0.0.0:32777->80/tcp   http-2
696fda388390        httpd               "httpd-foreground"   About a minute ago   Up About a minute   0.0.0.0:32776->80/tcp   http-1
```

## J. Supervisor
Esto no hace parte de las recomendaciones propias en docker, dado que el principio de las imagenes es tener uno y solo un servicio por cada una de ellas. 

Pero a veces y solo a veces se requiere contar con mas de un componente iniciado en el mismo contenedor, por ejemplo iniciar un contenedor basado en una imagen que tenga mysql y apache.

Para esto hacemos uso del componente supervisor el cual se tomara desde http://supervisord.org/ o bien mediante la descaga del paquete en la imagen (dockerfile)

Para este ejemplo haremos una imagen que cuente con un servidor apache y openssh (para poder acceder), para crearlo paso a paso vamos a realizar cada paso en un contenedor interactivo para ir construyendo nuestro Dockerfile.

1. Corremos una imagen ubuntu de forma interactiva para ir validando los pasos.
```bash
docker run -it ubuntu
```

2. Instalamos apache2, openssh-server,  supervisor y sudo (para hacer el sudo) y creamos directorio locales

```bash
root@e8eacec49ac4:/# apt get update && apt install -y apache2 openssh-server supervisor sudo 
root@e8eacec49ac4:/# mkdir -p /var/lock/apache2 /var/run/apache2 /var/run/sshd /var/log/supervisor
```

3. Agregamos el usuario jgaviria y le definimos la contraseña "manager2018" y colocamos este usuario como sudoer (permiso para hacer root)

```bash
root@e8eacec49ac4:/# useradd jgaviria
root@e8eacec49ac4:/# echo "jgaviria ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
root@e8eacec49ac4:/# echo 'jgaviria:manager2018' | chpasswd
```


Con estos pasos podemos decir que ya tenemos dos servicios creado y nuestro Dockerfile se veria de la sigiente forma 

```bash
FROM ubuntu:latest
LABEL Autor Jaime Gaviria (jgavria@lucasian.com)
LABEL version 1.0

#Instalamos apache2, openssh-server y supervisor
RUN apt get update && apt install -y apache2 openssh-server supervisor sudo 
RUN mkdir -p /var/lock/apache2 /var/run/apache2 /var/run/sshd /var/log/supervisor

#Creamos el usuario jgaviria (el cual voy a colocar en el grupo Root NO HACER ESTO ) 
#y lo colocamos como usuario admin
RUN useradd jgaviria
RUN echo "jgaviria ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
RUN echo 'jgaviria:manager2018' | chpasswd

#Para hacer uso de supervisor debemos tener el archivo de configuración supervisor.conf
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

#Definos los puetsos que se van a exponer
EXPOSE 22 80

#Solicitamos que se ejecute el servicio de supervisor, el cual inicia los servi
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

```

Ahora para poder utilizar supervisor debemos crear un archivo que defina los servicios que vamos a iniciar para lo cual creamos en nuestra maquina un archivo supervisord.conf  con la configuración de los servicio

```bash
[supervisord]
nodaemon=true
logfile=/var/log/supervisor/supervisord.log
childlogdir=/var/log/supervisor
logfile_maxbytes=10MB

[program:sshd]
command=/usr/sbin/sshd -D

[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND"
```

Dentro de esta configuracón encontramos 
nodaemon : No inicie como demonio ya que correra desde el CMD  del Dockerfile
logfile : El log de supervisord
childlogdir : Los logs de los servicios dejelos en esta ruta
logfile_maxbytes : El tamaño maximo de de los logs

program:NombreServicio : Define cada uno de los servicios y dentro de estos esta el "command" con el cual se inicia el servicio


ahora vamos a construir nuestra imagen con los archivos creados 
```bash
docker build -t sshapache .
```

Ahora iniciamos un contener para esta imagen
```bash
docker run -d -P --name sshapache-1 sshapache
```

Recordemos que el argumento -P realiza asignacion automatica de puertos, pero si deseo definir los puertos yo mismo defimos varios -p para definir puerto a puerto como ser realizara la asignación

```bash
docker run -d -p 80:80 -p 2222:22 --name sshapache-1 sshapache
```

Y al ver los procesos 
```bash
jgaviria@argos [~] $ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                          NAMES
97b5e90ef467        sshapache           "/usr/bin/supervisord"   2 minutes ago       Up 2 minutes        0.0.0.0:32779->22/tcp, 0.0.0.0:32778->80/tcp   sshapache-1

```

Al acceder al container por ssh (puerto 32779) vemos el directorio de /var/log/supervisor
```bash
root@97b5e90ef467:/var/log/supervisor# ls -l
total 8
-rw------- 1 root root 172 Jul  9 22:38 apache2-stderr---supervisor-aXsUxO.log
-rw------- 1 root root   0 Jul  9 22:38 apache2-stdout---supervisor-M52pPO.log
-rw------- 1 root root   0 Jul  9 22:38 sshd-stderr---supervisor-YxfLyD.log
-rw------- 1 root root   0 Jul  9 22:38 sshd-stdout---supervisor-dNqePx.log
-rw-r--r-- 1 root root 832 Jul  9 22:39 supervisord.log
```


## K. Creando un servicio de SO

Normalmente en Linux contaremos con systemd como estandar para el inicio de servicios de Linux, el objetivo de esta sección es crear en la maquina anfitrion un servicio para iniciar un contenedor cada vez que la maquina se reinicie, para lo cual creamos 

Para esto crearemos un archivo para el servicio dentro de /etc/systemd/system/, para este caso voy a crear el archivo 

 /etc/systemd/system/sshapache.service 
 
 Para registrar un servicio que inicie automaticamente un container para la imagen sshapache
 
```bash
[Unit]
Description=SSH Apache Docker 
Requires=docker.service network-online.target
After=docker.service network-online.target

[Service]
Restart=always
ExecStart=/usr/bin/docker run --rm -p 80:80 -p 2222:22 --name sshapache-1 sshapache
ExecStop=/bin/docker stop sshapache-1  


[Install]
WantedBy=multi-user.target  
```

Ahora simplemente recargamos los servicios con el comando 

```bash
sudo systemctl daemon-reload
```

Y habilitamos el servicio para que se inicie automaticamente

```bash
jgaviria@argos [~] $ sudo systemctl enable sshapache
Created symlink /etc/systemd/system/multi-user.target.wants/sshapache.service → /etc/systemd/system/sshapache.service
```

Podemos reiniciar la maquina o bien iniciarlos manualmente con 
```bash
jgaviria@argos [~] $ sudo systemctl start sshapache
```

Ahora veamos que ya se encuentra acivo

```bash
jgaviria@argos [~] $ sudo systemctl status sshapache
● sshapache.service - SSH Apache Docker
   Loaded: loaded (/etc/systemd/system/sshapache.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2018-07-09 18:31:11 -05; 4s ago
 Main PID: 7171 (docker)
    Tasks: 10 (limit: 4915)
   Memory: 8.8M
   CGroup: /system.slice/sshapache.service
           └─7171 /usr/bin/docker run --rm -p 80:80 -p 2222:22 --name sshapache-1 sshapache
```

Y mediante docker veamos el proceso corriendo

```bash
jgaviria@argos [~] $ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                      NAMES
5094c5b36e61        sshapache           "/usr/bin/supervisord"   11 seconds ago      Up 9 seconds        0.0.0.0:80->80/tcp, 0.0.0.0:2222->22/tcp   sshapache-1
```

Si lo llego a detener 

```bash
sudo systemctl stop sshapache
```

Gracias a la opción --rm  en el start  
```bash
ExecStart=/usr/bin/docker run --rm -p 80:80 -p 2222:22 --name sshapache-1 sshapache
```

El proceso no quedara en el listado de ps -a (donde quedaria registrado)
```bash
jgaviria@argos [~] $ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
jgaviria@argos [~] $ docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```











