## Cron-apt

Es el gestor que se encuentra en OMV para la **gestión de paquetes y actualizaciones** del sistema. Esta herramienta permite automatizar la ejecución de varios comandos de **apt-get** para mantener elsistema actualizado.

Una vez descargado el programa ejecutando `apt-get install cron-apt` (para otras distribuciones consultar la guía de instalación) hay varios ficheros a tener en cuenta. 

En el directorio `/etc/cron-apt/config.d/` no se encuentra ningún fichero por defecto.

- Fichero **config** que se encuentra en la ruta `/etc/cron-apt/`

```bash
# When to send email about the cron-apt results.
# Value: error   (send mail on error runs)
#        upgrade (when packages are upgraded)
#        changes (mail when change in output from an action)
#        output  (send mail when output is generated)
#        always  (always send mail)
#                (else never send mail)
MAILON="upgrade"

# Value: error   (syslog on error runs)
#        upgrade (when packages is upgraded)
#        changes (syslog when change in output from an action)
#        output  (syslog when output is generated)
#        always  (always syslog)
#                (else never syslog)
SYSLOGON="always"


OPTIONS="-o Acquire::http::Dl-Limit=25"
```

- Dos ficheros que se encuentran en el directorio `/etc/cron-apt/action.d/`. Los dos ficheros son los siguientes:

  - **0-update:**
  - **3-download:**

Con las líneas que se encuentran en estos dos ficheros se realizará una actualización de la lista local de paquetes, descargará los paquetes para los que se encuentre una versión más actual, pero no realizará la instalación de estos. Respondiendo al fichero de configuración, enviará un email a la cuenta configurada para la recepción de estos.

Cuando instalamos esta herramienta se añade automáticamente un fichero al directorio `/etc/cron.d/` llamada **cron-apt** que contiene los comandos que ejecutará cron. Como veremos en el script, **cron-apt** se ejecuta todas las noches a las 4 de la mañana.

```bash
# Regular cron jobs for the cron-apt package
#
# Every night at 4 o'clock.
0 4	* * *	root	test -x /usr/sbin/cron-apt && /usr/sbin/cron-apt
# Every hour.
# 0 *	* * *	root	test -x /usr/sbin/cron-apt && /usr/sbin/cron-apt /etc/cron-apt/config2
# Every five minutes.
# */5 *	* * *	root	test -x /usr/sbin/cron-apt && /usr/sbin/cron-apt /etc/cron-apt/config2
```

En **Openmediavault** cuando generamos una tarea desde la interfaz web, este es **añadido** en la ruta `/etc/cron.d/openmediavault-userdefined`.  El contenido de este fichero se muestra del siguiente modo:

```bash
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user    command
@daily root /var/lib/openmediavault/cron.d/userdefined-e68e3a58-58b9-49e4-92c4-aed639dbe2c1 | mail -E -s "Cron - Rclone copy" -a "From: Cron Daemon <root>" root >/dev/null 2>&1
```

Además del directorio **cron.d**, podemos encontrarnos los directorios **cron.daily**, **cron.hourly**, **cron.monthly** y **cron.weekly**, que almacena los scripts en función de la frecuencia con la que se ejecutarán.

El fichero `/etc/crontab` contiene las referencias a los directiorios mencionado en el párrafo anterior. Este fichero contiene las siguientes líneas:

````bash
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
````

#### ¿Cuál es su funcionamiento?

Este fichero envía una orden con el comando **run-parts** para que los scripts que se encuentren en el directorio **hourly** se ejecuten en el minuto 17 de cada hora. A su vez, con el resto de directorios, antes de enviar la orden con run-parts comprueba si **anacron** tiene permisos de ejecución. 

Para ello utiliza el operador lógico **OR** (**||**). Puesto que en el sistema anacron se encuentra con los permisos de ejecución, cron no realizará ninguna tarea con los scripts que se encuentren en los directorios **daily**, **weekly** y **monthly**. Esta tarea la realizará anacron como veremos explicado en el próximo apartado ([Anacron](#Anacron)).

Es necesario que el servicio **crond** se encargue del directorio **hourly** ya que **anacron** sólo puede ejecutar como máximo una tarea diaria.



## Anacron

Este servicio es ejecutado al inicio del sistema. Es un **complemento a cron** ya que permite comprobar si alguna tarea que se le haya especificado no ha sido ejecutado, y en tal caso la ejecutará.

El fichero **anacrontab** que nos encontramos en **OpenMediaVault** es el siguiente:

```bash
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
HOME=/root
LOGNAME=root

# These replace cron's entries
1	5	cron.daily	run-parts --report /etc/cron.daily
7	10	cron.weekly	run-parts --report /etc/cron.weekly
@monthly	15	cron.monthly	run-parts --report /etc/cron.monthly
```

Este fichero ejecutará todos los scripts que se encuentren en los directorios **cron.daily**, **cron.weekly** y **cron.monthly**.

Los valores que vemos en cada línea expresan lo siguiente:

- Primera columna: `period in days` — frequency of job execution in days

  The property value can be defined as an integer or a macro (`@daily`, `@weekly`, `@monthly`), where `@daily` denotes the same value as integer 1, `@weekly` the same as 7, and `@monthly` specifies that the job is run once a month regarless of the length of the month.

- Segunda columna: `delay in minutes` , número de minutos que anacron espera hasta ejecutar el comando.

- Tercera columna: `job identifier` es un nombre que identifica a la tarea que se está ejecutando, es útil para cuando se analicen logs.

- Cuarta columna: el comando a ejecutar.

Anacron se basa en el **timestamp** para saber si la tarea no ha sido ejecutada y en tal caso ejecutarla. De ahí que en los directorios **daily**, **monthly** y **weekly** haya un fichero llamado **0anacron**, siendo el script que actualizará las marcas de tiempo (**timestamp**). 

#### Script timestamp:

````bash
#!/bin/sh
#
# anacron's cron script
#
# This script updates anacron time stamps. It is called through run-parts
# either by anacron itself or by cron.
#
# The script is called "0anacron" to assure that it will be executed
# _before_ all other scripts.

test -x /usr/sbin/anacron || exit 0
anacron -u cron.daily
````


