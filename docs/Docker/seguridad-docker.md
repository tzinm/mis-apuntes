Este apartado lo he denominado "seguridad" porque creo que es donde mejor encajala gestión de los usuarios en Docker. Para profundizar en el tema de seguridad es conveniente revisar la [documentación oficial](https://docs.docker.com/engine/security/security/).

Simplificando mucho, sabemos que los contenedores de Docker son procesos para el sistema anfitrión, por lo tanto el kernel del sistema es compartido entre todos los contenedores y el propio sistema. Es importante tener en cuenta esto ya que el kernel de Linux es el encargado de gestionar el **uid** y el **gid** de los diferentes usuarios, por lo tanto, estos serán compartidos con los contenedores de Docker.

Cuando ejecutamos un contenedor, si no se especifica un usuario en la creación del propio contenedor o en la imagen base del mismo, el **uid** y **gid** por defecto será **0**. Este **id** corresponde al usuario **root**, el usuario administrador de nuestro sistema.

Una buena práctica para controlar este comportamiento es contar con un usuario específico para Docker, al que se le otorgarán los permisos exclusivamente necesarios para que el contenedor que hayamos creado pueda únicamente realizar la tarea para la que se ha creado.

Después de que hayamos creado el usuario que utilizaremos en los contenedores Docker, podemos obtener los ids que necestiamos (uid y gid) ejecutando el siguiente comando:

```bash
id usuario-docker
```

Para indicar al contenedor que usuario será el que ejecute el **CMD**, se lo indicaremos con el flag `-u`.

```bash
docker run -u="uid:gid"
```

Para verlo de una forma más clara, lo trataremos con un <u>ejemplo</u>.

Imaginemos que queremos montar una biblioteca multimedia al estilo "Netflix" a partir de las películas y series digitales que tenemos en nuestros equipos. Hay diversas herramientas que nos facilitan esto, como por ejemplo **Plex**, que es una de las más conocidas. 

Esta biblioteca podemos ponerla en marcha a través de un contenedor Docker, pero para que funcione correctamente este contenedor necesita acceso a las carpetas donde se encuentra todo nuestro contenido multimedia. Bien, si el usuario del contenedor no es el adecuado el contenedor no será capaz de leer los directorios donde se encuentra todo el contenido multimedia, y por lo tanto no podrá realizar su trabajo y no tendremos nuestra ansiada biblioteca multimedia.

Siguiendo con la premisa de un usuario exclusivo para Docker, este usuario debería tener acceso a los directorios donde se encuentra todo el contenido multimedia y a su vez, ser el usuario que hemos pasado por parámetros al contenedor de Plex.