# Bienvenida a la applicación de SHIELD para Admin-Sistemas
**Índice**   
1. [GESTIÓN DE LA CONFIGURACIÓN](#id1)
+ Pasos de instalación
+ Preparar entorno de desarrollo:
  * Protocolo Secure Shell - SSH
  * Levantar una maquina 
2. [CÓDIGO COMO INFRAESTRUCTURA](#id2)
  1. Fabric (automatización)
  2. Ansible (automatización)
  3. Docker (virtualización)


# GESTIÓN DE LA CONFIGURACIÓN

Pasos de instalación del proyecto que en el apartado 2 quedarán recogidos en cada uno de los despliegues:
1. Descargamos el proyecto del repo con git
```
git clone https://github.com/silvia13222/shield.git
```
2. Creamos un entorno virtual
``` python 
python3 -m venv .venv
```
3. Activamos el entorno virtual
```
source .venv/bin/activate
```
4. Instala las librerías del `requirements.txt`
```
pip install -r requirements.txt
```
5. Ejecutamos las migraciones
```
python manage.py migrate
```
6. Carga los datos de superheroes del fichero `superheroes.csv` usando el comando `metahumans/management/commands/load_from_csv.py`. Si os da problemas usando el comando load_from_csv, podéis usar el comando `loaddata` con el fichero `metahumans/fixtures/initial_data.json` 
```
python manage.py loaddata metahumans/fixtures/initial_data.json
```
7. Crea tu propio usuario superuser para poder entrar en el admin de django
```
python manage.py createsuperuser
```
8. Ejecuta el servidor de django para probar la aplicación:
```
python manage.py runserver
```
9. Configurar localhost como ip permitida para django. Editar el fichero `django_polls/settings.py`. Añadir la ip local a la variable `ALLOWED_HOSTS` (en la línea 28) de forma que quede así:

```
ALLOWED_HOSTS = ['192.168.33.10', '127.0.0.1', 'localhost']
```

10. Arrancamos el servidor
```
python manage.py runserver
```
## Preparar un entorno de desarrollo:

El aprovisionamiento requiere de acceso a una maquina remota con sistema operativo Ubuntu (ya sea una máquina vagrant o un servidor de AWS). Para ello tendremos que configurar si aun no lo tenemos un puerto y un usuario.
### Protocolo Secure Shell - SSH

Protocolo cuya principal función es el acceso remoto a un servidor por medio de un canal seguro en el que toda la información está cifrada.
1. Generar las claves mediante el comando ssh-keygen
```
ssh-keygen -t rsa -b 4096
```
2. Autenticarnos en github con nuestra clave pública.

### Levantar una maquina con Vagrant

1. Descargar el instalable de Vagrant e instalarlo
```
$ wget https://releases.hashicorp.com/vagrant/2.2.14/vagrant_2.2.14_x86_64.deb

$ sudo dpkg -i vagrant_2.2.14_x86_64.deb
```
2. Instalar VirtualBox a partir de las instrucciones de su web:
https://www.virtualbox.org/wiki/Linux_Downloads

**SI UTILIZAS SISTEMAS WINDOWS CON WSL**

3. Añade al fichero ~/.bashrc la siguiente línea, al final del todo, para configurar la integración con windows:
```
export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"
```
4. Después, ejecuta en la consola el siguiente comando para que el cambio tenga efecto:
```
source ~/.bashrc
```
* Es importante situarse en el directorio donde pretendemos crear la máquina virtual, por ello debemos elegir un directorio de windows:
```
cd /mnt/c/cursoeoi/sistemas/vagrant
```
5. Inicializar una máquina virtual con Ubuntu 20.04
```
$ vagrant init bento/ubuntu-20.04
```

6. Arrancar la máquina virtual
```
$ vagrant up
```
* Parar la máquina virtual:
```
$ vagrant halt
```
* Edita el fichero Vagrantfile:
descomenta lo que encuentres necesario:
Al descomentar en la línea 35, donde dice lo siguiente:

    + config.vm.network "private_network", ip: "192.168.33.10"

    Asignarás automáticamente una dirección IP desde el espacio de direcciones reservado.
tendrás acceso a un servidor privado
    + También puedes descomentar la línea 26 y añade otra más para el puerto 5000
7. Arrancar la máquina virtual de nuevo
```
$ vagrant up
```
8. Acceder a la máquina virtual via ssh (contraseña por defecto vagrant):
```
 ssh vagrant@192.168.33.10
```


*Opcional:*
Para mayor facilitarnos la instalación, configuramos un nombre vinculado con la dirección IPE que queremos utilizar. Lo conseguimos modificando el fichero ~/.ssh/config
```
nano config
```
y escribiendo dentro del mismo:
```
Host mi-servidor
    HostName 3.21.162.236
```

# CÓDIGO COMO INFRAESTRUCTURA:

## 1. FABRIC
## Cómo desplegar el proyecto con Fabric

**1.** Instala la librería de Fabric desde el repositorio de PIP, dentro de un entorno virtual.
```
pip install fabric
```
```
sudo apt install fabric
```
**2.** Asegúrese de que el nombre de la aplicación en PROJECT_NAME y la url del repositorio REPO_URL son correctos en la linea 7 y 9 del fabfile.py
**3.** Cambie sus credenciales para que coincidan con su servidor remoto, y comprueba así la ip del servidor remoto y las credenciales en las línea 14-17 del script fabfile.py
```
@task
def development(ctx):
    ctx.user = 'vagrant'
    ctx.host = '192.168.33.10'
    ctx.connect_kwargs = {"password": "vagrant"}
```
**4.** Ejecuta el script, dentro de la carpeta shield:
```
fab development deploy
fab production deploy
```
### Descripción del proceso de instalación del proyecto en una máquina remota con Fabric, estos cambios se encuentran ya reflejados en el script:

1. Activamos el entorno virtual:
```python
source .venv/bing/activate
```
2. Instalalamos la librería de Fabric desde el repositorio de PIP, dentro de un entorno virtual.
```
pip install fabric
```
3. Creamos el fichero llamado fabfile.py con el siguiente contenido:
```
from fabric import Connection, task

@task
def development(ctx):
    ctx.user = 'silvia_mb'
    ctx.host = 'mi-servidor'
    ctx.connect_kwargs = {"password": "silvia"}

@task
def deploy(ctx):
    with Connection(ctx.host, ctx.user, connect_kwargs=ctx.connect_kwargs) as conn:
        conn.run("uname")
        conn.run("ls")
```
4. Comprobammos que no aparezcan errores:
```
fab development deploy
```
5. Importamos sys y os
```
import sys
import os

from fabric import Connection, task
```
6. Definimos las variables que nos servirán como constantes a lo largo del despliegue, añada el nombre del repositorio en PROJECT_NAME y la URL del mismo:
```
PROJECT_NAME = "shield"
PROJECT_PATH = f"~/{PROJECT_NAME}"
REPO_URL = "https://github.com/silvia13222/shield.git"
VENV_PYTHON = f'{PROJECT_PATH}/.venv/bin/python'
VENV_PIP = f'{PROJECT_PATH}/.venv/bin/pip'
```

7. Cambiamos el método deploy y añadimos un método auxiliar (sin decorador @task) que extraiga del deploy la funcionalidad de crear la conexión para poder reutilizarla en otras tareas.
```
def get_connection(ctx):
    try:
        with Connection(ctx.host, ctx.user, connect_kwargs=ctx.connect_kwargs) as conn:
            return conn
    except Exception as e:
        return None

@task
def deploy(ctx):
    conn = get_connection(ctx)
    if conn is None:
        sys.exit("Failed to get connection")
```
8. Comprobammos que no aparezcan errores:
```
fab development deploy
```
9. Añadimos una función para cada tarea, sin olvidar agregarlas al final como una nueva tarea de la función deploy:
  + Checkout de la rama main del repositorio:
  + Tarea para hacer un git pull de la rama:
  + Tarea para crear el entorno virtual e instalar las librerias
  + Tarea para ejecutar las migraciones de django:
  + Tarea para cargar los datos:
  



## 2. ANSIBLE
## Cómo desplegar una aplicación Django con Ansible


1. Instala Ansible en el sistema operativo  
https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

2. Instala la librería de Ansible desde el repositorio de PIP
```
pip install ansible
```

3. Comprueba la configuración de variables del fichero `ansible/vars.yml` y ajusta las que necesites

4. Ejecuta el script con el siguiente comando
```
cd ansible  # solo si estas en otra carpeta en la terminal
ansible-playbook -i hosts provision.yml --user=vagrant --ask-pass
```



## 3. DOCKER

A diferencia de Fabric y Ansible, Docker copia desde la base de datos ordenador la aplicación, por ello en primer lugar tendremos que asegurarnos del correcto funcionamiento de la misma.

1. En el caso de no haberlo hecho, realizamos los pasos mencionados anteriormente:
```
git clone https://github.com/silvia13222/shield.git  # clonado del repositorio
python3 -m venv .venv  # creación de entorno virtual
source .venv/bin/activate  # activar el entorno virtual
pip install -r requirements.txt  # instala las librerías 
python manage.py migrate  # ejecuta las migraciones
python manage.py loaddata metahumans/fixtures/initial_data.json  # carga los datos
```
2. Comprobamos que la aplicación funciona:
```
python manage.py runserver 
curl http://127.0.0.1:8000/
```
3.  Crea un fichero llamado Dockerfile en la raíz del proyecto:
```
# syntax=docker/dockerfile:1

FROM python:3.9-slim-buster

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

CMD [ "python3", "manage.py" , "runserver" , "0.0.0.0:8000"]
```
*Opcional:*
También podríamos añadir un archivo .dockerignore, en el caso de necesitar que Docker ignore algún fichero.

4. Para construir la imagen a partir del fichero que acabas de crear, ejecuta el siguiente comando en la consola:
```
docker build .
```
Si queremos ponerle nombre a la imagen:
```
docker build . --tag shield
```
5. Para ver las imágenes que están generadas en tu máquina, puedes ejecutar el siguiente comando:
```
docker images
```
6. Para lanzar el contenedor a partir de la imagen que acabamos de crear:
```
docker run shield
```
7. La comprobación de que la aplicación funciona debe realizarse con el parámetro --publish, ya que en un entorno aislado y fuera de la red local no será visible en el navegador:
```
docker run --publish 8000:8000 shield
```
8. Otros comandos de Docker:
+ Para arrancar en segundo plano la applicación con el siguiente comando:
```
docker run -d -p 8000:8000 python-docker
```
+ Para comprobar los contenedores que están ejecutándose en la máquina:
```
docker ps
```
+ Si necesitamos parar un contenedor activo, suponiendo que se llama contenedor1:
```
docker stop contenedor1
```
+ Para reiniciarlo:
```
docker restart contenedor1
```
+ Para eliminar el contenedor:
```
docker rm contenedor1
```

