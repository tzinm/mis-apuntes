Los repositorios **bare** son repositorios remotos propios, es decir, prescindimos de servicios de hosting de git como GtiHub por un repositorio remoto de nuestra propiedad. Dos características importantes que debemos destacar.

- No cuentan con **Working Directory**.
- Se trabaja a través de **push** y **pull**.

Es frecuente su uso cuando deseamos trabajar de forma centralizada y por diversas razones no podemos/queremos utilizar servicios de hosting de git. Dispondremos de un servidor en el que tendremos un directorio que contendrá todo nuestro proyecto (semejante al Working Directory). 

Una vez que tenemos el servidor con el directorio respondiente, dentro del directorio debemos ejecutar un comando para iniciar git.

```bash
git init --bare nombre-repositorio.git
```

!!!note
	Es necesario que el nombre del repositorio termine el `.git`.

El próximo paso es clonar en local el repositorio que acabamos de crear en el servidor. Esto nos permitirá interactuar con el *bare repositorie* a través de los comandos `git push` y `git pull`.

```bash
git clone /ruta/nombre-repositorio.git /ruta//working-directory-local
```
