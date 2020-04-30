# Rclone in OMV

En esta pequeña guía voy a mostrar la configuración de Rclone con Google Drive. Se puede utilizar con muchos más servicios. Consultar [aquí](https://rclone.org/)

![Servicio Rclone](https://dl.dropboxusercontent.com/s/fpp146ye8ijyt2e/Rclone_servicios.png?dl=0)

La instalación es muy sencilla, en [esta web](https://rclone.org/install/) podemos ver la instalación tanto para Linux como para MacOS. Podemos optar por llamar a un script que realizará todo el proceso de instalación, o descargarnos un fichero *.zip* que descomprimiremos e instalaremos.

## Instalación:

Me he percatado que siguiendo la guía de instalación sobre Linux, no descarga la última versión. Para evitar esto, y descargar la última versión, iremos a este [enlace](https://rclone.org/downloads/) y copiaremos la dirección del fichero *.zip* que corresponde con nuestra máquina. 

![Rclone última versión](https://dl.dropboxusercontent.com/s/6vh7e5f6s9szhbp/Rclone_ultima_version.png?dl=0)

No sé si ya lo he comentado, pero la instalación que estoy realizando es sobre OpenMediaVault que es un sistema orientado a NAS. Tal y como dicen sus desarrolladores está orientado a utilizar la interfaz web y no la "cli" (Command line interface). 

Es por ello que iré comprobando que los binarios correspondientes a los comandos que debo ejecutar se encuentran instalados en el sistema. Para ello ejecutaré `apt list --installed | grep nombre_del_binario`. 

 A continuación pegaré de la web oficial los comandos necesarios para su instalación:

* Descarga del fichero *.zip* y descompresión.
````
curl -O https://downloads.rclone.org/v1.47.0/rclone-v1.47.0-linux-amd64.zip
unzip rclone-v1.47.0-linux-amd64.zip
cd rclone-v1.47.0-linux-amd64
````
*Como comentaba, la ruta que debemos poner es la que corresponde a la última versión de Rclone*

* Trasladar el binario *rclone*.

````
sudo cp rclone /usr/bin/
sudo chown root:root /usr/bin/rclone
sudo chmod 755 /usr/bin/rclone
````
*La ruta /usr/bin se encuentra en el PATH, que es la variable que contiene las rutas a los directorios que contienen los ficheros ejecutables. De ese modo podremos ejecutar Rclone sin importar en que directorio del sistema nos encontremos.*

* Añadiendo el manual de rclone a la base de datos de manuales.

````
sudo mkdir -p /usr/local/share/man/man1
sudo cp rclone.1 /usr/local/share/man/man1/
sudo mandb 
````
*Copiamos el manual y actualizamos la base de datos de los manuales para que podamos invocar el "man page" ejecutando `man rclone`*

## Configuración:

La configuración es muy sencilla ya que el propia sistema nos guía una vez que ejecutamos `rclone config`. Cuando ejecutamos este comando los aparecerá un pequeño menú desde donde crearemos los nuevos servicios remotos (por denominarlos de algún modo).

![Rclone config](https://dl.dropboxusercontent.com/s/6gxoqjoll0ekiz5/Rclone_config.png?dl=0)

La opción para crear es la "*n*", que una vez seleccionada esta opción nos aparecerá en el prompt para asignarle un nombre, y a continuación debemos elegir que servicio queremos configurar.

![Rclone configuración nube](https://dl.dropboxusercontent.com/s/110drdtikmgfmv2/Rclone_servicio_nube.png?dl=0)

En mi caso he seleccionado Google Drive que es la opción número 12 (quizá pueda variar de el número si la versión de Rclone es diferente). Nos aparecerán una serie de parámetros para ir configurando, únicamente mostraré los que he necesitado configurar, puesto que el resto los he dejado por defecto ya que no necesitaba configurarlos.

En uno de los pasos de la configuración debemos establecer los permisos que le otorgamos, he seleccionado la opción número "1" que es la más permisiva, ya que tenemos acceso total a todos los contenidos.

![Rclone permisos](https://dl.dropboxusercontent.com/s/yt8w5zqj8quut0x/Rclone_permisos.png?dl=0)

Otra de las preguntas que nos aparecerán a continuación es si deseamos editar la configuración avanzada, para nuestro supuesto no es necesario, por lo que seleccionaremos "**n**".

![Rclone configuración avanzada](https://dl.dropboxusercontent.com/s/28fayigbg9xro0q/Rclone_advanced_config.png?dl=0)

La siguiente cuestión a tener en cuenta es si deseamos utilizar la configuración automática. Como podemos ver en la explicación, en el segundo punto nos indica que sí estamos trabajando en una máquina remota (por ejemplo contra nuestro servidor vía ssh) debemos seleccionar la opción "**n**", que será la opción que seleccionaremos.

![Rclone remote server](https://dl.dropboxusercontent.com/s/fw1nfkx3uh1iye9/Rclone_servidor_remoto.png?dl=0)

Al haber pulsado la opción "**n**" nos mostrará una dirección que debemos copiar y pegar en nuestro navegador. Esta dirección nos solicitará autenticarnos con la cuenta de Google Drive que queremos vincular con Rclone.  Una vez que nos autentiquemos nos aparecerá un código que copiaremos y pegaremos en la línea que nos solicita Rclone.

![Rclone api google](https://dl.dropboxusercontent.com/s/wq5i7a9hbtkq8r5/Rclone_api.png?dl=0)

Antes de finalizar, nos preguntará si deseamos configurar como un Team Drive [^1], no es nuestro caso por lo tanto seleccionaremos "**n**".

![Rclone Team Drive](https://dl.dropboxusercontent.com/s/m70pq7mb4hsl9iw/Rclone_teamdrive.png?dl=0)

Nos mostrará un pequeño resumen con el "*remote*" que hemos configurado. Para finalizar debemos seleccionar con la letra "**q**" (Quit config).

![Rclone finalizamos configuración](https://dl.dropboxusercontent.com/s/xqbu414lkpwm5cn/Rclone_configuracion_final.png?dl=0)

## Opciones utilizadas:

En la creación de los **Remotes** he creado dos idénticos, es decir dos de cuentas Google Drive, uno para copiar el contenido, y el otro para sincronizarlo.

La diferencia es que con la opción copy quiero evitar que si borro contenido no se elimine de mi servidor, en cambio con la opción sync quiero conseguir que tanto en GD como en mi servidor el contenido sea idéntico.

Los comandos a ejecutar se introducirán en diferentes scripts, que a su vez se incorporarán a otro script para que compruebe si el script de *rclone sync* y *rclone copy* se están ejecutando, para evitar una nueva ejecución.

* **Rclone sync (script):**

````bash
#!/bin/bash
rclone sync ASIR: /sharedfolders/ASIRDrive -v --log-file=/home/Logs/rclonesync.txt -u --bwlimit 8650k --tpslimit 10 --transfers 15 --exclude Maquinas_Virtuales/ --exclude ISOS/
exit
````

````bash
#!/bin/bash
#Comprobamos que el script no se está ejecutando mediante pidof
if pidof -o %PPID -x sh /home/Scripts/rclonesync.sh; then
	exit 1
fi
#Después de comprobar, si no se está ejecutando pasará a ejecutar los siguientes comandos.
echo "Ejecutando rclonesync..."
#Puesto que la ruta de los scripts no se encuentra en la variable PATH es neceario indicarle la ruta completa
/home/Scripts/rclonesync.sh
echo "¡La ejecución del script ha finalizado!"
exit
````

* **Rclone copy (script):**

````bash
#!/bin/bash

rclone copy TzinmDrive:General /sharedfolders/CursosDrive -v --log-file=/home/Logs/rclonecopy.txt -u --bwlimit 8650k --tpslimit 10 -transfers 15
exit
````

````bash
#!/bin/bash
#Comprobamos que el script no se está ejecutando mediante pidof
if pidof -o %PPID -x /home/Scripts/rclonecopy.sh; then
	exit 1
fi
#Después de comprobar, si no se está ejecutando pasará a ejecutar los siguientes comandos.
echo "Ejecutando rclonecopy..."
#Puesto que la ruta de los scripts no se encuentra en la variable PATH es neceario indicarle la ruta completa
/home/Scripts/rclonecopy.sh
echo "¡La ejecución del script ha finalizado!"
exit
````

*La ruta que he establecido después de sh puede diferir con la que utilicéis. Tenéis que indicar la ruta donde se encuentra vuestro script.*

Una vez que tenemos los script creados sólo quedará añadirlos al cron, que podemos aprovecharnos de la interfaz web de OpenMediaVault o añadirlos mediante comandos, en ambos casos es muy sencillo.

Una vez que hemos creado los Scripts en el directorio de Rclone que hayamos decidido, les daremos permiso de ejecución para que no sea necesario llamar al comando `sh` y con poner la ruta completa del script sea suficiente. Esto nos evitará algunos problemas que he visto que surgen con el comando pidof.

```
chmod a+x /home/Scripts/*
```

* **a+x:** otorgamos permisos ejecución a todos los usuarios.
* **/home/Scripts/\*:** a todo el contenido que se encuentra en esta ruta. 

* Rclone sync: la siguiente configuración es sincronización cada 30 minutos.

![Rclone sync cron](https://dl.dropboxusercontent.com/s/qx21lyp39ke6kt6/Rclone_cron.png?dl=0)

* Rclone copy: la siguiente configuración es *rclone copy* los domingos a la 01:00.

![Rclone copy cron](https://dl.dropboxusercontent.com/s/59pmsddssoedc1c/Rclone_copy_cron.png?dl=0)

En la siguiente captura es como lo veríamos en nuestra terminal:

![Rclone cron cli](https://dl.dropboxusercontent.com/s/0gnyg7jx5tczbsh/Rclone_cron_cli.png?dl=0)

#### Ejecución cron alternativa:

Por la web de Rclone no he visto como poder controlar el tamaño de los log que se van generando. Sería interesante poder eliminar los datos antiguos, es decir establecer un límite de por ejemplo 10MB y cuando llegue a ese límite que vaya eliminando logs antiguos para poder introducir nuevos logs.

Como digo, no he visto forma de hacerlo por lo tanto voy a añadir al **Cron** un comando a ejecutar para que compruebe el tamaño de los logs y en caso de que supere el tamaño que nosotros le indiquemos (en mi caso serán 10MB) que realice una acción.

````
find /home/Logs/ -type f -size +10M -exec rm -f {} \;
````

 * **find:** comando utilizado para realizar búsquedas.
 * **/home/Logs/:** es la ruta en la que queremos buscar, podemos indicarle la que queramos. La ruta que he marcado es donde se encuentran los Logs de rclone.
 * **-type f:** es el tipo de fichero que queremos buscar (en linux todo es un fichero), en nuestro caso será un fichero de tipo *file*.
 * **-size +10M:** buscamos fichero mayor de 10 Megabytes. Si delante del 10 indicamos un `-` buscará ficheros menores a 10 Megabytes. Se puede establecer otras unidades de medida que se encuentran explicadas en el manual de find.
 * **-exec:** nos permite ejecutar una acción con todo aquello que ha encontrado (en función de las condiciones que le hemos marcado).
 * **rm -f:** rm = remove, es decir elimina lo que hemos encontrado, y como estamos seguros de que lo que ha encontrado lo queremos eliminar pasamos el parámetro *-f* para que no nos solicite confirmación.
 * **{}:** identifica a todo lo que ha encontrado el comando find.
 * **\;:** es la finalización del comando find cuando utilizamos la opción `-excec`.

[Man find](http://man7.org/linux/man-pages/man1/find.1.html)



**Logging:**

Utilizaremos dos opciones para monitorizar rclone.

[-v, --verbose](https://rclone.org/docs/#v-vv-verbose): rclone tiene 4 niveles de log, *ERROR*, *AVISO*, *INFO* Y *DEBUG*. Cuando utilizamos `-v` , rclone muestra los tres primeros niveles de error.

[--log-file=FILE](https://rclone.org/docs/#log-file-file): en combinación con la anterior, todo el log monitorizado lo enviamos a un fichero, que será el que establecemos después del símbolo `=`.

**Bwlimit:**

[--bwlimit](https://rclone.org/docs/#bwlimit-bandwidth-spec): con está opción limitamos el ancho de banda. En nuestro caso la utilizaremos para no ocupar todo el ancho de banda que nos proporciona nuestro ISP. Por la web se puede ver limitaciones de **8650k** (Hace referencia a 8650 KBytes, que si hacemos la operación matemática correspondiente son aproximadamente 750GB diarios, que es el limite para los Team Drive).

Según lo que se puede leer por algunas consultas realizadas al soporte de Google, no hay limitaciones, salvo en lo que se refiere a contenido de video (Por ejemplo si queremos utilizar GD para almacén del contenido de Plex) y de hosting.

Limitaciones de Google:
 * [Files you can store in Google Drive](https://support.google.com/drive/answer/37603?hl=es)
 * [Team Drive Limits](https://support.google.com/a/answer/7338880?hl=en); After you've uploaded 750 GB to a Team Drive in 1 day, you'll be blocked from uploading additional files that day. However, file uploads already in progress will complete, up to a 5 TB maximum for a single file.

**Update:**

[-u, --update](https://rclone.org/docs/#u-update): con esta opción obligamos a rclone que omita cualquier archivo que tenga una hora de modificación más reciente en destino que en origen. Además, si el archivo tiene la misma hora de modificación en destino como en origen, sólo se actualizará si el tamaño es diferente.

**Tpslimit:**

[--tpslmit](https://rclone.org/docs/#tpslimit-float): debido a los baneos de las cuentas de google drive (los baneos son de 24 horas, que es el tiempo que tarde Google en refrescar las *estadísticas*, desconozco si los baneos son mayores si el baneo es reiterado). Según lo que he podido leer en varios grupos de telegram y por internet, la cifra más acertada es 8, que son el número de peticiones máximas por segundo que haríamos a la API de Google.

**Transfers**:

[--transfers](https://rclone.org/docs/#transfers-n): es el número de transferencias en paralelo que se realizan. Por defecto rclone establece 4 transferencias en paralelo. Puesto que me parece un número excesivamente bajo, y aprovechando el ancho de banda que le hemos dados, vamos a elevar el número a 15.

**Exclude:**

Vamos a utilizar reglas de filtrado, para evitar incluir varios directorios cuando utilicemos la sincronización de una de las cuentas.

[--exclude](https://rclone.org/filtering/#exclude-exclude-files-matching-pattern): podemos utilizarlo de dos modos.

	1. --exclude nombre_del_fichero 
	2. --exclude-nombre_del_fichero

**¿Cuál es la diferencia entre ambos?** Con la primera opción enumeramos un fichero concreto, en cambio, con la segunda opción llamamos a un fichero que almacena diferente ficheros que queremos excluir. Evitamos tener que enumerar uno a uno los ficheros en el comando completo.



## Otros comandos útiles:

* **Listremotes:** nos permite listar todos los *remotes* que hemos configurado.

  ````bash
  rclone listremotes
  ````

* **List**: nos permite listar el contenido. Hay varios comandos que podemos ejecutar que nos mostrarán diferentes resultados.

  ````BASH
  #Lista los objetos que se encuentran en REMOTO.
  rclone ls REMOTO:
  #Lista los directorios que se encuentra en REMOTO con fecha y hora de creación.
  rclone lsd REMOTO:
  #Lista todo el contenido de la ruta que le indiques, tanto directorios como ficheros.
  rclone lsf REMOTO:
  #Lista todo el contenido (ficheros y directorios) incluido lo que se encuentra en todas las subcarpetas.
  rclone lsl REMOTO:
  ````

* **Tree**: nos muestra un lista de todo el contenido en forma de árbol. *(No lo recomiendo utilizar ya que tarda bastante en ejecutarse)*.

  ````
  rclone tree REMOTO:
  ````

![Rclone tree](https://dl.dropboxusercontent.com/s/18hofevtlq3t1j3/Rclone_tree.png?dl=0)

* **Help:** nos permite ver diferentes comandos o flags que podemos introducir.

  ````
  rclone --help
  rclone help flags
  rclone help backends
  ````

*Donde escribo **REMOTO** es necesario sustituirlo por el nombre de vuestra unidad configurada*



## Referencias:

* [^1]: [¿Qué son las unidades de equipos (Team Drive)?](https://support.google.com/a/answer/7212025?hl=es&ref_topic=7337266)

* [Funcionamiento de logging en rclone](https://rclone.org/docs/#logging)

* [Más filtros de Rclone](https://techwiki.co.uk/RClone_-_Filtering)

* [Google OAuth “invalid_grant” nightmare — and how to fix it](https://blog.timekit.io/google-oauth-invalid-grant-nightmare-and-how-to-fix-it-9f4efaf1da35)

* [Rclone commands](https://rclone.org/commands/)

* [Rclone configure](https://rclone.org/docs/#log-file-file)

