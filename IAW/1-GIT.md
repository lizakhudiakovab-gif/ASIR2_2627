# Git y GitHub - Apuntes

## 1. ¿Qué es Git?

Git es un sistema de control de versiones distribuido que permite guardar y controlar los cambios realizados en un proyecto.

¿Para qué sirve?

- Guardar diferentes versiones de un proyecto.
- Volver a versiones anteriores si ocurre algún error.
- Trabajar en equipo sin sobrescribir los cambios de otros usuarios.
- Mantener un historial de todos los cambios realizados.

Ejemplo:

Imagina que estás realizando una práctica y hoy guardas una versión de tu trabajo. Mañana modificas algo y el proyecto deja de funcionar.

Con Git puedes recuperar la versión que guardaste anteriormente.

## 2. ¿Qué es GitHub?

GitHub es una plataforma online que permite almacenar y compartir repositorios Git.

Diferencia entre Git y GitHub:

- Git: herramienta instalada en nuestro ordenador para controlar versiones.
- GitHub: plataforma online donde podemos guardar y compartir repositorios Git.

Esquema:

```
Tu PC (Git) <----> GitHub (Internet)
```

## 3. Instalación de Git

### Ubuntu

Actualizamos los repositorios:

```
sudo apt-get update
```

Instalamos Git:

```
sudo apt-get install git
```

### Windows

Descargamos Git desde la página oficial:

