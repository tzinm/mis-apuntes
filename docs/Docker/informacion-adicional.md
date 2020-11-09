### Docker Registry

Es un servicio donde se alojan las imágenes que utilizará Docker. A este servicio se realizan peticiones mediante los comandos `docker pull` (descargar imágenes) y `docker push` (subir imágenes).

El registry que utilizamos habitualmente es el propio de Docker, conocido como [Docker Hub](https://hub.docker.com/), del que nos descargamos las imágenes desarrolladas por la comunidad. Existe la posibilidad de mantener un registry local, podemos seguir la [documentación oficial](https://docs.docker.com/registry/.) para tenerlo en funcionamiento.

Los siguientes comandos muestran un ejemplo utilizando un registry local.

```bash
#Renombramos la imagen para que coincida con el nombre del registry, como lo haríamos en docker hub.
docker tag hello-world:lastest localhost:5000/hello-world

#Subimos la imagen.
docker push localhost:5000/hello-world

#Descargar la imagen.
docker pull localhost:5000/hellos-world
```

### Comandos más utilizados

| Comando                    | Descripción                                                  |
| -------------------------- | ------------------------------------------------------------ |
| docker images              | lista las imágenes que se encuentran descargadas             |
| docker build               | construir una imagen a partir de un fichero **Dockerfile**   |
| docker history             | listar las capas generadas en una imagen concreta            |
| docker run                 | crear un contenedor a partir de una imagen                   |
| docker rm                  | eliminar un contenedor                                       |
| docker rmi                 | eliminar una o más imágenes                                  |
| docker ps                  | listar los contenedores                                      |
| docker ps -a               | listar todos los contenedores                                |
| docker rename              | cambiar el nombre de un contenedor (renombrar)               |
| docker stop                | detener un contenedor (se puede utilizar el id o el nombre del contenedor) |
| docker start               | iniciar un contenedor (se puede utilizar el id o el nombre del contenedor) |
| docker restart             | reiniciar un contenedor (se puede utilizar el id o el nombre del contenedor) |
| docker logs                | para mostrar los logs de un contenedor. **Con el parámetro "-f" se actualizan los logs en tiempo real** |
| docker inspect             | muestra información detallada de como ha sido construido un contenedor |
| docker stats               | pasando el contenedor a este comando nos muestra cuantos recursos consume dicho contenedor |
| docker system df --verbose | muestra información detallada sobre el tamaño de todo el contenido de docker |
| docker volume ls           | lista los volúmenes de Docker. Únicamente lista los que se encuentran bajo el directorio root de Docker |
| docker exec                | permite ejecutar comandos dentro de un contenedor que esté activo |
| docker history imagen      | muestra como ha sido creado una imagen                       |
| docker cp                  | copiar archivos entre la máquina anfitrión y el contenedor, y viceversa |
| docker prune               | elimina todos los contenedores que se encuentran parados. Antes de eliminarlos se muestra un **aviso**. |

### Comandos adicionales

<ol><li><b>Docker build</b></li>


```bash
#Construir una imagen con un nombre de Dockerfile diferente al utilizado por defecto. Para ello se utiliza el parámetro "-f".

docker build -t test -f pruebadockerfile .
```

<li><b>Docker exec</b></li>

```bash
#Entrar en la terminal de un contenedor.

docker exec -ti nombre_del_contenedor bash

#exec: ejecutar
#-t: terminal
#-i: interactivo
#bash: la terminal seleccionada
#Se puede establecer el parámetro "-u" para seleccionar un usuario específico.
```

<li><b>Docker ps</b></li>

```bash
#Eliminar todos los contenedores a través de sus IDs.

docker ps -q | xargs docker rm -f
```

```bash
#Eliminar todos los contenedores con estados "exited".

docker ps -a | grep Exit | cut -d ' ' -f 1 | xargs sudo docker rm
```

<li><b>Docker cp</b></li>

```bash
#Copiar ficheros de mi máquina a un contenedor y viceversa.

docker cp mimaquina.txt test:/home

docker cp test:/home/mimaquina.txt .
```

<li><b>Docker run</b></li>

```bash
#Creación de un contenedor con el parámetro --rm para que se autoelimine una vez que haya salido de la sesión del contenedor.

docker run --rm -ti -name test ubuntu:latest bash
```

<li><b>Docker rmi</b></li>

```bash
#Eliminar imagenes huérfanas
docker rmi $(docker images -f "dangling=true" -q)

#Eliminar imagenes por fecha. Since (imagenes creadas posteriormente a la imagen pasada en el filtro), Before (imagenes creadas anteriormente a la imagen pasada en el filtro).
docker rmi $(docker images -f since="images" -q)
```

<li><b>Docker commit</b></li>

```bash
#Crear una imagen a partir de un contenedor
docker commit nombre-contenedor imagen-nueva
```

<li><b>Docker history</b></li>

```bash
#Visualizar el CMD del contenedor
docker history -h nombre-contenedor
```

</ol>