### Crear Ramas

Para crear ramas en git tenemos dos formas de hacerlo.

* Crear una rama

```bash
git branch nombre_de_rama
```

* Crear una rama y movernos a ella

````bash
git checkout -b nombre_de_rama
````

Ejecutando `git branch` no obtendremos ningún resultado por pantalla, pero podemos comprobar si la nueva rama se ha creado correctamente.

````bash
git branch #Este comando mostrará un listado de las ramas locales
````

### Movernos entre ramas

El comando que necesitamos para movernos de una rama a otra nos resultará familiar.

```bash
git checkout nombre_de_rama
```

Otro comando interesante que nos permite movernos a la rama anterior en la que nos encontrábamos cuando nos hemos movido de forma rápida.

```` bash
git checkout -
````

Este comando también es utilizado para descartar cambios que hayamos hecho en el Working Directory, es decir cambios sobre los que no hemos ejecutado el comando `git add` o `git commit`. 

````bash
git checkout -- nombre-fichero
````

### Cambiar nombre rama

```bash
git branch -m nombre_rama nombre_rama_nuevo
```

!!!note
	No es necesario indicar el **nombre_rama** si la rama que quieres modificar es en la que te encuentras.

Con el comando anterior únicamente conseguimos modificar el nombre de la rama local. El objetivo es que tanto la rama local como la rama remota (por ejemplo la rama del proyecto en GitHub) tengan el mismo nombre, con lo que deberemos ejecutar los siguientes comandos además del anterior.

````bash
#Eliminar rama remota
git push origin --delete nombre_rama

#Crear la nueva rama remota
git push origin -u nombre_rama
````

!!!note
	La opción **-u** (--set-upstream) se utiliza habitualmente cuando disponemos de varias ramas en nuestro proyecto. De este modo estamos "enlazando" la rama local con la rama remota. Cuando se ejecute `git push`, este enviará los cambios a la rama correspondiente.

### Fusionar ramas

Para fusionar ramas es necesario situarse en la rama que va a recibir los cambios hechos en esa segunda rama que queremos fusionar. Habitualmente cuando vamos a fusionar una rama lo haremos con la rama principal (no siempre tiene por qué ser así) que es la rama master.

```bash
git merge nombre_de_rama #Será la rama que queremos fusionar
```

Seguido de ejecutar el comando nos aparecerá en la terminar para poder insertar un mensaje de commit, ya que a partir de la fusión estaremos creando un commit nuevo a partir de las ramas fusionadas. Esto es así siempre y cuando no haya ningún conflicto entre ramas, de ahí que distingamos dos tipos de fusiones.

- **Fast-forward**: es la fusión que "simple", cuando no existe ningún conflicto la rama principal absorbe la rama "secundaria" y crea un commit a partir de las dos ramas fusionadas.
- **Manual Merge**: esto se da cuando se han modificado partes iguales de un mismo fichero en las ramas que vamos a fusionar. En el momento que ejecutamos `git merge` se nos mostrará un mensaje de error indicando que hay un conflicto y que debemos resolverlo. Debeos resolver el conflicto manualmente, aunque existen herramientas que nos facilitan el trabajo.

### Rama rebase

**Rebase**, a diferencia de **merge**, nos permite reescribir el historial de commit. Ambos comandos pretenden integrar una rama en otra, pero cada uno de ellos lo hace de forma diferente. En este caso lo que conseguimos con rebase es mantener el historial de commit de forma lineal, es decir, cuando hacemos un rebase lo que conseguimos es posicionar los commit (dependerá de las opciones agregadas al comando) de la rama actual por delante de la rama master (puede ser cualquier otra rama). Lo que conlleva este proceso es la reescritura del historial.

Plantear este método como _fusión_ tiene ciertas implicaciones, tanto positivas como negativas, que es necesario conocer.

#### Ventajas

* Historial de proyecto lineal.
* Evitamos crear los commit innecesario referentes a la fusión tal y como sucede con el comando `git merge`.

#### Desventajas

* A la vez que conseguimos un historial lineal alteramos dicho historial, con lo cual corremos el riesgo de perdida de commit y de alterar el proyecto en el que colaboramos.
* Perdemos trazabilidad por tener un historial lineal.

Los comandos más simples que entran en acción cuando queremos llevar a cabo rebase son los siguientes:

* Situarse en la rama sobre la que queremos realizar el rebase.

````bash
git checkout nombre_rama
````

* Ejecutamos el comando rebase, se tendrán en cuenta todos los commit de dicha rama.

```bash
git rebase master
```

* Una vez que los commit de la rama *secundaria* se han situado por delante de la rama master, nos movemos a esta.

````bash
git checkout master
````

* Por último moveremos la rama master hasta el último commit de la rama *secundaria*, para ello es necesario realizar una fusión **fast-forward**.

````bash
git merge nombre_rama
````

Para conocer este maravilloso comando en profundidad, tanto la parte interactiva como las recomendaciones, es imprescindible visitar estos dos enlaces:

* [Git rebase](https://www.atlassian.com/es/git/tutorials/rewriting-history/git-rebase)
* [Merging vs rebasing](https://www.atlassian.com/es/git/tutorials/merging-vs-rebasing)

### Eliminar ramas

En algunas ocasiones es necesario eliminar ramas, ya sea después de haber realizado fusionado la rama o porque no vamos a continuar trabajando sobre dicha rama.

````bash
git branch -d nombre_rama
````

El comando anterior nos permite eliminar una rama local, pero si queremos eliminar una rama remota tenemos dos opciones.

* Opción 1:

  ````bash
  git push origin :nombre_rama
  ````

* Opción 2:

  ````bash
  git push origin --delete nombre_rama
  ````

