Podemos entender los tags (etiquetas en castellano) como **puntos relevantes** del repositorio, habitualmente utilizados para versionar el proyecto en el que nos encontramos trabajando.

Existen dos formas de crear tags:

- **Annotated tags (etiquetas anotadas):** nos permite generar un mensaje en la etiqueta. 

````bash
git tag -a tag-elegido -m "mensaje"
````

- **Lightweight tags (etiquetas ligeras):** únicamente establecemos la etiqueta, a pesar de ello podemos añadir una descripción.

!!!note
	La gran diferencia entre estos dos tipos de etiquetas es que las **anotadas** permiten almacenar los metadatos (nombre, correo electrónico, fecha, etc.) tal y como hacen los commit. Se recomienda el uso de estas por encima de las etiquetas ligeras.

Los tags nos proporcionan la ventaja de poder movernos en el historial a través del propio tag en vez de hacerlo a través del **commit id** como se hace habitualmente. 

````bash
git checkout tag
````

Siguiendo con los tags, disponemos de otra serie de comandos para trabajar con ellos.

* Listar tags disponibles

```bash
git tag
```

* Buscar una tag en particular

```bash
git tag -l "tag"
```

* Asignar un tag a un **commit id**

```bash
git tag -a tag commit_id
```

* Subir tags

````bash
git push tags
````

