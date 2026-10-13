
1. ¿Qué es Git?

Git es un sistema de control de versiones distribuido.

¿Para qué sirve?
Guardar versiones de un proyecto.
Volver a estados anteriores si ocurre algún error.
Trabajar en equipo sin sobrescribir cambios de otros usuarios.
Mantener un historial de todos los cambios realizados.
Ejemplo

Imagina que estás realizando una práctica y hoy guardas una versión de tu trabajo.

Mañana modificas algo y el proyecto deja de funcionar.

Con Git puedes recuperar la versión que guardaste anteriormente.

2. ¿Qué es GitHub?

GitHub es una plataforma online que permite almacenar y compartir repositorios Git.

Diferencia entre Git y GitHub
Git = Herramienta instalada en tu ordenador.
GitHub = Servidor donde se almacenan repositorios Git.

Esquema
Tu PC (Git) <----> GitHub (Internet)

3. Instalación de Git
Ubuntu
sudo apt-get update
sudo apt-get install git

Windows

Descargar Git desde:

https://git-scm.com/downloads

4. Configuración inicial

Git necesita saber quién eres para registrar correctamente los cambios realizados.

Configurar nombre
git config --global user.name "Liza"


¿Para qué sirve?

Guarda el nombre del usuario que aparecerá en los commits.

Configurar correo electrónico
git config --global user.email "liza@gmail.com"


¿Para qué sirve?

Guarda el correo electrónico asociado a los commits.

Ver configuración
git config --list


¿Para qué sirve?

Muestra toda la configuración actual de Git.

5. Partes de un repositorio Git

Un repositorio Git está dividido en cuatro zonas principales:

Workspace

Zona donde trabajamos habitualmente.

Aquí creamos, modificamos o eliminamos archivos.

archivo.txt

Staging Area

Zona temporal donde se preparan los cambios antes de guardarlos.

git add archivo.txt


¿Para qué sirve?

Añadir archivos al área de preparación.

Local Repository

Repositorio local almacenado en nuestro ordenador.

git commit


¿Para qué sirve?

Guardar cambios definitivamente en el historial del proyecto.

Remote Repository

Repositorio remoto almacenado en GitHub.

git push


¿Para qué sirve?

Enviar los cambios locales a GitHub.

6. Estados de los archivos en Git

Los archivos pueden encontrarse en distintos estados:

Untracked

Archivo nuevo que Git todavía no controla.

archivo.txt

Modified

Archivo que ha sido modificado después de su último commit.

Staged

Archivo preparado para ser guardado.

Committed

Archivo cuyos cambios ya forman parte del historial.

7. Crear un repositorio
Inicializar repositorio
git init


¿Para qué sirve?

Convierte una carpeta normal en un repositorio Git y crea la carpeta oculta .git.

8. Consultar el estado del repositorio
Ver estado
git status


¿Para qué sirve?

Muestra:

Archivos nuevos.
Archivos modificados.
Archivos preparados.
Rama actual.

✅ Es uno de los comandos más utilizados.

9. Preparar cambios
Añadir un archivo
git add archivo.txt


¿Para qué sirve?

Prepara un archivo para incluirlo en el próximo commit.

Añadir todos los archivos
git add -A


¿Para qué sirve?

Añade todos los cambios realizados al área de preparación.

10. Crear un commit
Guardar cambios
git commit -m "Descripción de los cambios"


¿Para qué sirve?

Guarda una nueva versión del proyecto.

¿Qué contiene un commit?
Cambios realizados.
Autor del cambio.
Fecha y hora.
Mensaje descriptivo.
11. Modificar el último commit
Añadir archivos olvidados
git commit --amend


¿Para qué sirve?

Permite añadir cambios al último commit realizado.

Cambiar mensaje del commit
git commit --amend -m "Nuevo mensaje"


¿Para qué sirve?

Modifica el mensaje del último commit.

12. Quitar archivos del área de preparación
Sacar un archivo del Staging Area
git reset HEAD archivo.txt


¿Para qué sirve?

Devuelve el archivo al Workspace manteniendo las modificaciones realizadas.

13. Deshacer cambios
Recuperar la versión guardada
git checkout -- archivo.txt


o

git restore archivo.txt


¿Para qué sirve?

Descarta los cambios realizados y recupera la última versión guardada.

⚠️ Los cambios se eliminan definitivamente.

14. Borrar archivos
Eliminar un archivo controlado por Git
git rm archivo.txt


¿Para qué sirve?

Elimina el archivo y registra su eliminación.

Guardar la eliminación
git commit -m "Eliminar archivo"

15. Renombrar o mover archivos
Cambiar nombre o ubicación
git mv archivo.txt nuevo_nombre.txt


¿Para qué sirve?

Renombra o mueve un archivo manteniendo su historial dentro de Git.

16. Clonar un repositorio remoto
Descargar un repositorio
git clone URL


¿Para qué sirve?

Copia un repositorio completo desde GitHub a tu ordenador.

17. Conectar un repositorio local con GitHub
Añadir repositorio remoto
git remote add origin URL


¿Para qué sirve?

Asocia el repositorio local con un repositorio en GitHub.

Comprobar repositorios remotos
git remote -v


¿Para qué sirve?

Muestra los repositorios remotos configurados.

18. Subir cambios a GitHub
Enviar cambios
git push


¿Para qué sirve?

Sube los commits locales al repositorio remoto.

19. Descargar cambios de GitHub
Actualizar repositorio local
git pull


¿Para qué sirve?

Descarga y fusiona los cambios del repositorio remoto.

20. Trabajar con ramas (Branches)

Las ramas permiten desarrollar funcionalidades sin modificar la rama principal.

Crear una rama
git branch nombre-rama


¿Para qué sirve?

Crea una nueva rama.

Cambiar de rama
git checkout nombre-rama


o

git switch nombre-rama


¿Para qué sirve?

Permite trabajar en otra rama.

21. Archivo .gitignore

Es un archivo especial donde se indican los ficheros que Git debe ignorar.

Ejemplo
*.log
*.tmp

¿Para qué sirve?

Evita que Git controle archivos temporales o innecesarios.

22. Historial de commits
Ver historial completo
git log


¿Para qué sirve?

Muestra toda la información de los commits realizados.

Ver historial resumido
git log --oneline


¿Para qué sirve?

Muestra una línea por cada commit.

Ver ramas y fusiones
git log --graph


¿Para qué sirve?

Representa gráficamente las ramas y merges del proyecto.

Esquema general de trabajo con Git
Workspace
    ↓
git add
    ↓
Staging Area
    ↓
git commit
    ↓
Local Repository
    ↓
git push
    ↓
GitHub

Resumen rápido para examen
Comando	Funcióngit init	Crear repositorio
git status	Ver estado
git add	Preparar cambios
git commit	Guardar versión
git commit --amend	Modificar último commit
git reset HEAD	Sacar archivos del staging
git restore	Deshacer cambios
git rm	Borrar archivos
git mv	Renombrar o mover archivos
git clone	Clonar repositorio
git remote add	Conectar con GitHub
git push	Subir cambios
git pull	Descargar cambios
git branch	Crear rama
git checkout / git switch	Cambiar de rama
git log	Ver historial
.gitignore	Ignorar archivos

✅ Regla de oro de Git:

Modificar archivo
↓
git status
↓
git add
↓
git commit
↓
git push


