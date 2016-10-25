# Docker

![https://upload.wikimedia.org/wikipedia/commons/7/79/Docker_(container_engine)_logo.png](https://upload.wikimedia.org/wikipedia/commons/7/79/Docker_(container_engine)_logo.png)

Docker sirve para crear **contenedores** y poner dentro de ellos un software (o varios) junto con todo lo que necesitan para correr: su sistema de archivos, código, herramientas del sistema, librerías, etc, cualquier cosa que normalmente podamos instalarle a un sistema operativo.

Terminología básica:

* Un **contenedor** es una versión de Linux reducida a sus componentes más básicos. 

* Una **imagen** es el software que cargamos en un contenedor. 

* Un **dockerfile** es un script que describe (e instala) el software que pondremos en una imagen, pero esto no incluye sólo el programa en sí, sino también cualquier detalle de la configuración del ambiente y hasta los comandos que queremos corra.


![docker_layered.png](docker_layered.png)


Esto nos permite que un programa corra de manera idéntica sin importar el sistema operativo original del equipo, y hace que la instalación sea independiente de la instalación de otro software. Esto es importante porque al instalar un programa bioinformático es común "romper" las dependencias de otro programa.

Funciona también con Windows (7 onwards, 64 bit). Por lo que esto es una alternativa a hacer una partición de disco o a correr Ubuntu/Biolinux desde VirtualBox (docker es mucho más ligero, aunque hagan cosas parecidas).

![docker_VMvsDocker.PNG](docker_VMvsDocker.PNG)

En el siguiente link puedes encontrar las instrucciones para instalar Docker en tu computadora (cambiar tutorial según OS). Traerlo instalado para la próxima clase.

[Tutorial instalación y primeros pasos de Docker](https://docs.docker.com/mac/).

Y aquí unos videos de referencias extra, por si les quieres dar un ojo: [Tutoriales en video](https://training.docker.com/self-paced-training).

## Funcionamiento básico de docker:

* `pull` una imagen (solo la primera vez)
* `run` la imagen dentro de un contenedor (para crearlo, solo la primera vez)
*  `exit` para salir del contendor
*  `stop` para detener un contenedor 
*  `restart` para reactivar un contenedor
*  `exec` para entrar a un contenedor activo

1) Primero prendemos la máquina (para poder correr los comandos de docker, esto es similar a prender una máquina virtual en VirtualBox). 

* En Windows o Mac:
Click en QuickStartTerminal

Nota: al correr desde QuickStartTerminal los comandos son `docker OPTIONS`, mientras que si lo hacemos desde la terminal normal (en caso de que tengas Mac) es necesario escribirlo así: `docker $(docker-machine config default) OPTIONS`. Se ve más engorroso, pero permite quitar una capa extra (el QuickSratTerminal) y que montar volúmenes sea más sencillo. Esto lo veremos adelante, por lo pronto, volvamos al ejemplo. 

2) Bajamos la última versión de ubuntu con `pull`:

```
$ docker pull ubuntu #Baja la última versión de ubuntu 
Using default tag: latest
latest: Pulling from library/ubuntu

5a132a7e7af1: Pull complete 
fd2731e4c50c: Pull complete 
28a2f68d1120: Pull complete 
a3ed95caeb02: Pull complete 
Digest: sha256:4e85ebe01d056b43955250bbac22bdb8734271122e3c78d21e55ee235fc6802d
Status: Downloaded newer image for ubuntu:latest
```
 
Aquí por default bajó la última, pero también hubieramos podido especificar qué versión de ubuntu queríamos, así:  `docker pull ubuntu:14.04`
 
Para revisar hayamos bajado la imagen deseada:

```
$ docker images #Enlista imagenes ya bajadas
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              07c86167cdc4        11 days ago         188 MB
hello-world         latest              690ed74de00f        5 months ago        960 B
docker/whalesay     latest              6b362a9f73eb        9 months ago        247 MB
```    

3) Cargamos la imagen dentro de un contenedor con `run`. Voilá, estamos dentro de un Ubuntu, específicamente dentro de un **contenedor** corriendo Ubuntu.
      
```
$ docker run -it ubuntu bash
root@740df4e6d81e:/# 
root@740df4e6d81e:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
```

**Pregunta**: ¿Qué significa el `#` en vez del `$`?

Cuando termines puedes salir (`exit`) de este contenedor. Los cambios que hayas hecho **no se guardarán en la imagen**, pero **sí en el contenedor que se creó al correrla**. 

Vamos a ver qué contenedores tenemos:

```
$ docker ps  
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
a5864268eadd        ubuntu              "bash"              46 hours ago        Up 7 minutes                            sleepy_pasteur
```

Ese es nuestro contenedor. Se encuentra corriendo (aunque no haga nada). Para volver a entrar a él utilizamos `exec`:

