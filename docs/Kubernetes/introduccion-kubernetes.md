Es una tecnología desarrollada inicialmente por Google, aunque actualmente pertenece a Cloud Native Computing Foundation gracias a que fue donada por parte de Google. La primera versión de este sistema fue liberada el 21 de julio de 2015.

Kubernetes es un sistema de código abierto utilizado para automatizar el despliegue de aplicaciones containerizadas. Básicamente se encarga de orquestar los contenedores que se despliegan (el proceso de despliegue se automatiza), de forma transparente los distribuye entre los diferentes nodos (máquinas físicas o virtuales que componen el clúster) que conforman Kubernetes.

Por lo tanto, el usuario no tiene que preocuparse de la gestión física de los servidores, el clúster es interpretado como un conjunto de recursos que se encuentran disponibles para el despliegue de las diferentes aplicaciones.

Kubernetes facilita una API para trabajar con el clúster, que se utiliza para automatizar el proceso de despliegue ajustando el ciclo de vida de la aplicación. Por ejemplo, establecer cuantos recursos necesita la app para funcionar correctamente, "healtchecks" que informan si la aplicación sigue funcionando o seleccionar mediante etiquetas en que nodos se ejecutarán nuestras aplicaciones.

**¿Qué ventajas nos aporta Kubernetes?**

- **Escalabilidad** --> automatizar el despligue y la replicación de contenedores.
- **Confiabilidad** --> reemplazar contenedores en caso de que estos "mueran".
- **Eficiencia** --> recursos disponibles mejor optimizados.

### Características relevantes

**Inmutabilidad**: a diferencia de los sistemas operativos que hemos conocido hasta la actualidad, los cuales reciben actualizaciones incrementales reemplazando los ficheros ya existentes, los sistemas inmutables, las actualizaciones se aplican construyendo un sistema completamente nuevo, en el ambito en el que nos encontramos diríamos que se construye una nueva imagen.

**Declarative configuration**: todo lo que nos encontramos en Kubernetes es un objeto creado a través de un fichero en el que declaramos el estado de dicho objeto. Kubernetes se encarga de mantener el estado tal y como lo hemos definido.

**Self-healing systems**: la "tarea" de Kubernetes se centra, como hemos dicho, en mantener el estado del sistema tal y como lo hemos definido. Realiza tareas como la recreación de los contenedores que hayan dejado de funcionar, cumpliendo con lo declarado en el fichero de configuración.

### Componentes Kubernetes

#### Clúster

Un clúster es un grupo de nodos. Si hablamos físicamente, será un grupo de máquinas que trabajarán conjuntamente. El cluster es el que contendrá todo este sistema, del que partirán el resto de partes que lo conforman.

#### Nodo

Es la primera parte con la que nos encontramos cuando queremos desglosar el clúster de Kubernetes. Habitualmente está representado por una máquina (**workers**), por lo tanto cada máquina de un clúster podría interpretarse como un nodo.

Dentro de este "grupo" nos encontramos con dos tipos:

- **Nodo Master**: es el nodo que "central" que controla el resto de nodos. En este nodo se ejecutan varios procesos.
  
  * [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)
  - [kube-controller-manager](%5Bhttps://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)
  
  - [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)

- **Nodos**: en este grupo nos encontraríamos con el resto de nodos, los que son controlados desde el Master. Cada uno de estos nodos ejecutan los siguientes procesos.
  
  - **kubelet** --> es el encargado de comunicarse con el Master.
  
  - **kube-proxy**--> implementa los servicios de red de Kubernetes.

Una vez que los nodos están en funcionamiento, el único nodo con el que interactuaremos será con el Master, que es el que recibe las instrucciones cuando hacemos uso del comando `kubectl`.

#### Namespaces

Nos permiten ogranizar los diferentes objetos dentro del clúster. Podemos entender cada **namespace** como directorio que manteine un grupo de objetos. El namespace por defecto es `default`. Se puede utilizar un namespace diferente, lo veremos más adelante.

#### Pod

Es el objeto que representa uno o varios contenedores utilizados para desplegar nuestra aplicación. Es habitual que un pod sea exclusivamente un único contenedor. Todos los contenedores que se encuentra en un pod siempre están en la misma máquina, es decir en el mismo nodo.

Las aplicaciones/contenedores que se encuentra en el mismo pod comparten **dirección ip**, **espacio de puertos** (un mismo puerto no puede ser utilizado por más de un contenedor) y el mismo **hostname** (se comunican entre ellos utilizando **localhost**). Por tanto, las aplicaciones que se encuentran en diferentes pods se encuentran aisladas unas de otras.
