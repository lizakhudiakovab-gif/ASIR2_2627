1. ¿Qué es Git?

Git es un sistema de control de versiones distribuido.

Sirve para:

Guardar versiones de un proyecto.
Volver a estados anteriores si algo sale mal.
Trabajar en equipo sin sobrescribir el trabajo de otros.
Mantener un historial de los cambios realizados.

Ejemplo:

Imagina que estás haciendo una práctica y hoy guardas una versión.

Mañana rompes el código.

Con Git puedes volver a la versión de ayer.

2. ¿Qué es GitHub?

GitHub es una plataforma en Internet donde se almacenan repositorios Git.

Piensa así:

Git = herramienta local.
GitHub = servidor donde guardas repositorios para compartirlos.

Ejemplo:

Plain Text
Tu PC (Git) <---> GitHub (Internet)
3. Instalación de Git
Ubuntu
Shell
sudo apt-get update
sudo apt-get install git
Windows

Descargar desde:

Plain Text
https://git-scm.com/downloads
4. Configuración inicial

Git necesita saber quién eres.

Shell
git config --global user.name "Liza"
git config --global user.email "liza@gmail.com"

Comprobar configuración:

Shell
git config --list
5. Partes de un repositorio Git

Existen 4 zonas principales:

Workspace

Es donde trabajas normalmente.

Plain Text
archivo.txt

Aquí modificas archivos.

Staging Area

Zona temporal.

Aquí seleccionas qué cambios quieres guardar.

Shell
git add archivo.txt
Local Repository

Almacén local de commits.

Shell
git commit
Remote Repository

Repositorio en GitHub.

Shell
git push


1. Configurar Git
¿Qué se hace?

Se indica a Git quién eres para que registre tu nombre y correo en cada cambio.

Comandos
git config --global user.name "Tu Nombre"


¿Para qué sirve? Guarda tu nombre de usuario en Git.

git config --global user.email "correo@correo.com"


¿Para qué sirve? Guarda tu correo electrónico en Git.

git config --list


¿Para qué sirve? Muestra toda la configuración actual de Git.

2. Crear un repositorio
¿Qué se hace?

Se convierte una carpeta normal en un proyecto controlado por Git.

Comando
git init


¿Para qué sirve? Crea el repositorio Git y la carpeta oculta .git, donde se almacena todo el historial.

3. Comprobar el estado de los archivos
¿Qué se hace?

Se consulta qué archivos han cambiado y en qué estado están.

Comando
git status


¿Para qué sirve? Muestra:

Archivos nuevos (untracked).
Archivos modificados (modified).
Archivos preparados (staged).
Rama actual.

Es el comando más utilizado.

4. Preparar archivos para guardarlos
¿Qué se hace?

Se seleccionan los archivos que se incluirán en la próxima versión.

Comandos
git add archivo.txt


¿Para qué sirve? Añade un archivo al área de preparación (staging area).

git add -A


¿Para qué sirve? Añade todos los cambios realizados en el proyecto.

5. Guardar una versión del proyecto
¿Qué se hace?

Se crea una versión permanente de los cambios.

Comando
git commit -m "Descripción de los cambios"


¿Para qué sirve? Crea un commit que guarda una copia de los cambios realizados.

6. Modificar el último commit
¿Qué se hace?

Se corrigen errores en el último commit.

Comandos
git commit --amend


¿Para qué sirve? Añadir archivos olvidados al último commit.

git commit --amend -m "Nuevo mensaje"


¿Para qué sirve? Cambiar el mensaje del último commit.

7. Sacar archivos del área de preparación
¿Qué se hace?**

Quitar un archivo del staging porque todavía quieres seguir editándolo.

Comando
git reset HEAD archivo.txt


¿Para qué sirve? Devuelve el archivo al Workspace sin eliminar los cambios.

8. Deshacer cambios
¿Qué se hace?

Volver a la última versión guardada.

Comandos
git checkout -- archivo.txt


o actualmente

git restore archivo.txt


¿Para qué sirve? Elimina los cambios realizados y recupera la versión guardada en el último commit.

⚠️ Los cambios se pierden.

9. Borrar archivos
¿Qué se hace?**

Eliminar archivos controlados por Git.

Comando
git rm archivo.txt


¿Para qué sirve? Borra el archivo y registra su eliminación para el próximo commit.

Después normalmente:

git commit -m "Eliminar archivo"

10. Renombrar o mover archivos
¿Qué se hace?

Cambiar el nombre o ubicación de un archivo.

Comando
git mv archivo.txt nuevo.txt


¿Para qué sirve? Renombra o mueve un archivo manteniendo el seguimiento de Git.

11. Clonar un repositorio remoto
¿Qué se hace?

Copiar un proyecto de GitHub al ordenador.

Comando
git clone URL


¿Para qué sirve? Descarga completamente un repositorio remoto.

12. Conectar un repositorio local con GitHub
¿Qué se hace?

Vincular tu proyecto con un repositorio remoto.

Comando
git remote add origin URL


¿Para qué sirve? Añade la dirección del repositorio de GitHub.

Comprobar:

git remote -v


¿Para qué sirve? Muestra los repositorios remotos configurados.

13. Subir cambios a GitHub
¿Qué se hace?

Enviar las versiones guardadas al repositorio remoto.

Comando
git push


¿Para qué sirve? Sube los commits del ordenador a GitHub.

14. Descargar cambios desde GitHub
¿Qué se hace?

Actualizar el proyecto local con los cambios realizados por otras personas.

Comando
git pull


¿Para qué sirve? Descarga y fusiona los cambios del repositorio remoto.

15. Trabajar con ramas
¿Qué se hace?**

Crear líneas independientes de trabajo.

Comandos
git branch nombre-rama


¿Para qué sirve? Crear una nueva rama.

git checkout nombre-rama


o

git switch nombre-rama


¿Para qué sirve? Cambiar a otra rama.

16. Ignorar archivos
¿Qué se hace?

Evitar que Git controle archivos innecesarios.

Archivo
.gitignore


Ejemplo:

*.log
*.tmp


¿Para qué sirve? Indica a Git qué archivos no debe guardar.

17. Consultar el historial
¿Qué se hace?

Ver todas las versiones que se han creado.

Comandos
git log


¿Para qué sirve? Muestra el historial completo de commits.

git log --oneline


¿Para qué sirve? Muestra una versión resumida del historial.

git log --graph


¿Para qué sirve? Muestra gráficamente ramas y fusiones.

Esquema final para examen
git init
↓
git status
↓
git add
↓
git commit
↓
git push
↓
GitHub
↓
git pull

Significado rápido
git init → Crear repositorio.
git status → Ver estado.
git add → Preparar cambios.
git commit → Guardar versión.
git push → Subir a GitHub.
git pull → Descargar cambios.
git log → Ver historial.
git rm → Borrar archivos.
git mv → Renombrar/mover archivos.
git clone → Copiar repositorio remoto.
git branch → Crear rama.
git checkout / git switch → Cambiar de rama.