```
$ docker exec -it a5864268eadd bash
root@a5864268eadd:/#
root@a5864268eadd:/# mkdir Prueba # hacer un directorio prueba
root@a5864268eadd:/# ls
Prueba  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
(nota que el texto alfanumérico después de `exec` es el ID del container. 

Si nos salimos (`exit`) y luego queremos detenerlo por completo:

```
$ docker stop a5864268eadd 
```

Si enlistamos con `docker ps` los contenedores corriendo ya no tendremos ningún resultado. Sin embargo, aún podemos ver ver otros contenedores no activos:

```
$ docker ps -a 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
a5864268eadd        ubuntu              "bash"              46 hours ago        Exited (0) 32 hours ago                       sleepy_pasteur
28500c7d3069        ubuntu              "bash"              46 hours ago        Exited (0) 46 hours ago                       elegant_yalow
ee966523a24f        hello-world         "/hello"            46 hours ago        Exited (0) 46 hours ago                       tiny_feynman
f09c940dfdc9        docker/whalesay     "cowsay boo"        46 hours ago        Exited (0) 46 hours ago                       big_einstein
d44c3d46c6f9        hello-world         "/hello"            46 hours ago        Exited (0) 46 hours ago                       mad_euclid
e5af547543fa        ubuntu              "/bin/bash"         46 hours ago        Exited (0) 46 hours ago                       determined_mccarthy
a638c4048191        ubuntu              "/bin/bash"         46 hours ago        Exited (0) 46 hours ago                       big_ritchie
5b4ad6c46797        hello-world         "/hello"            46 hours ago        Exited (0) 46 hours ago                       adoring_babbage
```

Si queremos volver a ejecutar un proceso en nuestro contenedor (con `exec` como hicimos arriba) primero necesitamos reiniciarlo (o sea deshacer el stop):

`docker restart a5864268eadd`

Y ya luego podemos volver a entrar a el:

```
docker exec -it a5864268eadd bash
root@a5864268eadd:/# ls
Prueba  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

Nota que los cambios que hayas realizado dentro del contenedor continúan existiendo.

Si quieres borrar contenedores o imágenes (son espacio en disco):

* Borrar un contenedor: Primero deterlo con `docker stop CONTAINER_ID` y luego borrarlo con `docker rm CONTAINER_ID`

* Borrar una imagen: `docker rmi -f IMAGE_ID`


## BioContainers

En los ejemplos anteriores utilizamos un contenedor de Ubuntu en su expresión más sencilla (sin casi nada instalado). Docker permite además bajar imágenes que contengan no sólo Ubuntu a secas, sino otro sofware instalado sobre eso.

[BioContainers](https://github.com/BioContainers/specs) es un proyecto que está realizando contenedores de softawere bioinformático. 

[Checa la lista de contendores disponinbles](https://github.com/BioContainers/containers) 

El contendor básico BioContainers tiene varias herramientas de rutina que utilizaremos (por ejemplo `curl`), por lo que recomiendo que trabajemos en este contenedor para realizar nuestras prácticas.


```
$ docker pull biodckr/biodocker
Using default tag: latest
latest: Pulling from biodckr/biodocker
8387d9ff0016: Already exists 
3b52deaaf0ed: Already exists 
4bd501fad6de: Already exists 
a3ed95caeb02: Pull complete 
1271a85e53b2: Pull complete 
038e0519162f: Pull complete 
b7326f133df8: Pull complete 
174099b62d65: Pull complete 
aed7fb466079: Pull complete 
Digest: sha256:5856e57be18548b8f4244bad94b548a777c1b7261dba896f7ecd09d9a58aefeb
Status: Downloaded newer image for biodckr/biodocker:latest
$
$ docker images 
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
miproyecto/analisis1   v1                  b951cd1b24b5        11 days ago         188 MB
ubuntu                 latest              07c86167cdc4        3 weeks ago         188 MB
biodckr/biodocker      latest              5c0a896aa5b1        7 weeks ago         702.5 MB
hello-world            latest              690ed74de00f        5 months ago        960 B
$
$ docker run -it biodckr/biodocker /bin/bash
biodocker@4e415e3c6633:/data$ curl
curl: try 'curl --help' or 'curl --manual' for more information 
```

**Observaciones y preguntas**:

* En vez de ser root (´#´ al inicio de la línea de comando) como es el default de docker, somos un usuario normal y estamos en un directorio llamado ´data´. ¿Con qué líneas del dockerfile se realizó esto?

* Pude hacer un docker pull porque el dockerfile de arriba existe en un repositorio de contenedores llamado `biodckr`. ¿Cuál es la diferencia entre `pull` y `build` an image?

El contenedor creado a partir de `biodckr/biodocker` ya tiene varias cosas instaladas.

## Montar volúmenes para poder acceder a carpetas de la computadora host

Para que un contendor pueda "ver" contenido en nuestra computadora necestiamos correr la imagen deseada dentro de un contenedor, pero **montando un volumen**, es decir un directorio en tu equipo que podrá ser accedido por el contenedor:

```
docker run -v /Users/ticatla/hubiC/Science/Teaching/Mx/BioinformaticaCUCBA-2016:/data -it biodckr/biodocker bash
```

Desglozando el comando anterior:

`-v` es la bandera para indicar que queremos que monte un volumen 
`/Users/ticatla/hubiC/Science/Teaching/Mx/BioinformaticaCUCBA-2016` es la ruta absoluta. Sí, absoluta (así que cambiala por la ruta de tu equipo) ya que así es cuando se trata de montar volúmenes :(. Ojo, para windows debes iniciar la ruta con `/c/Users` y después el resto de la ruta.

`:/DatosContenedor` es el nombre del directorio como quremos que aparezca dentro de nuestro contenedor. 

Explora el volumen que montaste, prueba hacer un archivo. Nota que puedes acceder a el desde tu explorador, es decir todo lo que suceda en ese directorio puedes verlo/modificarlo desde dentro y fuera del contenedor. 

```
biodocker@fd13b1070dc0:/data$ ls
AsistentesCurso_MastrettaOct2016.xlsx  README.md  Tema1  Tema3  Tema5
Practicas                              Tareas     Tema2  Tema4  Tema6
```
