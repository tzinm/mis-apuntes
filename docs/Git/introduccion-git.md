### Introducción

Git es un software de control de versiones, es decir un software que "almacena" un historial de los cambios que hagamos sobre aquello que hayamos establecido para controlar. 

<ol>
  <li><b>Área Local - Working Directory</b>: es el área de trabajo, donde tenemos los recursos sobre los que queremos llevar un control.</li>
<li><b>Área Local - Staging Area</b>: es el área donde se preparan los archivos que se enviarán al repositorio. Es el espacio donde se "encapsula" el contenido para después realizar un commit sobre el mismo. No es necesario almacenar en este área todo el contenido que tenemos en el <b>Working Directory</b>.</li>
<li><b>Repositorio</b>: aquí se encuentra el registro de los cambios que hemos realizado y que hemos decidido "almacenar".</li>
</ol>


El flujo más habitual de **Git** es el siguiente:

<ol>
  <li>Trabajamos sobre los archivos que se encuentran en el <b>Working Directory</b>.</li>

<li>Una vez que hay modificaciones en el Working Directory pasamos a prepar los archivos añadiéndolos al <b>Staging Area</b>.</li>

<li>Confirmamos los cambios, esto significa que se recoge los archivos tal y como se encuentran en el área de preparación y se almacenan esas instántaneas de manera permanente en el repositorio.
 </li>
</ol>

!!!note
	Este proceso se lleva a cabo cada vez que necesitamos registrar algún cambio en Git.

### Instalación de Git

En la web [git-scm.com](https://git-scm.com) podemos encontrar información para descargar **Git** en las diferentes plataformas en las que se encuentra disponible. Entre estas plataformas tenemos Windows, MacOS y Linux. Además de la versión **cli** (línea de comandos), también dispone de versión con interfaz gráfica.

Después de seguir con el proceso de instalación de Git, podemos comprobar si se encuentra instalado ejecutando el comando `git --version`.