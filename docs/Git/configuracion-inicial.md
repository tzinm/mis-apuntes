### Configuración inicial

Lo primero que debemos hacer es aplicar una serie de configuraciones globales. Estas configuraciones globales que haremos servirán para añadir metadatos a los diferentes commit que vayamos haciendo.

* Nombre de usuario:

```bash
git config --global user.name "nombre"
```

- Email:

```bash
git config --global user.email "email"
```

- Listar las configuraciones globales:

```bash
git config --list
```

Git se encuentra rastreando de forma constante nuestro **Working Directory** en busca de cambios que se hayan realizado. Dentro de estos cambios se encuentran las modificaciones, eliminación y creación de ficheros y/o directorios. Para que Git realice su trabajo de rastreo es necesario ejecutar el comando para que inicie dicho rastreo.

```bash
git init
```

!!! note
	El comando ha de ejecutarse dentro del directorio que queremos rastrear.

A pesar de que Git rastrea continuamente qué cambios hacemos, no almacenará ningún cambio hasta que nosotros se lo indiquemos explícitamente. Podemos ver en que estado se encuentra nuestro **Working Directory**.

```bash
#Vista normal
git status

#Para ver de un modo resumido ejecutaremos:
git status --short
```

Después de ejecutar el comando `git status`, si ha habido algún tipo de cambio en el directorio, podemos observar que hay contenido dentro de nuestro Working Directory modificado o del que aún no llevamos ningún control. Si nos encontramos con algún cambio, tendremos que ejecutar una serie de comandos:

- Añadiendo el contenido al **Staging Area**.

```bash
#Para añadir un fichero o más los haremos escribiendo su nombre
git add 404.html

#Para añadir todo el contenido disponible (dos opciones)
git add -A

git add .
```

- Una vez que el contenido está preparado, crearemos la referencia (**commit**) a esos cambios.

```bash
git commit -m "Creación inicial del proyecto"
```

Estas dos operaciones, `git add` y `git commit` se pueden unir en una única operación.

```bash
git commit -am "Creación inicial del proyecto"
```

### Historial del proyecto

Hay un término muy conocido en el mundo de la informática que hace referencia a registros que se almacenan en un archivo, lo cuales hacen referencia a cambios o acontecimientos surgidos a lo largo del tiempo. A este termino se le denomina **log**, y en git se utiliza para hacer referencia al historial de commit.

```bash
git log
```

Este comando tiene muchos parámetros que podemos pasar y modificar la salida del comando. Por ejemplo si tenemos muchos commit, la forma que muestra por defecto la salida del comando no es demasiado cómoda, por lo que será interesante pasar diferentes parámetros como:

- **-\-oneline**: muestra una línea por commit. Commit id + título del commit.
- **-\-graph**: muestra de forma gráfica los commit. Se combina con --online y --decorate.
- **-\-date=relative**: fecha relativa a la fecha del sistema. Es decir mostrará hace "x" minutos.
- **--stat**: muestra qué cambios ha habido en cada commit.
- **--pretty=format:"formato personalizado"**: damos formato al log.
- **-n**: donde n es un número para mostrar los últimos commit en función del número que hayamos puesto.
- **--grep="mensaje"**: como el comando grep de la terminal, busca por mensaje. Si añadimos `-i` no distinguirá entre mayúsculas y minúsculas.
- -**-nombre_fichero**: nos busca en que commit se encuentra.

```bash
#Podemos ver que miembros han contribuido y cuantos commit han hecho cada uno
git shortlog
```

### Gitignore

Es un fichero oculto (su nombre nos lo indica **.gitignore**) con el que evitamos que ciertos archivos sean tenidos en cuenta por git. En el fichero se pueden definir ficheros por su nombre completo, o utilizar [expresiones regulares](https://refrf.shreyasminocha.me/) (regex).

<ul><li>Podemos hacer uso de <b>/</b>, terminando con este símbolo para referirnos a un directorio.</li>

<li>El signo de exclamación <b>!</b> al inicio denota negación.</li>
<li>Directorios anidados. Se expresa del siguiente modo:

  <ul><li>directorio1/**/directorio2, coincidiría con directorio1/otrodirectorio/directorio2</li>
	    <li>doc/**/*.txt ignorará todos los .txt que se encuentren bajo el directorio doc, aunque estén en otro directorio dentro de doc.</li></ul>

</li></ul>