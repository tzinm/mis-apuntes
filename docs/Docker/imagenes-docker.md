### ¿Qué es una imagen?

Una imagen en Docker es una especie de instantánea de un contenedor. Para entenderlo de una forma más sencilla, desglosaremos una imagen en diferentes capas.

- **Primera capa (FROM):** se define el sistema operativo (Alpine, CentOS, Ubuntu), de aquí partirá el tamaño mínimo de nuestra imagen.
- **Segunda capa (RUN):** se ejecutan diferentes comandos, habitualmente para la instalación de paquetes necesarios para nuestra aplicación.
- **Tercera capa (CMD):** es la parte que mantendrá en ejecución el contenedor.

Estas capas se definen en el fichero **Dockerfile **y son de sólo lectura (RO - Read Only). Aquí podemos ver un ejemplo de un **Dockerfile**.

```dockerfile
FROM centos:7

RUN yum -y install httpd

CMD ["apachectl","-DFOREGROUND"]
```

El parámetro **CMD** es el que mantiene "**vivo**" el contenedor, en el ejemplo anterior es necesario utilizar como parámetro `-DFOREGROUND` del comando `apachectl` para que el contenedor se mantenga en ejecución. 

!!!note
    Únicamente podemos encontrar una instrucción **CMD** en un **Dockerfile**, en caso de que     haya más de una, sólo se tendrá en cuenta la última

Podemos definir la instrucción **CMD** de tres modos diferentes:

<ol><li><b>Execform</b>, es la forma más adecuada.</li>


```dockerfile
CMD ["ejecutable","parámetro1","parámetro2"]

#Ruta completa al ejecutable.
CMD ["/bin/echo","Hola mundo"]

#Bash como ejecutable, pasando el parámetro "-c" podemos ejecutar comandos como si estuviesemos en la terminal.
CMD ["/bin/bash","-c", "echo Hola mundo"]
```

<li>Como parámetros que acompañan a la instrucción <b>ENTRYPOINT</b>.</li>

```dockerfile
CMD ["parámetro1","parámetro2"]
```

<li>En formato <b>shell</b>, por debajo ejecuta <code>/bin/sh -c</code>.</li></ol>

```dockerfile
CMD comando parámetro1 parámetro2
```

La diferencia fundamental entre **CMD** y **ENTRYPOINT** se basa en que con el primero de ellos, cuando ejecutamos `docker run` con un comando específico, este comando sobrescribirá el comando definido en el fichero **Dockerfile**.

- **CMD** será el que utilizaremos habitualmente porque nos permite pasar parámetros cuando iniciemos el contenedor.

- **ENTRYPOINT** en caso de querer tener un contenedor como ejecutable, sin intención de pasar ningún parámetro adicional.

### Descargando imágenes

Para la descarga de imágenes utilizaremos el comando `docker pull`. Es posible pasar diferentes parámetros, uno de los más habituales es el **tag** o **etiqueta**. Esta etiqueta nos permite elegir entre diferentes versiones del mismo contenedor, una imagen compatible con la arquitectura de nuestro procesador o una versión, de la aplicación empaquetada en la imagen, particular que necesitemos.

**Ejemplo:**

```bash
#Descarga la versión 3.6.5 de mongo
docker pull mongo:3.6.5-jessie
```

En el ejemplo anterior hemos descargado una versión específica indicándole el **tag** después de los dos puntos.

#### ¿Qué es una etiqueta (tag)?

Una **etiqueta** o **tag** identifica la imagen que vamos a descargarnos. Es una forma sencilla de **versionar** las imágenes. Si no establecemos este parámetro, la etiqueta por defecto que utilizará Docker es **latest**. Las etiquetas disponibles por cada imagen se encuentran enumeradas en el propio repositorio, además en la documentación se encuentran instrucciones que facilitan la elección del tag de la imagen.

A continuación se encuentran dos enlaces que explican de forma extendida las etiquetas de las imágenes de Docker y que formas sencillas hay para versionar una imagen.

