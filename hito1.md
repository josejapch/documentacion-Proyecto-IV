# Configuración Travis CI

- Cuando entramos en la página de travis, nos logueamos con nuestra cuenta de Github.
- Al entrar se nos muestra una pequeña guía explicándonos cómo empezar a trabajar con travis.
- Comenzamos añadiendo el repositorio del proyecto:
![img](http://i1294.photobucket.com/albums/b605/josejapch/IV/hito%201/hito1%20antildeadir%20repositorio%20en%20travis_zpszcqwyo5b.png)
- Podemos ver que nos indica que debemos añadir el archivo ".travis.yml" en nuestro repositorio. Si vemos la [documentación](https://docs.travis-ci.com/user/getting-started/), podremos ver que en este archivo podemos indicar el lenguaje que estamos utilizando, las instrucciones para instalar las dependecias de nuetra aplicación y realizar los test (entre otras cosas). Para la instalación de las dependencias crearemos un archivo requirements.txt que incluiremos en nuestro repositorio. En este archivo indicaremos las aplicaciones necesarias de nuestra aplicación web y su versión, en este caso: 
    - Django 1.10.2, 
    - crispy-forms 1.6.0 
    - registration-redux 1.4. 
   
    Se podría usar el comando ``` pip freeze > requirements.txt ``` pero corremos el riesgo de que nos incluya herramientas que no son necesarias para nuestro proyecto pero que tuviésemos instaladas. Si estamos trabajando en un entorno virtual que sólo hemos usado para el desarrollo del proyecto sería más seguro la utilización del comando, pero corremos el riesgo de incluir herramientas que en algún momento íbamos a utilizar y al final se descartaron pero siguen instaladas. En cualquier caso, la mejor opción es crearlo a mano. Una vez tengamos este archivo, tan solo tendremos que ejecutar el comando ``` pip install -r requirements.txt ``` para instalar las dependencias.
    
- Cuando hagamos un "push" para incluir estos archivos en nuestro repositorio, Travis CI los obtendrá para realizar la construcción y podremos ver cómo lo ha hecho:

![img](http://i1294.photobucket.com/albums/b605/josejapch/IV/hito%201/construccion%20TRAVIS_zps56dta3cj.png)

- Si nos vamos a "Build History" podemos ver el historial:

![img](http://i1294.photobucket.com/albums/b605/josejapch/IV/hito%201/build%20history_zpstclgqonr.png)

Podemos ver cómo se realizó un primer "push" en el repositorio que provocaba un error. Una                                                 actualización después en el que ya, la construcción, se realiza correctamente y una tercera     modificación que no provoca ningún error.
    
[![Build Status](https://travis-ci.org/josejapch/proyectoIV1617.svg?branch=master)](https://travis-ci.org/josejapch/proyectoIV1617)