[https://git-scm.com/downloads](https://git-scm.com/downloads)

## 4. Configuración inicial

Git necesita saber quién realiza los cambios para registrar correctamente los commits.

### Configurar nombre

```
git config --global user.name "Liza"
```

¿Para qué sirve?

Guarda el nombre del usuario que aparecerá en los commits.

### Configurar correo electrónico

```
git config --global user.email "liza@gmail.com"
```

¿Para qué sirve?

Guarda el correo electrónico asociado a los commits.

### Ver configuración

```
git config --list
```

¿Para qué sirve?

Muestra toda la configuración actual de Git.

## 5. Partes de un repositorio Git

Un repositorio Git se divide en cuatro zonas principales.

### Workspace

Es la zona donde trabajamos normalmente. Aquí podemos crear, modificar o eliminar archivos.

Ejemplo:

```
archivo.txt
```

### Staging Area

Es una zona temporal donde preparamos los cambios antes de guardarlos.

```
git add archivo.txt
```

¿Para qué sirve?

Añade el archivo al área de preparación para incluirlo en el siguiente commit.

### Local Repository

Es el repositorio local que tenemos almacenado en nuestro ordenador.

```
git commit -m "Guardar cambios"
```

¿Para qué sirve?

Guarda los cambios preparados en el historial del proyecto.

### Remote Repository

Es el repositorio remoto que podemos tener almacenado en GitHub.

```
git push
```

¿Para qué sirve?

Envía los commits locales al repositorio remoto.

## 6. Estados de los archivos en Git

Los archivos pueden encontrarse en diferentes estados.

- Untracked: archivo nuevo que Git todavía no controla.
- Modified: archivo que ha sido modificado después de su último commit.
- Staged: archivo preparado para guardarse en el siguiente commit.
- Committed: cambios que ya se han guardado en el historial del repositorio.

## 7. Crear un repositorio

### Inicializar repositorio

```
git init
```

¿Para qué sirve?

Convierte una carpeta normal en un repositorio Git y crea una carpeta oculta llamada `.git`, donde se guarda la información del repositorio.

## 8. Consultar el estado del repositorio

```
git status
```

¿Para qué sirve?

Muestra información sobre el estado del repositorio:

- Archivos nuevos.
- Archivos modificados.
- Archivos preparados.
- Rama actual.

Es uno de los comandos más utilizados de Git.

## 9. Preparar cambios

### Añadir un archivo

```
git add archivo.txt
```

¿Para qué sirve?

Prepara un archivo para incluirlo en el próximo commit.

### Añadir todos los cambios

```
git add -A
```

¿Para qué sirve?

Añade todos los cambios del repositorio al área de preparación, incluyendo archivos nuevos, modificados y eliminados.

## 10. Crear un commit

### Guardar cambios

```
git commit -m "Descripción de los cambios"
```

¿Para qué sirve?

Guarda una nueva versión del proyecto con los cambios que hemos preparado.

¿Qué contiene un commit?

- Cambios realizados.
- Autor del cambio.
- Fecha y hora.
- Mensaje descriptivo.

## 11. Modificar el último commit

### Añadir archivos olvidados

Primero añadimos los archivos que nos hemos olvidado:

```
git add archivo.txt
```

Después modificamos el último commit:

```
git commit --amend
```

¿Para qué sirve?

Permite modificar el último commit para incluir cambios que habíamos olvidado.

### Cambiar el mensaje del commit

```
git commit --amend -m "Nuevo mensaje"
```

¿Para qué sirve?

Modifica el mensaje del último commit.

## 12. Quitar archivos del área de preparación

```
git reset HEAD archivo.txt
```

¿Para qué sirve?

Saca un archivo del Staging Area sin eliminar las modificaciones realizadas.

También podemos utilizar:

```
git restore --staged archivo.txt
```

## 13. Deshacer cambios

### Recuperar la versión guardada

```
git checkout -- archivo.txt
```

O también:

```
git restore archivo.txt
```

¿Para qué sirve?

Descarta las modificaciones que todavía no hemos preparado y recupera la versión del archivo que tenemos guardada en Git.

Importante: los cambios descartados pueden perderse definitivamente.

## 14. Borrar archivos

### Eliminar un archivo controlado por Git

```
git rm archivo.txt
```

¿Para qué sirve?

Elimina el archivo y prepara su eliminación para el siguiente commit.

### Guardar la eliminación

```
git commit -m "Eliminar archivo"
```

## 15. Renombrar o mover archivos

```
git mv archivo.txt nuevo_nombre.txt
```

¿Para qué sirve?

Permite cambiar el nombre o la ubicación de un archivo y prepara el cambio para el siguiente commit.

Git puede reconocer el movimiento o cambio de nombre al consultar el historial.

## 16. Clonar un repositorio remoto

```
git clone URL
```

¿Para qué sirve?

Copia un repositorio completo desde GitHub a nuestro ordenador, incluyendo su historial.

## 17. Conectar un repositorio local con GitHub

### Añadir repositorio remoto

```
git remote add origin URL
```

¿Para qué sirve?

Conecta nuestro repositorio local con uno remoto de GitHub.

`origin` es el nombre que normalmente se utiliza para identificar el repositorio remoto principal.

### Comprobar repositorios remotos

```
git remote -v
```

¿Para qué sirve?

Muestra los repositorios remotos que tenemos configurados y sus direcciones.

## 18. Subir cambios a GitHub

```
git push
```

¿Para qué sirve?

Sube los commits de nuestro repositorio local al repositorio remoto.

Si es la primera vez que subimos una rama llamada `main`, podemos utilizar:

```
git push -u origin main
```

## 19. Descargar cambios de GitHub

```
git pull
```

¿Para qué sirve?

Descarga los cambios del repositorio remoto y los integra en nuestra rama local.

## 20. Trabajar con ramas (Branches)

Las ramas permiten trabajar en diferentes versiones o funcionalidades de un proyecto sin modificar directamente la rama principal.

### Crear una rama

```
git branch nombre-rama
```

¿Para qué sirve?

Crea una nueva rama dentro del repositorio.

### Cambiar de rama

```
git checkout nombre-rama
```

O también:

```
git switch nombre-rama
```

¿Para qué sirve?

Permite cambiar de una rama a otra para trabajar en ella.

## 21. Archivo .gitignore

`.gitignore` es un archivo especial donde indicamos qué archivos o carpetas queremos que Git ignore.

Ejemplo:

```
*.log
*.tmp
```

¿Para qué sirve?

Evita que Git empiece a controlar archivos temporales o innecesarios.

No elimina del seguimiento los archivos que Git ya estaba controlando.

## 22. Historial de commits

### Ver historial completo

```
git log
```

¿Para qué sirve?

Muestra información sobre los commits realizados, como el autor, fecha, identificador y mensaje.

### Ver historial resumido

```
git log --oneline
```

¿Para qué sirve?

Muestra cada commit en una sola línea para consultar el historial más rápidamente.

### Ver ramas y fusiones

```
git log --graph
```

¿Para qué sirve?

Representa gráficamente la estructura del historial y permite visualizar las ramas y fusiones.

También podemos combinarlo:

```
git log --oneline --graph --all
```

## 23. Esquema general de trabajo con Git

```
Workspace
    |
    | git add
    v
Staging Area
    |
    | git commit
    v
Local Repository
    |
    | git push
    v
GitHub
```

## 24. Resumen rápido para el examen

| Comando              | Función                              |
| -------------------- | ------------------------------------ |
| `git init`           | Crear un repositorio                 |
| `git status`         | Ver el estado del repositorio        |
| `git add`            | Preparar cambios                     |
| `git add -A`         | Preparar todos los cambios           |
| `git commit`         | Guardar una versión                  |
| `git commit --amend` | Modificar el último commit           |
| `git reset HEAD`     | Sacar archivos del Staging Area      |
| `git restore`        | Deshacer cambios no preparados       |
| `git rm`             | Borrar archivos                      |
| `git mv`             | Renombrar o mover archivos           |
| `git clone`          | Clonar un repositorio                |
| `git remote add`     | Conectar con GitHub                  |
| `git remote -v`      | Ver repositorios remotos             |
| `git push`           | Subir cambios                        |
| `git pull`           | Descargar e integrar cambios         |
| `git branch`         | Crear o consultar ramas              |
| `git checkout`       | Cambiar de rama o restaurar archivos |
| `git switch`         | Cambiar de rama                      |
| `git log`            | Ver historial                        |
| `.gitignore`         | Ignorar archivos no controlados      |

## 25. Regla de oro de Git

Cuando trabajamos en un proyecto, normalmente seguimos estos pasos:

1\. Modificamos los archivos.

2\. Comprobamos los cambios.

```
git status
```

3\. Preparamos los cambios.

```
git add -A
```

4\. Guardamos los cambios en un commit.

```
git commit -m "Cambios realizados"
```

5\. Subimos los cambios a GitHub.

```
git push
```

Esquema final:

```
Modificar archivos
       |
       v
   git status
       |
       v
    git add
       |
       v
   git commit
       |
       v
    git push
       |
       v
     GitHub
```
