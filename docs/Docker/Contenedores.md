En este apartado veremos diferentes imágenes con las que podremos poner en marcha diferentes contenedores. 

Docker nos ofrece un repositorio donde podemos encontrar una gran cantidad de imagenes, [Docker Hub](https://hub.docker.com). Dentro de Docker Hub podemos encontrarnos con una comunidad ([Linuxserver](https://hub.docker.com/u/linuxserver))que se dedica a desarrollar diferentes imágenes de una gran variedad de servicios. El código de sus imagenes podemos revisarlo en [GitHub](https://github.com/linuxserver).

Desde hace algún tiempo, la comunidad de Linuxserver ha implementado una forma sencilla de realizar modificaciones a sus imagenes, sin que estas modificaciones alteren la imagen base. Esta funcionalidad la han denominado [**Docker Mod**](https://blog.linuxserver.io/2019/09/14/customizing-our-containers/).

En GitHub han dejado una pequeña [guía](https://github.com/linuxserver/docker-mods) de como realizar dichos mods y además varios ejemplos, desde una modificación sencilla hasta modificaciones más complejas.

He aprovechado esta nueva funcionalidad y he desarrollado un mod para transmission. 

[https://github.com/tzinm/remove-finished](https://github.com/tzinm/remove-finished)

[https://hub.docker.com/r/tzinm/remove-finished](https://hub.docker.com/r/tzinm/remove-finished)

### Imágenes interesantes

#### [Watchtower](https://hub.docker.com/r/v2tec/watchtower)

El objetivo de este contenedor es mantener los contenedores actualizados (incluido él mismo). Automáticamente ejecutará `docker pull` para la descarga de la imagen más actual, parará el contenedor en ejecución y lo iniciará de nuevo con las mismas opciones con las que se había creado inicialmente.

Algunas de las opciones que podemos utilizar:

* **Notificaciones:** este contenedor permite el envío de notificaciones a través de **slack** y vía **email**. He seleccionado el envío de notificaciones a través de email, para ello es necesario añadir una serie de variables que veremos en el ejemplo más adelante. Respecto a las notificaciones, el nivel de notificaciones por defecto es **info**, en caso de querer modificarlo es necesario añadir la variable `WATCHTOWER_NOTIFICATIONS_LEVEL`.
* **Contenedores:** si no indicamos lo contrario se comprobará y actualizarán todos los contenedores. Puesto que mi interés es actualizar únicamente una serie de contenedores, es necesario indicarle el nombre estos (importante escribirlo correctamente, teniendo en cuenta mayúsculas y minúsculas) como argumentos.
* **Programación:** para programar las actualizaciones tenemos dos opciones `--interval` y `--schedule`.
  * **Interval:** se establece un valor en segundos (por defecto son 300 segundos).
  * **Schedule:** es similar a las expresiones de [cron](../Linux/Cron.md) pero con un **6** campos. Por ejemplo `--schedule "0 0 5 * * *"` ejecutaría las comprobaciones todos los días a las 5 de la mañana.
* **Debug mode:** estableciendo esta opción el log recoge más información.
* **Cleanup**: elimina las imagenes antiguas.

##### Ejemplo

````dockerfile
docker run -d \
  --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e WATCHTOWER_NOTIFICATIONS=email \
  -e WATCHTOWER_NOTIFICATION_EMAIL_FROM=fromaddress@gmail.com \
  -e WATCHTOWER_NOTIFICATION_EMAIL_TO=toaddress@gmail.com \
  -e WATCHTOWER_NOTIFICATION_EMAIL_SERVER=smtp.gmail.com \
  -e WATCHTOWER_NOTIFICATION_EMAIL_SERVER_PORT=587 \
  -e WATCHTOWER_NOTIFICATION_EMAIL_SERVER_USER=fromaddress@gmail.com \
  -e WATCHTOWER_NOTIFICATION_EMAIL_SERVER_PASSWORD=app_password \
  v2tec/watchtower tautulli plex transmission heimdall syncthing watchtower --schedule "0 0 5 * * *" --cleanup --debug
````

!!!note
	Muchos de los usuarios que utilizamos Docker, utilizamos los contenedores desarrollado por la comunidad LinuxServer. Estos no recomiendan el uso de watchtower par actualizar los contenedores (al menos los desarrollado por ellos), sino que recomiendan varios scripts que podemos encontrar [aquí](https://blog.linuxserver.io/2019/10/01/updating-and-backing-up-docker-containers-with-version-control/) para actualizar y realizar backups del estado de los contenedores.

