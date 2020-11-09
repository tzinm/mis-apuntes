El cliente de Kubernetes que nos permite controlarlo es **kubectl**. Es una herramienta de línea de comandos para interactuar con la API de Kubernetes, utilizada para gestionar la mayor parte de objetos de Kubernetes, tales como los pods, servicios, namespaces, etc. Además permite conocer el estado general del clúster.

Esta herramienta se encuentra disponible en los diferentes Sistemas Operativos. En este caso la instalación se ha realizado en **MacOS Catalina** y en **Fedora 30**, que serán los equipos desde controlaremos el clúster de Kubernetes.

```bash
brew install kubectl
```

```bash
#Añadir repositorio
cat <<EOF> /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    basurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOF

#Descargar el paquete
sudo dnf install kubectl
```

Para más información: [Install kubectl](https://kubernetes.io/es/docs/tasks/tools/install-kubectl/)

Una vez que tenemos kubectl instalado localmente, necesitamos un fichero **kubeconfig** (fichero utilizado para configurar el acceso al clúster) para que kubectl se comunique con la API de Kubernetes. Aprovecharemos el fichero que se ha generado en el nodo master, lo descargaremos y guardaremos en el directorio `$HOME/.kube`, que es el directorio donde kubectl busca el fichero con el nombre **config**.

Podemos visualizar el fichero con kubectl.

```bash
kubectl config view
```

Si el fichero de configuración no tiene el nombre por defecto (config), deberemos utilizar la opción `--kubeconfig=fichero-configuracion`.

```bash
kubectl config --kubeconfig=fichero-configuracion view
```

[Set the KUBECONFIG environment variable](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#set-the-kubeconfig-environment-variable) - [Organize cluster access kubeconfig](https://kubernetes.io/es/docs/concepts/configuration/organize-cluster-access-kubeconfig/)

Si ya disponemos de conexión con el clúster podremos conocer tanto la versión de **kubectl** como de la **API de Kubernetes**.

```bash
kubectl version    
```

!!! note
    Para facilitar el uso de **kubectl** podemos habilitar el autocompletado ejecutando el comando `sudo kubectl completion bash > /etc/bas_completion.d/kubectl` [^1].

El siguiente paso será comprobar el estado del clúster.

```bash
kubectl get cs
```

El resultado será similar a esto:

```bash
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
```

- **Scheduler**: es el responsable de distribuir los pods en los diferentes nodos del clúster.

- **Controller-manager**: es el encargado de correr varios controladores que regulan el estado del clúster, por ejemplo que las replicas que hayamos definido se encuentren disponibles.

- **Servidor etcd**: es el lugar en donde se almacenan todos los objetos.

Listamos todos los nodos del clúster.

```bash
kubectl get nodes
```

Tal y como hemos visto en estos dos últimos comandos, `kubectl get recurso` (pods, services, nodes, etc.) nos muestra un listado de los recursos en el **namespace** en el que nos encontramos (recordamos que **default** es el namespace por defecto). 

Podemos ampliar la información que nos muestras utilizando la opoción `-o wide`. Además, otra de las opciones interesantes es `--no-headers`, suprime los encabezados.

Otro de los comandos importantes que nos provee **kuibectl** es `kubectl desribe recurso objeto`. La salida de este comando muestra información muy completa del objeto que le hayamos pasado.

Por ejemplo podemos solicitar la información de un nodo del clúster en particular.

```bash
kubectl describe nodes rpi4
```

### Creando objetos

Los objetos en Kubernetes se definen en un fichero con sintaxis **json** o **yaml**. Estos ficheros se utilizan tanto para crear, actualizar o eliminar dichos objetos.

Para crear o actualizar un objeto utilizaremos el siguiente comando.

```bash
#Este comando nos permite crear y actualizar un objeto
kubectl apply -f fichero.yaml
```

Disponemos de la posibilidad de actualizar los objetos de forma interactiva, es decir se abre un editor en la terminal, realizaremos las modificaciones que deseemos y una vez que guardemos Kubernetes se encargará de aplicar los cambios.

```bash
kubectl edit recurso objeto    
```

!!! note
    Es importante tener cuidado con los cambios que se realizan al vuelo puesto que estos no modifican el manifiesto, por lo que lo más recomendable es aplicar cambios con el comando `kubectl apply -f`. 

El comando para eliminar los objetos es muy similar al de crearlos. Cuando eliminamos un pod, todos los datos almacenados dentro de los contenedores asociados al pod son eliminados. Para que esto no suceda disponemos de **volúmenes persistentes** que veremos más adelante.

```bash
 kubectl detete -f fichero.yaml
```

```bash
 kubectl delete recurso objeto
```

### Desplegando objetos

Disponemos varias formas de desplegar pods en nuestro clúster. La forma más sencilla es ejecutando `kubectl run`. Por ejemplo si queremos desplegar un pod de Nginx como servidor web.

Podemos desplegar un pod ejecutando el siguiente comando:

```bash
kubectl run nombre-pod --image=nginx:latest
```

En el flag `--image` podemos añadir una imagen local o una imagen que se encuentra alojada en el registry oficcial de Docker.

Ejemplo de un manifiesto de pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - image: nginx:latest
      name: nginx-prueba
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
```

!!! info
    El fichero yaml que vemos será el fichero base que utilizaremos para añadirle nuevas secciones que iremos conociendo a lo largo del proyecto.

### Gestión de recursos

Kubernetes no solo trata que el despliegue de aplicaciones sea más sencillo, sino que además intenta que el aprovechamiento de los recursos esté mejor optimizado.

Nos encontramos con dos formas para la gestión de los recursos.

- **Resource requests**: la cantidad mínima de recursos que necesita una aplicación para funcionar.
- **Resource limits**: la cantidad máxima de recursos que una aplicación puede hacer uso.

!!! note
    Los recursos se especifican por cada contenedor, no por cada pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - image: nginx:latest
      name: nginx-prueba
      resources:
        requests:
          cpu: "500m"
          memory: "128Mi"
        limits:
          cpu: "1000m"
          memory: "256Mi"
      ports:
        - name: web
          containerPort: 80
          protocol: TPC
```

Cuando el sistema se queda sin memoria, **kubelet** se encarga de "matar" los contenedores que están utilizando más memoria de la que hemos definido. Los contenedores volverán a iniciarse con un consumo menor de recursos ajustados a los que hayamos definido.

### Volúmenes

Los volúmenes son una herramienta muy útil cuando trabajamos con contenedores, nos permite mantener datos más haya del contenedor que los haya generado. Los principales motivos por los que deseamos utilizar volúmenes:

- **Sincronización**: acceso a por parte de más de un contenedor a los mismos datos.

- **Caché**: datos como miniaturas que no son relevantes pero que se utilizan para mostrar algo dentro de una aplicación. Si los datos los almacenamos en un volúmen no será necesario crearlos cada vez que el pod sea recreado. Evitamos un consumo extra de recursos.

- **Datos persistentes**: datos que queremos que se mantengan en el tiempo a pesar de que el pod del que dependían no vaya a utilizarse.

#### ¿Cómo se definen en el fichero yaml?

- **Nivel spec**: se definen los volúmenes que pueden ser accedidos por el/los contenedores de un pod. Si un pod dispone de más de un contenedor no es necesario que todos ellos utilicen los volúmenes definidos. Además existe la posibilidad de utilizar discos remotos, como por ejemplo volúmenes NFS.

!!! info
    Es interesante utilizar servicios remotos como NFS porque nos permite que los datos persistentes sean independientes del nodo donde se encuentre el Pod.

- **Nivel container**: en este apartado se define el volumen del contenedor que irá asociado al volumen definido en el apartado anterior.

A continuación podemos ver un ejemplo con dos volúmenes que podemos utilizar en los contenedores que contenga el Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  volumes:
    - name: "nginx-data"
      hostPath:
        path: "/nginx/my-web"
    - name: "nginx-data-nfs"
      nfs:
        server: rpi-nfs.local
        path: "/exports"
  containers:
    - image: nginx:latest
      name: nginx-prueba
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: "nginx-data"
      ports:
        - name: web
          containerPort: 80
          protocol: TPC
```

### Labels

Nos facilita la identificación de los diferentes objetos de nuestro clúster. Esto nos permite **agrupar**, **visualizar** y **operar** de una forma más sencilla.

```bash
--labels="version=1,app=nginx"
```

Podemos añadir manualmente **labels** a un objeto que ya hemos creado.

```bash
kubectl label pods nginx "env=lab"
```

También podemos eliminar un **label** (o varios) que esté asociado a un objeto. Se utiliza un el símbolo **-** para ello.

```bash
kubectl label pods nginx "app-"
```

Mostrar columnas adicionales con la información referente a los **labels**.

```bash
kubectl get pods -L app
```

El resultado nos muestra una columna adicional que hace referencia a la clave del label que hemos indicado en el comando anterior. En dicha columna aparecerá el valor que tiene asignado dicha clave. Si el pod no tiene asignada **clave:valor**, no aparecerá nada.

```bash
NAME     READY   UP-TO-DATE   AVAILABLE   AGE     APP
nginx    1/1     1            1           6d23h   nginx
nginx2   1/1     1            1           3d1h    
```

Una de las funciones que nos aportan los **labels** es la posiblidad de filtrar objetos.

```bash
kubectl get pods --selector="version=1"
```

Podemos utilizar una serie de operadores para utilizar junto al flag **--selector**.

| Operador                   | Descripción                                        |
| -------------------------- | -------------------------------------------------- |
| key=value                  | **Key** es igual a **value**                       |
| kye!=value                 | **Key** es diferente a **value**                   |
| key in (value1, value2)    | **Key** puede ser tanto **value1** como **value2** |
| key notin (value1, value2) | **Key** no es ni **value1** ni **value2**          |
| key                        | **Key** está definida                              |
| !key                       | **Key** no está definida                           |

Por ejemplo podemos mostrar los pods que se correspondan tanto con la versión 1 como con la versión 2.

```bash
kubectl get pods --selector="version in (1,2)"
```

También podemos utilizar una estructura similar en los fichero **yaml**.

```yaml
selector:
  matchLabels:
    app: nginx
  matchExpressions:
    - {key: ver, operator: IN, values: [1, 2]}
```

### ReplicaSet

Un ReplicaSet nos garantiza que un número concreto de instancias (lo definimos nosotros) de un Pod estén siempre operativas. La recomendación es aplicarlo exclusivamente a Pods sin estado (**stateless**), es decir que no tengan sesiones como puede ser una base de datos, ya que podría darnos problemas. Este objeto nos ofrece una serie de ventajas.

- **Redundancia** --> la posibilidad de disponer de varias instancias de un mismo Pod nos proporciona **tolerancia a fallos**.
- **Escalabilidad** --> nos permite repartir el "trabajo" entre las diferentes instancias.

El manifiesto de un ReplicaSet debe contener lo siguiente:

1. Un nombre que identifique al propio ReplicaSet.
2. En la sección **spec** es necesario indicar el número de replicas.
3. Podemos añadir una sección más para definir una serie de Pods en el manifiesto del ReplicaSet. No es necesario añadir esta seccion (**template**) ya que el ReplicaSet a través de los labels será capaz de identificar los Pods que debe "adoptar".

Podemos comprobar si un Pod está siendo controlado por un ReplicaSet. Para ello ejecutaremos `kubectl get pods nombre-pod -o yaml` que nos dará como resultado el siguiente contenido.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-f89759699-b662g
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: nginx-f89759699
    uid: 8743e705-b5a0-4f04-8e7b-09acea2418da
```

Observamos que `controller: true` y es del tipo ReplicaSet `kind: ReplicaSet`.

En ocasiones es posible que neceistemos escalar de forma urgente y temporal un set de Pods, para ello tenemos el comando `kubectl scale`.

```bash
kubectl scale replicasets nginx --replicas=8
```

Con el ejemplo anterior hemos escalado hasta ocho Pods, lo que nos permitirá en un momento determinado recibir más peticiones en nuestro servidor web Nginx.

!!! note
    Para que estos cambias sean permanente deberemos actualizar el manifiesto correspondiente al ReplicaSet.

### Health Checks

Es la forma de mantener una aplicación viva en todo momento, si una aplicación falla Kubernetes se encarga de reiniciarla. Kubernetes nos proporciona dos formas *health checks* que se utilizan para objetivos diferentes, ambas se definen a nivel de contenedor.

* **Liveness**: verifica a nivel lógico de aplicación si esta está funcionando correctamente.

* **Readiness**: verifica cuando un contenedor está preparado para recibir peticiones.

Cuando tenemos definidos cualquiera de estos dos *health checks*, podemos verificar el estado de los pods ejecutando `kubectl describe pods nombre-pod`.

A continuación vemos un ejemplo de **livenessProbe** que hemos aplicado al deployment de Ghost.

```yaml
spec:
  containers:
    livenessProbe:
      tcpSocket:
        port: ghost-port
      initialDelaySeconds: 30
      timeoutSeconds: 5
      periodSeconds: 20
      failureThreshold: 3
```

En este ejemplo hemos utilizado 

* **initialDelaySeconds**: es el tiempo en segundos que debe esperar Kubernetes desde que el contenedor se ha iniciado para iniciar **livenesProbe**.

* **timeoutSeconds**: es el tiempo en segundos de espera hasta que **livenessProbe** entiende que el contenedor está fallando.

* **periodSeconds**: cuanto tiempo en segundos espera Kubernetes para realizar la siguiente prueba.

* **failureThreshold**: el número de intentos que realizará **livenessProbe**.

### Deployment

Es el objeto que se utiliza habitualmente para el despliegue de aplicaciones ya que engloba la creación de un Pod y un ReplicaSet. Además es una forma sencilla para controlar el lanzamiento de nuevas versiones de nuestras aplicaciones debido a que nos permite movernos tanto hacía atrás (versiones antiguas) como hacía delante (versiones nuevas) facilmente.

Para que la gestión de las diferentes versiones de nuestra aplicación sea sencilla, este guarda un histórico de los cambios que se han ido desplegando. Es recomendable establecer un máximo de versiones, lo haremos en el manifiesto del Deployment.

```yaml
spec:
  revisionHistoryLimit: 10
```

Podemos ver el histórico de un deployment.

```bash
kubectl rollout history deployment nginx
```

El resultado nos muestra una lista con las diferentes versiones, desde la más antigua hasta la más nueva. Si queremos conocer más sobre una versión específica podemos utilizar el flag `--revision`, al que deberemos indicar el número de la columna **Revision**.

Es importante mantener actualizada la key **annotations**, ya que esta recibe la descipción de los cambios que hemos realizado en la versión que vamos a desplegar. Además es la información que aparece en la columna **Change-cause**.

```yaml
spec:
  template:
    metadata:
      annotations:
        kubernetes.io/change-cause: "Actualización versión 2"
```

Al igual que con otros objetos, podemos interactuar con este modificando su configuración "al vuelo" o modificando el manifiesto (lo más recomendable).

```bash
kubectl rollout undo deployment nginx
```

Este comando nos permite volver a la versión anterior del Pod nginx. Podemos utilizar el flag `--to-revision` para volver a una versión específica.

```bash
kubectl rollout undo deployment nginx --to-revision=2
```

!!! note
    Lo más recomendable es actualizar el manifiesto, modificar la versión y aplicarla con el comando `kubectl apply`.

Disponemos de dos estrategias diferentes para el despliegue de nuevas versiones:

* **Recreate**: esta estrategia no se utilizar para entornos de producción. El funcionamiento es sencillo y rápido, el ReplicaSet se encarga de las actualizaciones parando todas las instancias y recreandolas. No hay ningún tipo de control sobre los updates, por lo tanto tendremos el servicio parado durante un determinado tiempo y además es posible que este no vuelva a inciarse correctamente.

* **RollingUpdate**: es una estrategia más sofisticada y robusta que la anterior, pero implicia mayor demora en la actualización de todas la instancias. La actulización se va aplicando de forma incremental, actualizando las instancias en bloques hasta que todas hayan recibido la actualización. De este modo evitamos que haya una parada del servicio y en caso de fallo de la actualización que el servicio no vuelva a iniciarse. Esta estrategia mantiene durante el periodo de actualización tanto la nueva versión como la anterior funcionando de forma concurrente.
  
  Disponemos de varias configuraciones
  
  * **maxUnavailable**: se establece el número máximo de Pods que pueden no estar disponibles durante la actualización. 
  
  * **maxSurge**: controla cuantos recursos extras podemos definir para llevar a cabo la actualización. Si por ejemplo asignamos 50% sobre 10 Pods, creará 5 Pods extra durante la actualización para realizar el despliegue, para mantener 10 Pods de la versión anterior siempre en funcionamiento hasta que la actualización haya finalizado con éxito.
  
  ```yaml
  spec:
    strategy:
      rollingUpdate:
        maxUnavailable: 50%
        maxSurge: 50%
      type: RollingUpdate
  ```

!!! info
    En ambas configuraciones se puede definir como número entero o en porcentaje.

El **RollingUpdate** es la estrategia más adecuada porque es un despliegue por etapas, lo que asegura que las actualizaciones terminarán en estado *healthy*. El estado de cada Pod actualizado se determina con los **readiness checks**.

Ademas:

* **minReadySeconds**: es el tiempo mínimo de espera por cada etapa para poder pasar a la siguiente.
  
  ```yaml
  spec:
    minReadySencods: 60
  ```

* **progressDeadlineSeconds**: en este caso se indica el tiempo máximo que debe esperar por cada etapa. 
  
  ```yaml
  spec:
    progressDeadlineSeconds: 300
  ```

### Services

Según la definición que podemos encontrar en la [documentación oficial](https://kubernetes.io/docs/concepts/services-networking/service/) de Kubernetes, es una forma abstracta de exponer una aplicación que se encuentra en uno o más Pods como un servicio de red.

Una de las razones de la existencia de este objeto es la efimeridad de los Pods, estos pueden _morir_ y volver a _nacer_, por lo que a lo largo del tiempo la dirección IP de los Pods puede cambiar. Si disponemos de un conjunto de Pods que deben interactuar entre ellos (frontend vs backend) necesitamos una formula para que estos puedan comunicarse sin depender de su dirección IP.

Los servicios son un **endpoint** permanente (punto de entrada) que nos permite acceder a un conjunto de Pods. Un servicio utiliza los **labels** para _reconocer_ los Pods que deben estar detrás de un servicio.

Kubernetes nos ofrece tres tipos de servicios:

- **ClusterIP** (es el tipo de servicio por defecto): este servicio es expuesto con una dirección IP interna del clúster, por lo tanto solo es accesible desde dentro del clúster. 

- **NodePort**: este servicio expone un puerto de forma estática en todos los nodos, por lo que nuestros Pods serán accesibles desde el exterior realizando una petición a `IP:Puerto`.

- **LoadBalancer**: es un servicio utilizado en Plataformas Cloud que lo que ofrece es una única dirección IP que reenviará todo el tráfico al servicio correspondiente, que se encargará de realizar el reenvío a los Pods.

### Debug

Hay formas muy avanzadas de gestionar los logs, aunque kubectl nos proporciona un comando para ello.

```bash
kubectl logs pod
```

Si queremos mantener la terminal actualizada con la salida de los logs podemos utilizar el flag `-f`. Además, si el pod contiene más de un contenedor la opción `-c contenedor` nos permite revisar los logs de un contenedor en particular.

En el caso de que la información mostrada con los anteriores comandos no sea suficiente, podemos "entrar" dentro del contendor para buscar de forma más precisa posibles errores que hayan surgido.

```bash
kubectl exec -it pod -- bash
```

[^1]:[Autocompletado kubectl](https://kubernetes.io/es/docs/tasks/tools/install-kubectl/#habilitar-el-auto-completado-en-el-int%C3%A9rprete-de-comandos)
