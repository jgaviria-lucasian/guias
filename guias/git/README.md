#GIT Basico

<!-- TOC -->

- [GIT Basico](#git-basico)
    - [A. Configurar](#a-configurar)
    - [B. Comandos](#b-comandos)
    - [C. Branch](#c-branch)
    - [D. Tags](#d-tags)
    - [E. Push/Pull a servidor](#e-pushpull-a-servidor)

<!-- /TOC -->
## A. Configurar

Configurar quien soy cuando hago algun commt 

Para listar toda la configuración que tengo 

```bash
git config --list
```

**Información clave a adicionar** 
user.name : Nombre completo
user.email : Correo Electronico
core.editor : Editor favorito (vi, nano, ...)

Hay tres niveles en los cuales se configuran las variables
Global (--global) : Se aplica a nivel del usuario de sistema operativo /home/usuario/.gitconfig
System  (--system) : Se aplica a nivel de maquina en el archivo /etc/gitconfig
Local (--local) : Solo aplica para la configuración de un repositorio

**Para establecer el valor de forma Global** 

```bash
git config --global user.name "Jaime Gaviria"
git config --global user.email "jgaviria@lucasian.com"
git config --global core.editor vi
```


Para ver un valor puntual, por ejemplo el user.name
```bash
git config user.name
```

Haciendo esto dado que lo configure en global (usuario), el archivo .gitconfig
```bash
jgaviria@argos [~] $ cat /home/jgaviria/.gitconfig 
[user]
	name = Jaime Gaviria
	email = jgaviria@lucasian.com
[core]
	editor = vi
```

## B. Comandos

**1. Inicializar**
Para iniciar a trabajar hay que crear un drectorio para definir que se trata de una carpeta que estara en un reporsitorio, esto lo hacemos mediante el comado 

git init

Para esto creo el directorio test el cual va a ser un directorio del repositorio y lo inicializo

```bash
jgaviria@argos [~] $ mkdir test
jgaviria@argos [~] $ cd test
jgaviria@argos [~/test] $ git init
Inicializado repositorio Git vacío en /home/jgaviria/test/.git/
```

**2. Clonar un repositorio**
Otra forma de iniciar es clonando una carpeta o un repositorio que se encuentre en otro lugar para lo cual hago uso del comando 

```bash
git clone URL_REPOSITORIO
```


**3. Añadir archivos (svn add)**
Para añadir archivo al staging index (antes de hacer commit), utilizamos el comando 

```bash
git add NOMBRE_ARCHIVO|REGEXP
```

**3. Eliminar archivos rm (svn delete)**
Para eliminar archivo del repositorio, utilizamos el comando 

```bash
git rm NOMBRE_ARCHIVO|REGEXP
```

**4. Ver el estado (svn status)**
Para ver como esta el estado de los archivos simplemente ejecuto 

```bash
git status
```

**5. Enviando cambios (commit)** 
Para hacer el commit y enviarlo al repositorio simplemente ejecuto 

```bash
git commit 
```

**6. Deshacer cambios** 
Para deshacer cambios utilizo el comando

```bash
git reset HEAD NOMBRE_ARCHIVO
```

O bien lo podemos volver al punto inicial con el comando 

```bash
git checkout -- NOMBRE_ARCHIVO
```


**7. Ver los cambios**
Para ver como ha cambiado el repositorio en el tiempo utilizo 

```bash
git log
```

Para ver solo la primera linea en el log 
git log --oneline


## C. Branch

Para crear una nueva rama simplemente ejecutamos 

```bash
git branch NOMBRE_BRANCH
```

Para ver los branch que tengo ejecuto 

```bash
git branch
```

El resultado sera 

```bash
jgaviria@argos [~/test] $ git branch
  CaracteristicaA
* master
```

Master es la rama principal.

**OJO : Se marca con * el branch en el cual me encuentro** 

Para cambiarme de branch ejecuto el comando 

```bash
git checkout NOMBRE_BRANCH
```

El resultado sera

```bash
jgaviria@argos [~/test] $ git checkout CaracteristicaA
Cambiado a rama 'CaracteristicaA'
jgaviria@argos [~/test] $ git branch
* CaracteristicaA
  master

```

Para crear el Branch y pasarse a este directamente, ejecurmos

```bash
git checkout -b NOMBRE_BRANCH
```

El resultado sera 

```bash
jgaviria@argos [~/test] $ git checkout -b CaracteristicaB
Cambiado a nueva rama 'CaracteristicaB'
jgaviria@argos [~/test] $ git branch
  CaracteristicaA
* CaracteristicaB
  master
```

Para borrar una rama utilizo el argumento -d 

```bash
git branch -d NOMBRE_BRANCH
```

Para fusionar las ramas 

1. Cambiamos a la rama destino (en este caso ire a master)
2. Solicito un merge con la rama que quiero mezclar (CaracteristicaB)

Para esto ejecuto 

```bash
git checkout master
git merge CaracteristicaB
```

Como resultado

```bash
jgaviria@argos [~/test] $ git checkout master
Cambiado a rama 'master'
jgaviria@argos [~/test] $ git merge CaracteristicaB
Actualizando f8b3db5..46ab141
Fast-forward
 jk | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 jk
```

Finalmente se recomienda eliminar la rama al terminar

## D. Tags
Cuando estoy trabajando con versiónes e historia puedo colocar nombres o valores mas claros a la versiones

Veo todos los commit que tenga 
```bash
jgaviria@argos [~/test] $ git log --oneline
46ab141 (HEAD -> master) Se aidciona archivo jk
f8b3db5 eliminado c
a5465eb aa
9ca9186 Elimino el archivo a
fc0c111 Este es un primer commit en el repo de test
```

Para colocar un tag a la versión puntual utilizo 

```bash
git tag NOMBRE_TAG NUMERO_VERSION
```

Por ejmplo 

```bash
git tag v0.0 fc0c111
```

Cunado estoy en una versión puedo colocar el tag simplemente colocando 

```bash
git tag NOMBRE_TAG
```

Para ver todos mis tags
```bash
git tags
```


Y puedo hacer un checkuout para ver como estaba en ese momento 

```bash
git checkout v0.0
```

Llamando 

```bash
jgaviria@argos [~/test] $ git tag v0.0 fc0c111
jgaviria@argos [~/test] $ git checkout v0.0
Nota: actualizando el árbol de trabajo 'v0.0'.

Te encuentras en estado 'detached HEAD'. Puedes revisar por aquí, hacer
cambios experimentales y confirmarlos, y puedes descartar cualquier
commit que hayas hecho en este estado sin impactar a tu rama realizando
otro checkout.

Si quieres crear una nueva rama para mantener los commits que has creado,
puedes hacerlo (ahora o después) usando -b con el comando checkout. Ejemplo:

  git checkout -b <nombre-de-nueva-rama>

HEAD está ahora en fc0c111 Este es un primer commit en el repo de test
jgaviria@argos [~/test] $ ls
a  b  c  d
```



Y me puedo devolver a master  (para volver a la ultima versión)

```bash
git checkout master
```

## E. Push/Pull a servidor

Para adicionar un repositorio remoto 

```bash
git remote add NOMBRE URL
```

Por ejemplo 

```bash
git remote add origin http://tets:87222/...
```


Para subir codigo al repositorio 

```bash
git push NOMBRE RAMA
```

Por ejemplo 

```bash
git puch origin master
```


Para bajar el codigo del repositorio con los cambios que han hecho otros 

```bash
git pull NOMBRE RAMA
```

Por ejemplo 

```bash
git pull origin master
```







