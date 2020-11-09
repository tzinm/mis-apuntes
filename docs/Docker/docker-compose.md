Es una herramienta que nos ayuda a orquestar contenedores en Docker, gestionar los diferentes contenedores de los que depende una aplicación. Existen aplicaciones que para su correcto funcionamiento dependen de varios servicios, para seguir con la simplicidad de "**un servicio = un contenedor**", Docker Compose nos permitirá administrar los diferentes contenedores de forma grupal. 

Docker Compose trabaja con ficheros de tipo **yaml**, en los que se definen los contenedores, volúmenes, redes, etc. Después de completar el fichero, `docker-compose` como comando se encarga de la lectura del fichero y lanzar todos los contenedores definidos en dichero fichero.

Durante la instalación de Docker-CE esta herramienta no se instala, por lo que será necesaria la instalación de forma independiente. En la [documentación](https://docs.docker.com/compose/install/) oficial se encuentran diferentes guías para la instalación en los diferentes Sistemas Operativos.

En la máquina actual, en la que estamos trabajando bajo Linux, la instalación se haría del siguiente modo:

<ol><li>Descargamos Docker-Compose.</li>
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

<li>Aplicar permisos de ejecución al ejecutable que hemos descargado.</li>

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

</ol>### Comenzando con Docker Compose

El nombre del fichero que debemos utilizar es **docker-compose.yml**, donde definiremos los diferentes contenedores.

Este fichero cuenta con cuatro grandes partes:

#### Version

Para saber que número de versión (Docker-Compose) debemos establecer, buscamos en la documentación cual es la última versión, suele ser la que se recomienda utilizar. Esta línea dentro del fichero es obligatoria.

#### Services

Los servicios hacen referencia a los contenedores. En primera instancia definimos los nombres de cada servicio (podemos elegir el que queramos), y debajo de ellos irán los parámetros que hacen referencia al propio contenedor, como el nombre del contenedor, la imagen a la que hace referencia, puertos, variables de entorno, etc.

#### Volúmenes

<ul><li><b>Volúmenes nombrados</b>: funcionan del mismo modo que ejecutando <code>docker run</code>. Lo que hacemos es definir en primer lugar el volumen que crearíamos con la instrucción <code>docker volume create</code> en "volumes" con el nombre que queramos. A continuación lo definimos dentro del servicio.</li>
```yaml
version: '3'
services:
  web:
    image: nginx
    container_name: nginx-prueba
    volumes:
      - "html:/usr/share/nginx/html"
volumes:
  html:
```

<li><b>Volúmenes host</b>: no necesitamos la parte <b>volumes</b> dentro del fichero <b>docker-compose.yml</b> sino que directamente en el contenedorlo podemos parametrizar.</li></ul>

```yaml
version: '3'
services:
  web:
    image: nginx
    container_name: nginx-prueba
    volumes:
      - "/home/miusuario/html:/usr/share/nginx/html"
```

#### Networks

* **Red Host**: [es importante establecer la versión 3.4 para que funcione correctamente](https://stackoverflow.com/questions/47074457/how-to-specify-docker-build-network-host-mode-in-docker-compose-at-the-time/47545276#47545276). Para que funcione es necesario añadir **build**, y además añadir el [contexto](https://docs.docker.com/compose/compose-file/#build), que hace referencia a un directorio que contenga un Dockerfile o una url a un repositorio git. Si establecemos la ruta relativa hace referencia a la ubicación del archivo de composición. 

```yaml
version: '3.4'
services:
  web:
    build:
      context: .
      network: host
    image: nginx
    container_name: nginx-prueba
    ports:
      - "8181:80"
```

* Creación de una nueva red con subnet. **Aún no se permite establecer gateway**.

```yaml
version: '3'
services:
  web:
    image: nginx
    container_name: nginx-prueba
     ports:
      - "8181:80"
    networks:
      red-prueba:
        - ipv4_address: 192.168.50.10
networks:
  red-prueba:
    ipam:
      driver: default
      config:
        - subnet: "192.168.50.0/24"
```

En cuanto a las redes, cuando ejecutamos el comando <code>docker-compose up -d</code> se genera una red específica para <b>docker-compose</b>. Si no definimos una red, la red utilizada será la red por defecto (docker-compose_default). 

Cuando se crea la red, el nombre que es otorgado "<b>directorioactual_nombredelared</b>". Es posible modificar la parte del nombre que hace referencia al "directorioactual", pasando el parámetro <code>-p</code>.

Por lo tanto si nuestro directorio actual se denomina <b>docker-compose</b> y queremos que el prefijo sea por ejemplo **dcprueba** ejecutaríamos el siguiente comando:

```bash
#Modificar el nombre de red a "dcprueba_default"

docker-compose -p dcprueba up -d
```

Por otro lado, tenemos dos partes importantes que se utilizan a la hora de crear contenedores.

- **Variables de entorno:** estas se pueden definir de dos modos.

=== "Fichero docker-compose.yml"
      ```yaml
      environment:
        - "VARIABLES1=docker"
      ```

=== "Fichero con extensión .env."
      ````yaml
      environment:
        - variables.env
      ````

* **Command:**  se utiliza para establecer un CMD al contenedor.

```yaml
commmand: mkdir /bin/bash
```

El comando `docker-compose` se encarga de realizar un proceso similar que el comando `docker run`, pero en este caso los diferentes parámetros se encuentran definidos en un fichero. Por lo tanto, las [políticas de reinicio](#Política de reinicio) o la [limitación de recursos](limitando recursos) también se pueden definir.

* Políticas de reinicio:

```yaml
#Reinicio siempre
restart: always

#Reinicio hasta que lo detengamos de forma manual
restart: unless-stopped

#Únicamente se reinicia el contenedor en caso de que haya habido un fallo
restart: on-failure
```

* Limitar recursos:

```yaml
#Limitar memoria
mem_limit: 20m

#Cpu
cpuset: "0"
```

Otras de las directivas relevantes de este fichero es **depends_on**, la cual hace referencia a las dependencias que tiene se contenedor respecto a los que se incluyan en esta directiva.

```yaml
version: '3'
services:
  web:
    image: nginx
    container_name: nginx-prueba
    depends_on: db
  db:
  image: postgres
  container_name: nginx-database
```

El último paso será conocer como podemos eliminar un contenedor creado con la herramienta Docker-Compose. Para ello debemos ejecutar el siguiente comando, situados en el directorio donde se encuentra el fichero **yaml**.

```bash
docker-compose down
```

El comando anterior sigue el siguiente proceso:

1. Detiene el contenedor.
2. Elimina el contenedor.
3. Elimina la red que ha creado por defecto.

### Docker-compose build

Ya conocemos el comando `docker build`, es el encargado de crear imágenes. El comando [`docker-compose build`](https://stackoverflow.com/a/39988980/12868523) tiene un funcionamiento similar, se encarga de generar una imagen a partir de la definición de **build** en el fichero **yaml**. Esta imagen será utilizada como imagen base en la creación del contenedor.

Para que se construya una imagen, sabemos que necesitamos un fichero **Dockerfile**, en el cual se encuentran las capas para la construcción de una imagen. En la sentencia **build** nos encontramos con dos parámetros importantes.

* **Context:** se indica la ruta donde se encuentra el **Dockerfile** que utilizaremos para crear la imagen. Si se encuentra en el mismo directorio se utilizará el punto (**.**).
* **Dockerfile:** el nombre del fichero si este es diferente al nombre por defecto (Dockerfile).

```yaml
version: '3'
services:
  web:
    container_name: web
    image: web-test
    build:
      context: .
      dockerfile: Dockerfile1
```

!!!note
	Si el nombre que hace referencia al Dockerfile no se ha modificado, podemos obviar estos dos parámetros.

```yaml
version: '3'
services:
  web:
    container_name: web
    image: web-test
    build: .
```