[The misunderstood Docker tag: latest](https://medium.com/@mccode/the-misunderstood-docker-tag-latest-af3babfd6375)

[Using Semver for Docker Image Tags](https://medium.com/@mccode/using-semantic-versioning-for-docker-image-tags-dfde8be06699)

### Desarrollando imágenes

La forma sencilla para comenzar ha realizar pruebas es crear un directorio de trabajo que utilizaremos para  desarrollar las primeras imágenes. En este directorio es necesario contar con un fichero **Dockerfile** (lo podemos crear ejecutando `touch Dockerfile`), el cual contendrá las capas que hemos visto más arriba. Este fichero es el utilizado por el comando `docker build` para crear las imágenes.

!!!note
    Podemos llamar al fichero Dockerfile de cualquier otro modo, pero cuando creemos el contenedor (`docker build`) deberemos indicarle con el parámetro `-f` el nombre del fichero.

#### Dockerfile

¿Qué instrucciones nos podemos encontrar dentro de este fichero?

- **COPY**: permite copiar un fichero o directorio del host a la imagen.
- **ADD**: además de lo que nos permite realizar la instrucción <u>COPY</u>, esta nos **permite pasar una URL** para que descargue el contenido en la ruta que indiquemos dentro de la imagen.
- **ENV**: se utiliza para añadir variables de entorno. La variable se establece **sin el signo =**, la variable y el valor se define con un espacio entre ambos. Ejemplo:

```dockerfile
ENV nombre juan
```

- **WORKDIR**: define el directorio de trabajo dentro del contenedor. Si no definimos este parámetro, por defecto nos encontramos en el directorio raíz **/**.
- **EXPOSE**: permite exponer puertos. Realmente esta instrucción no realizar ninguna exposición, simplemente informa a la persona que va a utilizar la imagen para crear el contenedor que puertos debe configurar. Estos puertos se expondrán utilizando la opción **-p** del comando `docker run`.
- **LABEL**: hace referencia a una etiqueta, que se utiliza habitualmente para añadir metadatos (información sobre la imagen como versión, creador, etc.).
- **USER**: identifica al usuario que ejecuta las instrucciones definidas en el fichero Dockerfile (las instrucciones que se establezcan a partir de la instrucción **user**). Por defecto el usuario que ejecuta las instrucciones es el usuario **root**.
- **VOLUME**: permite que los datos que se encuentran en estos "volúmenes" sean persistentes, ya que después de eliminar el contenedor asociado a este volumen los datos no se perderán. 
- **CMD**: se entiende como la instrucción que mantiene vivo el contenedor.

A continuación veremos un **Dockerfile** con todas las instrucciones vistas hasta el momento:

```dockerfile
FROM nginx

LABEL version=1.0

RUN useradd prueba

USER prueba

WORKDIR /usr/share/nginx/html

COPY web /usr/share/nginx/html

ENV variable1 tzinm

EXPOSE 90

VOLUME  /var/log/nginx

CMD nginx -g 'daemon off;'
```

Las instrucciones que definimos dentro de este fichero deben ser lo más estrictas posibles para evitar errores, por ello es importante tener en cuenta lo siguiente:

- Un servicio por contenedor.
- Utilizar la capa **Labels** para añadir metadatos.
- Agrupar los argumentos, evitando crear una capa por cada argumento.

=== "Forma correcta"
    ```dockerfile
    RUN \
      python3 -m pip install telegram --upgrade && \
      python3 -m pip install python-telegram-bot --upgrade && \
      chown root:root AddToQbitTorrentFolder.py && \
      chmod 644 AddToQbitTorrentFolder.py
    ```

=== "Forma incorrecta"
    ```dockerfile
    RUN python3 -m pip install telegram --upgrade
    RUN python3 -m pip install python-telegram-bot --upgrade
    RUN chown root:root AddToQbitTorrentFolder.py
    RUN chmod 644 AddToQbitTorrentFolder.py
    ```

- Evitar la instalación de paquetes innecesarios, de ese modo crearemos imágenes más ligeras.

Además del fichero **Dockerfile** hay otro fichero importante llamado **.dockerignore**. El funcionamiento de este es similar al fichero **.gitignore**, es un fichero oculto en el que definimos que contenido del directorio no queremos que sea tenido en cuenta.

#### Docker Build

Una vez que tenemos nuestro **Dockerfile** terminado debemos ejecutar el comando `docker build` para crear nuestra imagen.

```bash
docker build -t tzinm/test .
```

En el comando ejecutado se ha denominado a la imagen **tzinm/test**, al no haber especificado ningún tag será el tag **latest** el que se haya establecido. Otro dato importante es el **punto** del final, que hace referencia al [contexto](https://github.com/docker/docker.github.io/issues/5606#issuecomment-354004184), es decir el directorio en el cual Docker buscará los ficheros necesarios para crear la imagen.

#### Multi-Stage Builds

Esta nueva funcionalidad que nos ofrece Docker se encuentra disponible a partir de la versión 17.05. Es una utilidad interesante ya que nos permite crear

En ocasiones, algunas imágenes necesitan cierto contenido que es generado mediante la compilación o ejecución de una serie de instrucciones que hace que el peso de una imagen crezca y su rendimiento no esté lo más optimizado posible. 

Es aquí cuando esta herramienta (**multi-stage builds**) es útil, ya que nos permite en una primera instancia crear una imagen "al vuelo" en la que generaremos esos ficheros mediante las instrucciones que necesitemos. Después esos ficheros serán enviados (veremos luego en un ejemplo como se realiza esto) a la segunda imagen, exclusivamente esos ficheros, desprendiéndonos de la primera imagen y todo lo que ello conlleva.

Recordamos que para hacer un uso adecuado de Docker es necesario optimizar al máximo las imágenes, es la base de las buenas prácticas.

A la hora de crear el fichero **Dockerfile** es muy similar al que utilizamos construyendo una única imagen, salvo que en esta ocasión nos encontramos con la construcción de dos imágenes - por lo tanto dos **FROM**- y en la segunda imagen añadimos una capa - **COPY**- con el flag **--from=primar-imagen** que se encargará de pasar el contenido generado en la primera imagen (**primer FROM**). Se puede ver un poco más claro en el siguiente ejemplo.

```dockerfile
FROM centos as test

RUN fallocate -l 10M /tmp/file1 && \
    fallocate -l 40M /tmp/file2

FROM alpine

COPY --from=test /tmp/file1 /tmp/test1
```

#### Dangling images

Este concepto hace referencia a imágenes huérfanas, que no son más que aquellas imágenes que se encuentran **sin referenciar**.

Esto sucede cuando una imagen es construida utilizando el mismo **nombre** y **tag** que otra creada anteriormente. La nueva imagen creada será válida, en cambio las imágenes anteriores que existiesen no serán válidas y pasarán a tener como nombre y tag el valor **\<none\>**.

A pesar de que estas imágenes no son válidas, seguirán estando en nuestro sistema y ocupando espacio, por ello es recomendable eliminarlas. El modo más rápido de eliminar todas estas imágenes es mediante el siguiente comando:

```bash
docker rmi $(docker images -f "dangling=true" -q)
```

En el comando anterior hemos utilizado el flag **-f**, que permite el filtrado de las imágenes siguiendo una serie de [patrones](https://github.com/moby/moby/blob/10c0af083544460a2ddc2218f37dc24a077f7d90/docs/reference/commandline/images.md#filtering).