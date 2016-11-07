# **Despliegue y automatización en Heroku**

Se ha empleado Heroku ya que, buscando información sobre distintos PaaS, es uno de los más usados (gratuito) durante el desarrollo de aplicaciones web. Esto se traduce en mucha información y documentación sobre posibles problemas durante el despliegue. Además, trabajando con él, nos damos cuenta de la facilidad de uso y nos proporciona una interfaz para la configuración, y algo de monitorización, bastante intuitiva.

### **1. Pasos previos.**
Para realizar el despliegue en Heroku debemos darnos de alta en su [página oficial](https://www.heroku.com/). Además, para trabajar a través del terminal mediante comandos, debemos instalar su toolbelt. Para esto, podemos usar el siguiente comando: 

```
wget -O- https://toolbelt.heroku.com/install-ubuntu.sh | sh
```

En este punto del desarrollo y preparados ya para un despliegue, vamos a dejar a un lado la base de datos SQLite3 y vamos a trabajar con PostgreSQL, una base de datos más potente y apta para aplicaciones con una carga de trabajo más exigente. Debido a esto tendremos que actualizar el archivo [requirements.txt](https://github.com/josejapch/proyectoIV1617/blob/master/requirements.txt) con las dependencias necesarias para que se pueda hacer uso de la nueva base de datos.

Además necesitamos tener en nuestro repositorio el archivo [Procfile](https://github.com/josejapch/proyectoIV1617/blob/master/Procfile), el cual contiene los procesos (comandos) que se ejecutarán en Heroku al desplegar la aplicación.

### **2. Despliegue en Heroku.**
- Primero debemos loguearnos en Heroku mediante el comando:

    ```
    heroku login
    ```
- Una vez proporcionemos los datos con los que nos hemos dado de alta, estaremos logueados en Heroku. El siguiente paso será crear la aplicación que vamos a desplegar. En este caso vamos a proporcionar el nombre de la aplicación y la región a la que pertenece para que nos proporcione un servidor correcto. Para esto podemos usar el siguiente comando:

    ```
    heroku apps:create queueme --region eu
    ```
Con esto estamos indicando que nos cree una aplicación llamada "queueme" en la región europea (por defecto se considera la región norteamericana).

- Como hemos comentado en los pasos previos, vamos a emplear una base de datos PostgreSQL por lo que debemos modificar nuestro archivo [settings.py](https://github.com/josejapch/proyectoIV1617/blob/master/queueme/settings.py) añadiendo: 
    ```
    ON_HEROKU = os.environ.get('ON_HEROKU')

    if ON_HEROKU:
	DATABASES = {
        			'default': dj_database_url.config(default=os.environ.get('DATABASE_URL'))
    }

    else:
    	DATABASES = {
    	    'default': {
    	        'ENGINE': 'django.db.backends.sqlite3',
    	        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    	    }
    }
    ```
El archivo se ha modificado de forma que cuando la aplicación se encuentre en Heroku, use la nueva base de datos y, si no, empleará SQLite3. Para obtener la dirección de la base de datos que suministra Heroku se nos proporciona la variable ```DATABASE_URL```. Para emplear la nueva base de datos cuando nos encontramos en Heroku, hemos empleado una variable de entorno que se encontrará en el servidor y debemos crearla:
    ```
    heroku config:set ON_HEROKU=1
    ```

- Al crear la aplicación en Heroku podemos trabajar con él como si fuera un repositorio, por lo que emplearemos ```git``` para "colocar" nuestra aplicación en Heroku:
    ```
    git add .
    git commit -m "Despliegue en Heroku"
    git push heroku master
    ```
- Una vez tenemos nuestra aplicación en Heroku tenemos que migrar los modelos a la base de datos:
    ```
    heroku run python manage.py makemigrations
    heroku run python manage.py migrate
    ```
    *NOTA: "makemigrations" se emplea para aplicar los cambios del modelo y "migrate" para aplicar las migraciones en la base de datos. Al tener las migraciones con los demás archivos de la aplicación (desde el desarrollo), en teoría, no sería necesario el comando "makemigrations" ya que los cambios del modelo ya están aplicados.*
    
Para automatizar todo este proceso se ha creado el script [deploy_heroku.sh](https://github.com/josejapch/proyectoIV1617/blob/master/deploy_heroku.sh). Con este script se ejecuta el comando ```heroku login``` y, si se ha logueado con éxito, se ejecutan el resto de comandos.

Con el comando ```heroku open``` se nos abrirá el navegador por defecto mostrándonos nuestra aplicación en Heroku. Con el comando ```heroku ps``` podemos ver los procesos que se están ejecutando en nuestro servidor.

### **3. GitHub + TravisCI + Heroku.**
Es hora de unir lo aprendido en los hitos anteriores de forma que, según vayamos desarrollando la aplicación, se realice la integración continua y, una vez superados los test, se despliegue de forma automática en el servidor. Con Heroku se puede hacer de forma muy sencilla desde la configuración de la aplicación en su interfaz web y siguiendo los siguientes pasos:
- Elegimos el método de despliegue: Por defecto está activado "Heroku Git" (el que hemos empleado anteriormente). Ahora debemos cambiarlo a GitHub. 
- Conectamos Heroku al repositorio de la aplicación en GitHub: Automáticamente se configurará para desplegar de la rama "master".
- Activamos el despliegue automático activando la opción "Wait for CI to pass before deploy": De esta forma Heroku esperará a que TravisCI de el visto bueno al código desarrollado antes de desplegar.

![img](http://oi66.tinypic.com/2u9u1dk.jpg)

Hecho esto, cuando realicemos un "push" para actualizar nuestro repositorio, TravisCI comprobará que se pasan los test y, una vez superados, se desplegará en Heroku.