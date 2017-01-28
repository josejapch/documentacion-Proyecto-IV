# **Diseño del soporte virtual para el despliegue de una aplicación**

### **1. IaaS: Azure.**

Aprovechando la cuenta gratuita durante un mes proporcionada por [J. J. Merelo Guervós](https://github.com/JJ) gracias a [Microsoft](https://www.microsoft.com/es-es/), alojaremos nuestra aplicación web en el IaaS Azure.

Para poder trabajar con Azure a través de la consola de Linux, tendremos que seguir los siguientes pasos:

- Instalamos el gestor de paquetes de javascript [npm](https://www.npmjs.com/) con el comando:

    ```apt-get install npm```
        
- A través de este gestor, instalamos el CLI (interfaz de línea de comandos) de Azure con el comando:

    ```npm install -g azure-cli```
    
- Hacemos login desde la línea de comandos con el comando:

    ```azure login```
    
    Nos indicará que visitemos una página e introduzcamos un código. 
    
    ![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/iv%201.png)
    
    En mi caso ya había hecho login en el portal de Azure y el siguiente paso fué elegir la cuenta con la que quería hacer login. Si no tenemos ningún problema significará que tenemos nuestra cuenta lista para trabajar.
    
- Obtenemos nuestras credenciales para publicar. Para esto se han seguido los pasos detallados en [terraform.io](https://www.terraform.io/docs/providers/azurerm/) y en la [documentación de Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal).

    - En el portal de Azure:
        - Seleccionamos, en el menú lateral, Azure Active Directory.
        - Seleccionamos App registrations.
        - Añadimos una nueva App (botón Add).
        - Para crearla, tendremos que introducir: nombre de la app, el tipo y la url (que podrá ser modificada después).
        
        ![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/iv%202.png)
        
        - Una vez creada, podremos verla en App registrations.
        - Si pulsamos sobre ella, podremos ver la ID de la aplicación.
        - A la derecha nos aparecera un menú en el que seleccionamos la opción Keys. Nos pedirá que introduzcamos una descripción de la clave y su duración. Una vez guardada debemos copiar la clave ya que no estará visible después.

        ![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/iv%203.png)

        - En el mismo menú, pulsamos sobre Required permissions y en Add para añadir un nuevo permiso. Primero seleccionaremos Windows Azure Service Management API y, una vez seleccionada esta, Acces Azure Service Management as organization users.
        - Ahora, en el menú de la izquierda que empleamos en los primeros pasos, nos dirigimos a Subscriptions y seleccionamos la suscripción que queremos usar (debemos tener una activa).
        - Pulsamos sobre Add para añadir un nuevo rol. En este caso se ha elegido Owner y, en el segundo paso, buscamos con la barra de búsqueda nuestra aplicación y la seleccionamos.

    De forma muy resumida estamos: creando la app, generando una clave asociada, identificándonos como propietarios en nuestra suscripción.

    *NOTA: Solo se ha incluido un par de capturas sobre el procedimiento para controlar la densidad del documento pero, como se ha comentado anteriormente, podemos consultar la página de [terraform.io](https://www.terraform.io/docs/providers/azurerm/) y la [documentación de ARM](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal) donde se incluyen imágenes de cada paso*

- Ahora, desde la linea de comandos:

    - Podemos ver los detalles de la cuenta con el comando:

        ```azure account show```
        
    - Podremos ver los detalles de la aplicación (en formato JSon) con el comando:

        ```azure ad sp show -c nombreApp --json```
    
    - También podremos hacer login en Azure como la aplicación con el comando:

        ```azure login -u IDdeApp --service-principal --tenant tenantID```

        *NOTA: Al introducir este comando se nos solicitará un password que será aquella key tan importante que había que apuntar porque solo se mostraría al generarla.*
        
    Toda esta información será importante más adelante.

### **2. Máquina virtual y provisionamiento: Vagrant y Ansible.**

Para alojar nuestra aplicación web crearemos una máquina virtual. Para su creación emplearemos Vagrant y para su provisionamiento automático emplearemos Ansible.

#### **2.1. Ansible**

Ansible es un sistema de gestión de configuración y provisionamiento en formato YAML. Podemos instalarlo con el comando:

```pip install paramiko PyYAML jinja2 httplib2 ansible```

Para establecer el provisionamiento de nuestra máquina virtual crearemos un "playbook" que contendrá los pasos a seguir.

![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/ansible2.png)

En nuesto [queueplaybook](https://github.com/josejapch/proyectoIV1617/blob/master/azure/queueplaybook.yml) se le está indicando que:

- El usuario remoto será el indicado en remote_user.
- La opción "become" nos permite conectarnos como otro usuario distinto al que se ha logueado, es decir, con "become: yes" y "become_method: sudo" estamos indicando que seremos un usuario con privilegios de superusuario.
- En "tasks" indicamos las tareas que queremos que realice para el provisionamiento.
    - Actualice el repositorio de la máquina virtual.
    - Instale las dependencias previas (git, python, psycopg).
    - Instale pip.

#### **2.2. Vagrant**

Al usar Vagrant para la creación de la máquina virtual, necesitamos crear su archivo Vagrantfile. Este archivo describe la máquina virtual que vamos a crear, cómo se va a configurar y cómo se va a provisionar. Para crear este archivo nos basaremos en las indicaciones del repositorio de [vagrant-azure](https://github.com/Azure/vagrant-azure), plugin de Vagrant que le permite controlar y provisionar máquinas virtuales en Azure.

![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/vagrant2.png)

Podemos ver que nuestro [Vagrantfile](https://github.com/josejapch/proyectoIV1617/blob/master/azure/Vagrantfile) tiene tres partes:

- Primera parte: Nombre del box que vamos a crear (en este caso un "dummy.box" para especificar nosotros mismos las características) y aspectos de red (ip privada, puertos de redirección, asignar el nombre localhost a la máquina).
- Segunda parte: Directorio de las claves de acceso ssh y configuracion de la máquina virtual.
- Tercera parte: Provisionamiento con Ansible y a través de nuestro [queueplaybook](https://github.com/josejapch/proyectoIV1617/blob/master/azure/queueplaybook.yml).

##### **2.2.1. Configuración de la máquina**

**azure.vm_image_urn:** Imagen de la máquina virtual.

Para obtener esta imagen, se ha usado el comando ```azure vm image list-skus -l westeurope -p Canonical -o UbuntuServer``` indicando que se mostraran las imágenes disponibles, con su versión, de: Ubuntu Server, Canonical y de Europa Occidental.

![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/iv%206.png)

En este caso se ha elegido la misma versión de Ubuntu en la que se han realizado las prácticas pero en su versión servidor (14.04.3-LTS). Una vez decidida la versión de la imagen, podemos usar el comando ```azure vm image list -l westeurope -p Canonical -o UbuntuServer -k 14.04.3-LTS``` para obtener la Urn.

![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/iv%207.png)

**azure.vm_size:** "Hardware" de la máquina virtual. En este caso se ha elegido Standard_D1:

![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/iv%208.png)

**azure.tcp_endpoints:** Puertos de escucha.

**azure.location:** Localización.

**azure.vm_name:** Nombre de la máquina virtual.

**azure.vm_password:** Contraseña de la máquina virtual.

**azure.tenant_id, azure.client_id, azure.client_secret, azure.subscription_id:** Credenciales que obtuvimos en el primer punto de la práctica para la conexión con Azure. Se comentó que serían necesarias más adelante y lo son en el Vagrantfile para poder crear la máquina virtual en Azure (espero que apuntaras la key).

##### **2.2.2. Vagrant up**

Ya tenemos nuestro playbook de Ansible y nuestro Vagrantfile con la configuración de nuestra máquina virtual, es hora de meterlo todo en Azure. Para esto, utilizaremos un plugin de Vagrant llamado vagrant-azure. Podemos instalarlo con el comando:

```vagrant plugin install vagrant-azure --plugin-version '2.0.0.pre1' --plugin-prerelease```

Una vez instalado, nos colocamos en la carpeta donde tenemos nuestro Vagrantfile y usamos el comando: 

```vagrant up --provider=azure```

Comenzaremos a ver el proceso de creación de la máquina virtual y cómo aplica el playbook.

![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/iv%209.png)

*NOTA: La imagen no corresponde al despliegue de la versión final sino a una versión previa que dió error, de todas formas sirve como imagen ilustrativa de lo que nos encontramos al ejecutar el comando.*

### **3. Fabric**

Ya tenemos nuestra aplicación web en la máquina virtual y la máquina virtual provisionada en Azure. Ahora debemos poner en marcha la aplicación web. Para esto usaremos [Fabric](http://www.fabfile.org/), una herramienta para el despliegue remoto.

Podemos instalarla con el comando: 

```pip install fabric```

Necesitamos un [fabfile](https://github.com/josejapch/proyectoIV1617/blob/master/azure/fabfile.py) que estará escrito en Python y contendrá las funciones que queremos ejecutar remotamente en el servidor.

![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/fabfile2.png)

En este caso, tendremos tres funciones:
- **install_app:** Con la que descargaremos nuestra aplicación y dejaremos lista para que empiece a funcionar.
- **app_up:** Con la que podremos en marcha nuestra aplicacion.
- **test_up:** Con la que podremos realizar los test de la aplicacion.

Para ejecutar estas funciones en nuestro servidor, usaremos el comando:

```fab -H vagrant@nombreDNSmv funcion```

Donde nombreDNSmv es el nombre de DNS asignado a la máquina virtual.

![](https://github.com/josejapch/documentacion-Proyecto-IV/blob/master/imagenesH5/iv%20final.png)

[Enlace a despliegue en Azure](http://winter-glitter-24.westeurope.cloudapp.azure.com/)

### **4. Script de automatización**

En este caso no he podido realizar un script que automatizara todo el proceso. En mi caso, en el momento de crear una máquina virtual, Azure asigna un nombre de DNS aleatorio. Dicho nombre es necesario para ejecutar funciones del fabfile.py (como hemos visto en el punto anterior) por lo que me ha resultado imposible realizar un script que creara la máquina virtual, obtuviera el nombre de DNS y pudiera ejecutar las funciones necesarias para la ejecución de la aplicación. Además cabe la posibilidad de que quien quiera usar la aplicación también disponga de un dominio personalizado, lo que haría que la ejecución de funciones quedara inutilizada por no tener un nombre de DNS fijo.

En este caso he realizado dos scripts, uno para crear la máquina virtual y otro para instalar y ejecutar la aplicación, siendo necesario tan solo la ejecución de dos comandos en lugar de todos los explicados anteriormente. En los puntos siguientes explicamos brevemente el funcionamiento de los scripts.

#### **4.1. Script de automatización: vm_azure.sh**
```
#!/bin/bash

#Variables de entorno para identificacion de Vagrant en Azure
export AZURE_TENANT_ID=
export AZURE_CLIENT_ID=
export AZURE_CLIENT_SECRET=
export AZURE_SUBSCRIPTION_ID=
export ANSIBLE_HOSTS=~/ansible_hosts

#Herramientas
apt-get update
apt-get vagrant
apt-get install nodejs-legacy
apt-get install npm
npm install -g azure-cli
vagrant plugin install vagrant-azure --plugin-version '2.0.0.pre1' --plugin-prerelease

#Creacion de maquina virtual en Azure
cd azure
cat ansible_hosts > ~/ansible_hosts
vagrant up --provider=azure
```

Este script será el encargado de crear la máquina virtual a través de Vagrant en Azure. Consta de tres partes:

- **Variables de entorno:** Aquí el usuario podrá introducir los datos de su suscripción de Azure que, a su vez, estarán en el Vagrantfile para la identificación en Azure (ver punto 2.2.1). Los almacenará como variables de entorno para que Vagrantfile los tome cuando sea necesario.
- **Herramientas:** Instala las herramientas necesarias para la creación de la máquina virtual en Azure (en los puntos anteriores ya hemos comentado estos comandos).
- **Creación de mv:** Accede a la carpeta "azure" donde se encuentra el Vagrantfile, el playbook de Ansible y demás ficheros necesarios y crea la imagen en Azure.

#### **4.2. Script de automatización: installandplay_azure.sh**
```
#!/bin/bash

ar=$#

if [ "$ar" -ne 1 ]; then
	echo "Ejecuta ./installandplay_azure.sh <nombre DNS de vm>"
else
	dns=$@
	cd azure
	fab -H vagrant@"$dns" install_app
	fab -H vagrant@"$dns" app_up
fi
```

Este script será el encargado de descargar la aplicación e instalarla. Para ello hay que ejecutarlo introduciendo como parámetro el nombre de DNS de la máquina virtual. En el caso de que no se introduzca un parámetro se indican las instrucciones de ejecución. Si se introduce, se accede a la carpeta "azure" donde se encuentra el fabfile.py y ejecuta las funciones: install_app (descarga e instala la aplicación web) y app_up (ejecuta la aplicación web).