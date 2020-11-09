Un contenedor se entiende como una capa adicional que mantiene en ejecución las capas que se han definido en el fichero Dockerfile, por este motivo es posible crear tantos contenedores como se deseen a partir de la misma imagen. A diferencia del resto de capas, esta capa es de lectura y escritura (**rw**), por lo tanto nos permite realizar modificaciones sobre las capas anteriores cuando el contenedor se encuentra en ejecución. 

Todas estas modificaciones son temporales puesto que no sobrescriben el fichero Dockerfile, por lo tanto una vez que eliminemos el contenedor esas modificaciones se perderán (algunas modificaciones pueden ser persistentes mediante el uso de volúmenes que veremos más adelante).

**¿Qué nos encontramos dentro de un contenedor?**

- Imagen
- Volúmenes (se utilizan para mantener persistente cierto contenido)
- Redes (útil para comunicar contenedores entre sí)

### Contenedor VS Máquina Virtual

El contenedor es un proceso aislado más del sistema, por lo tanto compartirá el mismo hardware que esté utilizando el sistema operativo anfitrión, es decir, no necesitamos asignarle recursos específicamente (aunque es posible limitar los recursos utilizados por un contenedor).

En cambio, cuando creamos una máquina virtual debemos asignarle una serie de recursos, como pueden ser el número de nucleos, la memoria ram o el espacio en disco duro. En este caso obtenemos una máquina completa dentro del propio sistema anfitrión.

La principal diferencia que nos podemos encontrar es la ligereza, un contenedor está destinado a ser muchísimo más ligero que una máquina virtual, habitualmente un contenedor se utiliza para ejecutar una única aplicación concreta.

### Docker run

Este comando se utiliza para iniciar los contenedores. Aunque parece evidente, no puede haber dos contenedores con el mismo nombre, aunque uno de ellos se encuentre parado.

```bash
docker run -d jenkins -p 80:8080
```

* **-d**: permite correr un contenedor en segundo plano.

* **jenkins**: es el nombre de la imagen utilizada.

* **-p**: permite mapear los puertos. El puerto que se encuentra en el lado izquierdo es el que utilizará el sistema anfitrión, el puerto de la derecha es el interno del contenedor.

* **-e:** asignación de variables de entorno.

En algunas ocasiones si no definimos la capa **CMD** es posible que el contenedor se muera. Si queremos evitar que esto sucede podemos utilizar los parámetros **-i** y **-t** para interactuar con el contenedor.

### Limitando recursos

Por defecto Docker utiliza los recursos de la máquina anfitriona sin límite alguno, a pesar de ello los contenedores consumen muy pocos recursos. Podemos limitar tanto el uso de **CPU** como de **RAM**.

```bash
#Limitando la memoria RAM
docker run -d --memory "100mb" --name recursos-limitados centos 
```

Cuando se muestran las estadísticas con el comando `docker stats` ya no mostrará la RAM limite de nuestra máquina, sino que los que hayamos establecido,  en este caso serán 100MB. Podemos pasar la limitación en **bytes** (b), **kilobytes** (k), **megabytes** (m) o **gigabytes** (g).

```bash
#Limitando el número de cores de la CPU
docker run -d --cpuset-cpus 0-3 --name reclimited2 centos
```

Los núcleos (o cores) se pueden definir mediante un rango (como hemos hecho en el ejemplo anterior) o numerándolos y como separación una coma.

- **0-2** --> utilizaríamos los núcleos 0, 1 y 2.
- **0,2** --> utilizaríamos los núcleos 0 y 2.

### Políticas de reinicio

Establecemos que sucede cuando un contenedor se "apaga" de forma inesperada. Disponemos de las siguientes opciones:

- **No** (es la política por defecto): el contenedor no se reiniciará automáticamente. Si hacemos una prueba deteniendo cualquier contenedor que tengamos en funcionamiento, veremos que este se detiene y ejecutando el comando `docker ps` no aparece. 
- **Always**: con esta opción el contenedor siempre se reiniciará.
- **Unless-stopped:** se reiniciará siempre, salvo que sea detenido manualmente.
- **On-failure**: se reiniciará en caso de que el contenedor se haya detenido por algún fallo.

La política de reinicios se establece ejecutando el parámetro **--restart**.

```bash
docker run --restart unless-stopped 
```
