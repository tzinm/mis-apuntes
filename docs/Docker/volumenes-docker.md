Como ya sabemos, si eliminamos un contenedor todos sus datos se eliminarán con él. Para evitar esto tenemos los volúmenes que nos permiten hacer persistentes algunos datos de nuestros contenedores. Estos datos se almacenan en un directorio de la máquina anfitriona.

Docker cuenta con **tres tipos** de volúmenes.

### Volúmenes Host

En este tipo de volúmenes definimos el directorio donde queremos que se almacene la información que debe ser persistente. Es el tipo de volúmenes que más utilizaremos:

```bash
docker run -v /midirectorio:/directorio-docker
```

### Volúmenes Anonymous

Seguimos haciendo persistentes los datos, pero en este caso no definimos donde queremos que se almacene la información, sino que Docker genera un directorio aleatorio. 

```bash
docker run -v /directorio-docker
```

Estos directorios se encuentran en una ruta específica bajo el directorio "**raíz**" del servicio Docker. Para conocer dicho directorio es necesario ejecutar la siguiente instrucción:

```bash
docker info | grep -i root
```

Entre los diferentes directorio bajo el directorio raíz de Docker, tenemos un directorio denominado volúmenes, que es el directorio donde se almacenaran estos directorios llamados **anonymous**.

Los directorios **anonymous** también aparecen cuando en un fichero Dockerfile definimos la capa **VOLUME** pero a la hora de ejecutar el contenedor no definimos ningún volumen.

Esta forma de crear volúmenes no es aconsejable por varios motivos.

1. El nombre que otorga al directorio es aleatorio (habitualmente conformado por números), por lo tanto no será sencillo saber que volumen se encuentra asociado a cada contenedor.
2. A la hora de eliminar el contenedor si pasamos la opción **-v** eliminaremos también el volumen asociado a este contenedor.

### Volúmenes Nombrados

Es una mezcla de los dos anteriores. El volumen se almacena en el mismo lugar que los volúmenes anonymous, pero en este caso somos nosotros quienes elegimos un nombre para estos volúmenes en vez de ser un nombre aleatorio.

Los siguientes comandos se utilizan para la gestión de volúmenes:

<ol><li>Creando un volumen</li>


```bash
docker volume create nombre_del_volumen
```

<li>Listar los volúmenes</li>

```bash
docker volume ls
```

<li>Eliminar un volumen</li>

```bash
docker volume rm nombre_del_volumen
```

<li>Utilizar un volumen nombrado</li>

```bash
docker run -v nombre_del_volumen:/dockercontainer
```

</ol>A diferencia de los volúmenes host, donde debemos indicar la ruta completa del directorio, solo es necesario indicar el nombre del volumen que hayamos creado.

### Dangling Volumes

Este concepto que hemos visto con los contenedores también existe con los volúmenes. Estos aparecen cuando hemos eliminado un contenedor y el volumen asociado a este (volumen anonymous) no se ha eliminado.

Para eliminar estos volúmenes podemos hacerlo con la siguiente línea de terminal:

```bash
docker volume ls -f "dangling=true" -q | xargs docker volume rm
```

- **-f / --filtering**: flag utilizado para filtrar los volúmenes.
- **dangling**: es un filtro booleano que únicamente permite **true** o **false**.
- **-q **(quiet): imprime el ID que identifica a cada volumen.

Con [xargs](https://www.patapalo.net/blog/view/7/ejemplos-de-xargs-en-sistemas-linux-unix) pasamos el resultado a un segundo comando, de ese modo conseguimos eliminar con una única línea todos los volúmenes.

!!!note
    Los volúmenes se pueden compartir entre varios contenedores, simplemente es indicar el volumen en el parámetro correspondiente cuando creemos los contenedores. Puede ser útil cuando necesitamos que dos contenedores accedan a la información que se encuentra en un directorio de nuestra máquina.