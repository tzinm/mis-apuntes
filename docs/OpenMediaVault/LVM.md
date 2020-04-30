#Logical Volume Management

LVM es un gestor de volúmenes lógicos (Logical Volume Manager) utilizado en Linux. Este sistema de almacenamiento permite establecer una capa lógica entre el sistema de archivos y las particiones de los diferentes dispositivos de almacenamiento. Gracias a este sistema se puede combinar diferentes tipos de almacenamiento, como pueden ser particiones, RAID, etc.



## PV - Volumen físico:

Hace referencia al dispositivo de almacenamiento, pudiendo ser un disco duro, una partición, un RAID, es decir, cualquier dispositivo de bloque.

**Comandos a utilizar:**
* Añadiendo un disco completo al volumen físico

  ````
  pvcreate /dev/sdX - X será la letra del disco que queramos añadir
  ````

![LVM pvcreate](https://dl.dropboxusercontent.com/s/0s89wpe2lgfbwq3/LVM_pvcreate.png?dl=0)

* Mostrando información de los volúmenes físicos:

  ```pvs ```

![LVM pvs](https://dl.dropboxusercontent.com/s/um5dujo4dxn09y1/LVM_pvs.png?dl=0)

```bash
pvdisplay /dev/sdX - X será la letra del disco sobre el que queremos obtener la información
```

![LVM pvdisplay](https://dl.dropboxusercontent.com/s/8ad5yxc68jl2bi0/LVM_pvs_display.png?dl=0)



## VG - Grupo de volúmenes:

Podría considerarse como un disco duro virtual, donde agruparemos los volúmenes físicos que consideremos oportunos. Un grupo de volúmenes puede contener un único volumen físico, pero siempre disponemos la posibilidad de aumentar el tamaño de un *vg* añadiendo más volúmenes físicos a dicho grupo de volúmenes.

**Comandos a utilizar:**

Al igual que con los comandos de los volúmenes físicos, veremos dos grupos de comandos, uno para crear los grupos de volúmenes y otros para mostrar la información.

* Creando un grupo de volúmenes:

  ````
  vgcreate nombre_del_grupo /dev/sdX
  ````

  * Nombre_del_grupo --> podemos establecer el nombre que queramos, aprovecharemos para darle un nombre que le identifique y sea fácil reconocer que tipo de datos introduciremos en él.
  * /dev/sdX --> será el dispositivos que identifica al volumen físico.

![LVM vgcreate](https://dl.dropboxusercontent.com/s/4sfnkmxcj4faic1/LVM_vgcreate.png?dl=0)

* Obteniendo información de los volúmenes físicos:

  ````
  vgs
  ````

![LVM vgs](https://dl.dropboxusercontent.com/s/5xz46takxaj00rh/LVM_vgs.png?dl=0)

​	

```
vgdisplay
```

![LVM vgdisplay](https://dl.dropboxusercontent.com/s/b9jv623xsl9gfc6/LVM_vgdisplay.png?dl=0)



## LV - Volumen lógico:

Aquí es donde se crearan los sistemas de ficheros, para hacer la analogía a los discos duros tradicionales, sería como una partición. Un volumen lógico puede crecer siempre y cuando el grupo de volúmenes al que pertenece disponga de espacio disponible para asignárselo.

**Comandos a utilizar:**

Al igual que los anteriores veremos comandos que idénticas funcionalidades.

* Creación de volúmenes lógicos:

  ````
  lvcreate -L 500G -n nombre_del_lv nombre_del_vg_perteneciente
  ````

![LVM lvcreate](https://dl.dropboxusercontent.com/s/62fr6yyg9obm9no/LVM_lvcreate.png?dl=0)

	* -L --> indicamos el tamaño en Megabytes (M), en Gigabytes (G), Terabytes (T), etc. En la siguiente fotografía se puede ver de todas las opciones que disponemos.

![Tamaño disponible -L](https://dl.dropboxusercontent.com/s/izm4py2lvv8ixj2/LVM_lvcreate_L.png?dl=0)

> * nombre_del_lv --> elegimos un nombre que identifique al volumen lógico.
> * nombre_del_vg_perteneciente --> debemos establecer el nombre del grupo de volúmenes al que va a pertenecer.

* Obteniendo información de los volúmenes lógicos:


![LVM lvs](https://dl.dropboxusercontent.com/s/n7btngsyhjrgpsq/LVM_lvs.png?dl=0)

```bash
lvdisplay
```

![LVM lvdisplay](https://dl.dropboxusercontent.com/s/gileu5fi8citeoh/LVM_lvdisplay.png?dl=0)

Si queremos saber que volúmenes físicos está utilizando nuestro volumen lógico debemos ejecutar `lvdisplay -m`.

![LVM lvdipslay -m](https://dl.dropboxusercontent.com/s/tzrl2i7x26d3mc6/LVM_lvdisplay_m.png?dl=0)

Una vez llegados hasta aquí, tenemos los volúmenes lógico en *crudo*, es decir, nuestro siguiente paso será crear el sistema de fichero (le otorgaremos ext4) mediante comandos con *make file system (`mkfs`)* o a través de la interfaz web de OMV (así lo realizaremos).

En el panel lateral de OMV seleccionaremos **Sistema de Archivos** y deberemos pulsar el botón **crear** que vemos en la siguiente captura.

![LVM OpenMediaVault](https://dl.dropboxusercontent.com/s/tnzxn2joz5j5zyt/LVM_sistema_ficheros.png?dl=0)

Se abrirá una pequeña ventana donde deeberemos elegir el dispositivo (en este caso será el volumen lógico que hayamos creado), podemos asignarle una etiqueta para identificarlo en la interfaz web de OMV y por último seleccionar el Sistema de Archivo (ext4 como habíamos comentado).

![LVM OpenMediaVault creación Sistema de Archivos](https://dl.dropboxusercontent.com/s/o6akgvfpqej1cgb/LVM_OMV.png?dl=0)

Nos mostrará el proceso, no suele tardar demasiado tiempo. Una vez que termine cerramos la venta y debemos montar la unidad para que este disponible y podamos utilizarla como almacenamiento. Esto también lo podemos realizar desde la interfaz web. Debemos seleccionar la unidad que deseamos montar y pulsar el botón de **montar** tal y como se ve en la siguiente captura.

![LVM Montaje unidad](https://dl.dropboxusercontent.com/s/tzkkk8xsenw4cbe/LVM_montaje.png?dl=0)

**Referencias:**

* [LVM para torpes I](https://blog.inittab.org/administracion-sistemas/lvm-para-torpes-i/)
* [LVM para torpes II](https://blog.inittab.org/administracion-sistemas/lvm-para-torpes-ii/)
* [LVM para topres III](https://blog.inittab.org/administracion-sistemas/lvm-para-torpes-iii-ampliando-espacio/)

