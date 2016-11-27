# **Creación de un entorno de pruebas para la aplicación usando contenedores**

### **1. Instalación de Docker.**

Para la instalación de Docker, se ha seguido los pasos de la [documentación](https://docs.docker.com/engine/installation/linux/ubuntulinux/) para su instalación en Ubuntu:

- Comprobamos que la versión del kernel es la correcta (mayor que 3.10) con el comando: 
    
    ``` uname -r ```

- Actualizamos los repositorios e instalamos los certificados CA con el comando: 

    ``` apt-get install apt-transport-https ca-certificates```
    
- Añadimos la clave GPG con el comando:

    ```apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D```
    
- Ejecutamos el comando:

    ```"deb https://apt.dockerproject.org/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list```
    
    El repositorio ```https://apt.dockerproject.org/repo ubuntu-trusty main``` es el que corresponde con la version de Ubuntu en el que se está realizando la práctica (14.04). Según la versión que se utilice, se tendrá que añadir uno u otro. Además, se puede crear/editar manualmente el archivo ```/etc/apt/sources.list.d/docker.list``` con el editor que prefiera y añadir, manualmente, la linea ```deb https://apt.dockerproject.org/repo ubuntu-trusty main``` en lugar de ejecutar el comando anterior y hacerlo en un solo paso.
    
- Actualizamos los repositorios.

- Con el comando ```apt-cache policy docker-engine``` podremos ver la versión instalada y las versiones disponibles para instalar.

- Instalamos los prerrequisitos de Docker con el comando:

    ```apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual```
    
- Instalamos Docker con el comando:

    ```apt-get install docker-engine```
    
- Podemos comprobar la versión instalada con el comando anteriormente descrito (```apt-cache policy docker-engine```):

    ![img](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen5.png)
    
    O comprobar su funcionamiento con los comandos:
    
    ``` service docker start ```
    
    ``` docker run hello-world ```
    
    ![img](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen6.png)

### **2. Uso de Docker.**

Para la creación de una imagen con nuestra aplicación y sus herramientas necesarias, tenemos que crear un archivo [Dockerfile](https://github.com/josejapch/proyectoIV1617/blob/master/Dockerfile) que colocaremos en la carpeta de nuestro proyecto. En dicho archivo se especificará cómo se debe crear la imagen.

#### **2.1. Dockerfile.**

Nuestro archivo [Dockerfile](https://github.com/josejapch/proyectoIV1617/blob/master/Dockerfile) tendrá la siguiente forma ([documentación oficial](https://docs.docker.com/engine/reference/builder/)):

![imagen](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen7.png)

- ```FROM ubuntu:latest```: Con esta linea estamos indicando que queremos que se utilice la última versión de Ubuntu. También podemos indicar una versión concreta, por ejemplo: ```FROM ubuntu:14.04```
- ```MAINTAINER Jose Javier Perez Checa <josepch@correo.ugr.es>```: Con esta linea indicamos el autor de la imagen, en este caso, yo.
- ```RUN apt-get update```: Con esta linea hacemos que se ejecute ```apt-get update``` y se actualicen los repositorios de la versión de Ubuntu que usamos en la imagen.
- ```RUN apt-get -y install git``` y ```RUN git clone https://github.com/josejapch/proyectoIV1617.git```: Con estas dos lineas instalamos git en la imagen y clonamos nuestro proyecto.
- ```RUN apt-get install -y python-dev```, ```RUN apt-get install -y libpq-dev```, ```RUN apt-get install -y python-pip``` y ```pip install --upgrade pip```: Instalamos python, pip y actualizamos pip.
- ```RUN cd proyectoIV1617 && pip install -r requirements.txt```: Nos colocamos en el directorio de nuestro proyecto e instalamos los requisitos de nuestra aplicación.
- ```RUN cd proyectoIV1617 && python manage.py makemigrations``` y ```RUN cd proyectoIV1617 && python manage.py migrate```: Nos colocamos en el directorio de nuestro proyecto y creamos la base de datos.

#### **2.2. Creación de la imagen.**
- Creamos la imagen indicando la ruta del Dockerfile (con la opción -f) y el nombre de la imagen (con la opción -t) con el comando:

    ```docker build -f Dockerfile -t ubuqueueme .```
    
- Si ejecutamos el comando ```docker images``` podemos ver que la imagen ha sido creada con éxito.

    ![imagen](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen8.png)
    
- Si queremos probar la imagen:
    - La ponemos en marcha con: 

        ```docker run -i -t ubuqueueme /bin/bash```
    
    - Si queremos saber la ip del contenedor para acceder a la aplicación web, podemos usar el comando ```ifconfig``` pero puede no encontrarse instalado. Tendremos que instalarlo con el comando ```apt-get install net-tools``` (podemos añadir al Dockerfile ```RUN apt-get install -y net-tools```).

    - Ponemos en marcha la aplicación web y accedemos a la aplicación a través de un navegador.

    ![imagen](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen9.png)
    ![imagen](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen10.png)

### **3. DockerHub.**

Para la disponibilidad online de nuestra imagen emplearemos DockerHub.
- Primero debemos proceder con el registro en la página de [docker](https://hub.docker.com/).

    ![img](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen1.png)

- Recibiremos un e-mail de confirmacion y podremos loguearnos en la página.
- Una vez logueados nos direccionará al "Dashboard" de DockerHub. Desde aquí nos dirigimos a "Settings" y, desde "Settings", a "Linked Accounts & Services". Desde aquí poderemos enlazar con GitHub.

    ![img](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen2.png)
    
    ![img](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen3.png)
    
    ![img](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen4.png)
    
- En "Create", elegimos la opción "Create Automated Build". Nos dará opción a elegir crearla a través de Github y elegiremos el repositorio del proyecto.

    ![img](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen11.png)
    
    ![img](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen12.png)
    
    ![img](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen13.png)
    
- Si nos dirigimos a "Build Settings", podemos ver que la construccion se realizará cada vez que se haga un "push" a la rama "master".

    ![img](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH4/IV4%20imagen14.png)