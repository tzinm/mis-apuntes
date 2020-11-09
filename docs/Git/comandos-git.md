### Git alias

Git nos permite trabajar con alias. Tienen un funcionamiento similar a los alias de Linux, es últil crear estos alias para evitar introducir largos comandos.

````bash
git conifg --global alias.nombre-del-alias "comando-git"
````

### Git stash

Este comando nos permite hacer un guardado de cambios que hayamos hecho en nuestro **Working Directory** pero sobre los que no queremos hacer por el momento ningún commit. 

```bash
git stash
```

Una vez que ejecutamos este comando, los cambios que hemos hecho desde que iniciamos el commit hasta la ejecución del comando desaparecerán del commit actual, pero serán almacenados para recuperarlos posteriormente cuando lo necesitemos.

Continuamos con nuestro proyecto, incluso realizando nuevos commit, y llega el momento que decidimos recuperar algunos de los cambios sobre los que no hemos hecho commit pero sí tenemos almacenados (WIP = Working In Progress). Listamos todos los cambios que hemos ido guardando.

```bash
git stash list
```

Una vez que los tenemos listados procedemos recuperarlos con el siguiente comando:

```bash
#Recuperamos todos los cambios almacenados
git stash apply

#Recuperamos un cambios específico
git stash apply stash{1}
```

[Guardado rápido y limpieza](https://git-scm.com/book/es/v2/Herramientas-de-Git-Guardado-r%C3%A1pido-y-Limpieza#r_git_stashing)

### Git Reset

Hay ocasiones, y no son pocas, en las que queremos volver atrás en el tiempo (siempre refiriéndonos a los commit) y para ello tenemos el comando **git reset**. Dependiendo de qué contenido queremos recuperar, este comando tiene diferentes opciones que son importante conocer.

Antes de conocer los tres argumentos que podemos utilizar es necesario conocer alguna peculiaridad de este comando.

* Cuando invocamos `git reset` sin argumentos, de forma implícita se está ejecutando `git reset --mixed HEAD`.
* Si no especificamos **commit id**, el comando se invocará con `HEAD` como commit objetivo.

!!!note
	**HEAD** es sinónimo de "último commit".

#### Parámetro hard

De las tres opciones que tenemos esta es la más "bestia" (también peligrosa)  ya que restablece el **staging area** y el **working directory**. Esto significa que todo el trabajo que se encontrase en estos dos "árboles" se perderá.

```bash
git reset --hard commit_id
```

#### Parámetro mixed

El argumento **mixed** no realiza modificaciones sobre el **working directory** como si hace el argumento hard. Es decir, movemos el "puntero" `HEAD` al commit que hayamos especificado y restablecemos el **staging area**. 

Ejecutar `git reset` junto al argumento mixed es muy útil cuando queremos resumir varios commit en uno. Esto se debe a que trasladamos todos los cambios al working directory y después podemos ejecutar `git add` y `git commit -m` para crear el nuevo commit.

```bash
git reset --mixed commit_id
```

#### Parámetro soft

Solo realiza cambios de "puntero", mantiene el **working directory** y el **staging area** sin alterar. 

```bash
git reset --soft commit_id
```

​	!!!note
Si necesitamos movernos entre diferentes commit sin restablecer ninguno de los tres árboles de git, podemos ejecutar `git checkout`(estudiaremos este comando más adelante) el cual funciona del mismo modo que al ejecutarlo para movernos entre diferentes ramas de un proyecto.

### Git diff

Este comando nos permite comparar cambios a nivel de commit, ramas, archivos, etc.

```bash 
git diff commmit_id
```

Para conocer más a fondo este comando podemos revisar [este enlace](https://www.atlassian.com/es/git/tutorials/saving-changes/git-diff) que nos llevará a los tutoriales de [Atlassian](https://es.m.wikipedia.org/wiki/Atlassian)

### Git commit --amend

El parámetro **amend** nos permite modificar el commit más reciente, es decir el último commit de nuestro proyecto. ¿Qué podemos conseguir?

* Combinar un nuevo commit que vayamos a hacer junto con el último commit en el "historial", con lo que se creará un nuevo commit con la "mezcla" de ambos. El comando debería ejecutarse del siguiente modo:

```bash
#Modificando el mensaje
git commit -m "Descripción" --amend

#Sin modificar el mensaje
git commit --amend --no-edit
```

* Editar el mensaje del último commit. Cuando ejecutemos el comando se nos abrirá un editor para realizar dicha modificación. La ejecución en este caso es más simple.

```` bash
git commit --amend
````

!!!note
	Si ejecutamos el último comando `git commit --amend -m "Descripción actualizada"` se actualizará el mensaje sin necesidad de abrir el editor para realziar cambios.

