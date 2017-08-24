# COMANDOS DOCKER

## Instalar imagenes

Lista imagenes docker

``docker images``

Instala una imagen docker, aqui instalamos dos una imagen de postgres y otra de centos, siempre será en su última versión LATEST si no le agregas la versión como por ejemplo: centos:6

``docker pull postgres``

``docker pull centos``

Lista informaciones de docker es interesante cambiar el tipo de controlador de almacenamiento (Storage Driver): aufs para overlay2 porque es más rápido

``docker info``

## Redes

Lista redes de docker

``docker network ls``

Si listamos con ifconfig las configuraciones de ip veremos que se crea por default una interface para conectarse mediante NAT

``sudo ifconfig``

```bash
*docker0*   Link encap:Ethernet  direcciónHW 0x:xX:Xx:xx:x5:45
Direc. inet:172.10.0.1  Difus.:0.0.0.0  Másc:255.255.0.0
ACTIVO DIFUSIÓN MULTICAST  MTU:1500  Métrica:1
Paquetes RX:0 errores:0 perdidos:0 overruns:0 frame:0
Paquetes TX:0 errores:0 perdidos:0 overruns:0 carrier:0
colisiones:0 long.colaTX:0
Bytes RX:0 (0.0 B)  TX bytes:0 (0.0 B)
```

Para crear una nueva red 

``docker network create --driver bridge redlocal``

Cuando listes las redes de docker aparece tu última red

``docker network ls``

```
NETWORK ID          NAME                DRIVER              SCOPE
a28ecacbbeb8        bridge              bridge              local               
bfdba3c8d9d2        host                host                local               
7d1495893dd1        none                null                local               
**06356c8a86db        redlocal            bridge              local**
```

Ahora si listas con ifconfig deberia aparecer tu ultima red creada es importante saber que esta nueva red no se podrá comunicar con docker0

``sudo ifconfig -s``

```
Iface MTU Met RX-OK RX-ERR RX-DRP RX-OVR TX-OK TX-ERR TX-DRP TX-OVR Flg
**br-06356c8a86db  1500 0         0      0      0 0             0      0      0      0 BMU**
docker0    1500 0         0      0      0 0             0      0      0      0 BMU
enp0s25    1500 0         0      0      0 0             0      0      0      0 BMU
lo        65536 0     35176      0      0 0         35176      0      0      0 LRU
```

Si quieres ahora puedes ver que docker creara una nueva IP para conectarse via NAT

`` sudo ifconfig br-06356c8a86db ``

***
> **br-06356c8a86db** Link encap:Ethernet  direcciónHW 0x:cc:xx:cc:xx:2c  
          Direc. inet:**172.22.0.1**  Difus.:0.0.0.0  Másc:255.255.0.0
          ACTIVO DIFUSIÓN MULTICAST  MTU:1500  Métrica:1
          Paquetes RX:0 errores:0 perdidos:0 overruns:0 frame:0
          Paquetes TX:0 errores:0 perdidos:0 overruns:0 carrier:0
          colisiones:0 long.colaTX:0 
          Bytes RX:0 (0.0 B)  TX bytes:0 (0.0 B)
***

## Ejecutar imagen

Para seleccionar un contenedor y ejecutar una imagen de forma interactiva (-ti = terminal interactivo)

`` docker run -ti centos ``

Ahora puedes ejectuar cualquier comando por el terminal y por ejemplo listar o instalar un editor de texto

`` ls ``

***
> [root@f1ae50f01b23 /]# ls
**anaconda-post.log  etc   lib64 mnt   root  srv  usr bin home  lost+found  opt   run   sys  var dev lib   media proc  sbin  tmp**
***

Podrás ejecutar cualquier comando dentro del terminal de linux como por ejemplo instalar un programa como nano

***
> [root@f1ae50f01b23 /]# yum install nano
Loaded plugins: fastestmirror, ovl
base                                            | 3.6 kB     00:00     
extras                                          | 3.4 kB     00:00     
updates                                         | 3.4 kB     00:00     
(1/4): extras/7/x86_64/primary_db                 | 191 kB   00:00     
(2/4): base/7/x86_64/group_gz                     | 155 kB   00:00     
(3/4): updates/7/x86_64/primary_db                | 7.8 MB   00:04 
***

Utiliza exit para salir del terminal

***
> [root@f1ae50f01b23 /]# exit
***

## Remover contenedores e imagenes

Para listar las imagenes activas 

