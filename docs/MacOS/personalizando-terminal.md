## Personalizando el terminal

Powerline es una utilizar que nos permite personalizar nuestra terminal. 

Las instalación de esta herramienta es sencilla, aunque necesitaremos el gestor de paquetes **pip** proporcionado por **Python**.

* Instalando **python** a través del gestor de paquetes [Homebrew](https://brew.sh/index_es)

```bash
brew install python
```

* Instalando **powerline**

```bash
pip3 install --user powerline-status
```

Para comprobar la ruta donde se ha instalado esta utilidad ejecutaremos el siguiente comando:

```bash
pip3 show powerline-status
```

Aparecerá una serie de información, entre ellas una línea que comienza por **"Location"**, la cual nos informa de la ruta. Es importante tenerla en cuenta ya que será necesario para las siguientes configuraciones.

Lo siguiente que haremos será introducirlo en el fichero **.bash_profile** (es uno de los ficheros utilizados por bash para configurar el etorno del sistema) de nuestro usuario. En caso de que este fichero no se encuentre en nuestro sistema podemos crearlo ejecutando `vim .bash_profile`.

A este archivo le añadiremos lo siguiente:

```bash
#Añadimos python a la variable PATH
export PATH=$PATH:$HOME/Librar/Python/3.7/bin

#Habilitamos powerline
powerline-daemon -q
POWERLINE_BASH_CONTINUATION=1
POWERLINE_BASH_SELECT=1
source $HOME/Library/Python/3.7/lib/python/site-packages/powerline/bindings/bash/powerline.sh
```



### Copiando la configuración

Para una edición posterior de los ficheros de configuración que nos permitirán personalizar powerline al extremo, es necesario copiar el directorio `config_flies` a nuestro $HOME. Para ello ejecutaremos los siguiente comandos:

```bash
#Creamos el directorio en nuestro $HOME
mkdir ~/.config/powerline
#Copiamos los ficheros de configuración
cp -R $HOME//Library/Python/3.7/lib/python/site-packages/powerline/config_files/* ~/.config/powerline
```



### Instalando las fuentes

```bash
# Clonación del repositorio
git clone https://github.com/powerline/fonts.git --depth=1
# Instalación de las fuentes
cd fonts
./install.sh
# Eliminamos el repositorio descargado
cd ..
rm -rf fonts
```

Como se puede observar en el script de instalación (**install.sh**), al realizar la instalación en MacOS las fuentes se almacenan en la ruta `$HOME/Library/Fonts`.



### Problemas a solucionar

1. Para que la terminal muestre correctamente los iconos es necesario cambiar la fuente en la configuración del terminal, a una de las descargadas compatibles con powerline.
2. En caso de que nos encontremos trabajando con git y queramos que muestra la rama de trabajo es necesario realizar una pequeña modificación en el fichero de configuración **config.json**. En el bloque **shell** debemos establecer como "theme", **default_leftonly**, tal que así:

```json
		"shell": {
			"colorscheme": "default",
			"theme": "default_leftonly",
			"local_themes": {
				"continuation": "continuation",
				"select": "select"
			}
```

​		Después debemos ejecutar el siguiente comando:

```bash
powerline-daemon --replace
```



### Referencias

[Documentación Powerline](https://powerline.readthedocs.io/en/latest/index.html)

[Shell configuration](http://hayne.net/MacDev/Notes/unixFAQ.html#shellStartup)

[How to install Powerline to pimp you Bash prompt (for Mac)](https://medium.com/@ITZDERR/how-to-install-powerline-to-pimp-your-bash-prompt-for-mac-9b82b03b1c02)