`` docker ps -a ``

---
> CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
**f1ae50f01b23**        centos              "/bin/bash"         2 hours ago         Exited (1) 9 seconds ago                       tiny_elion
---

Y para eliminar las imagenes activas, esto porque cuando están activas consume disco duro, recuerda que puedes forzar la remoción con -f, pero siempre es aconsejable parar el servicio y luego eliminar

`` docker stop f1a `` si solo colocas los primeros caracteres docker distinguirá y parará el servicio, sino te preguntará que seas más específico  

`` docker rm f1ae50f01b23 ``
          
Para probar una imagen con una comando y luego borrarla

`` docker run -ti --rm centos ls ``

Podemos cambiar el nombre del contenedor, si ejecutamos el contenedor con el comando `` run -d httpd `` colocará el nombre por defecto pero si le damos un nombre nos ayudará a tener una mejor nomenclatura, tambien podrás eliminar el contenedor con el nombre que le has creado

`` docker run -d --name servidor_apache httpd ``

Lista ahora y verás el cambio 

`` docker ps -a ``
***
> CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS                 PORTS               NAMES
  685961bc644b        httpd               "httpd-foreground"   8 seconds ago       Up 7 seconds           80/tcp              **servidor_apache**
***

Para remover imagenes debemos escribir el nombre de la imagen junto con su version o escribir el ID

`` docker rmi postgres:latest ``

*** 
Ahora si listamos con `` docker images `` habrá eliminado la imagen del disco

> REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              latest              328edcd84f1b        2 weeks ago         192.5 MB
  httpd               latest              e74fcb59d25b        3 weeks ago         177.3 MB

***

Si tenemos una imagen que esta compartida que hace referencia a la misma

***
> REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              latest              328edcd84f1b        2 weeks ago         192.5 MB
  **httpd**               latest              e74fcb59d25b        3 weeks ago         177.3 MB
  **httpd**               2.4                 e74fcb59d25b        3 weeks ago         177.3 MB
***

Tendrás que borrar

`` docker rmi httpd:2.4 ``

***
> Untagged: httpd:2.4
***

Finalmente borra la ultima imagen y todas sus referencias

`` docker rmi httpd:latest ``
***
>Untagged: httpd:latest
Untagged: httpd@sha256:5b35d13089db73df620f4c198f5a4bfa56b8fe45a0364f343df9a26d874fef6c
Deleted: sha256:e74fcb59d25bdf03adcc8d89bcda8ae9456d2041c557c973072085708814e5c7
Deleted: sha256:74baa82fa7b49621f26d837d1946b239eab368714791e47dbc9fef63e808d563
Deleted: sha256:45f77e6fb6f40c7ae6d8ab47a2fb68aeb22366daa8203bfce4156a8121341b1b
Deleted: sha256:d88f40d9b23524a317ed26592089b884d6f3d1135761233b5b3fa6d175e3b4b2
Deleted: sha256:ae203f0b5fb6d009c5497b5f38411087d2fa50d0e7c3e7b5a707c8f83d761104
Deleted: sha256:68ba39668b7fee407c39a6b62559ffea45fbd53af01b913f59544a01fc341ca7
Deleted: sha256:0b5f5b5372687dc3ee654a390291ff3fad4a16453982324c7a1516b5fdad3344
***

## Dockerfile

Si vas a **dockerhub** (el repositório de imagenes de docker) puedes encontrar los archivos que crean las imagenes por ejemplo en el link de abajo encontrarás como es creado **nginx** (servidor http) desde un **Dockerfile**, así puedes personalizarlo a gusto según tus necesidades

> https://github.com/nginxinc/docker-nginx/blob/14aa3b1b80341099afbf90eb0a9b9061b7145f18/mainline/stretch/Dockerfile

Para instalar tu imagen desde el Dockerfile le das un build -t tag el nombre y la version. Si estás dentro de la carpeta del Dockerfile . docker reconocerá y ejecutará el archivo

`` docker build -t nginx:niko . ``

Para correr la imagen

`` docker run -d nginx:niko ``

Para crear nuestro proprio Dockerfile tenemos que saber cúal es su sintaxis, crea con tu editor favorito el archivo `` nano Dockerfile `` y escribe lo siguiente, ahí instalará la imagen ubuntu:latest

`` 
FROM ubuntu:latest
``
Para correr la imagen creada nuevamente le damos 

`` docker run -d ubuntu:latest ``

***
