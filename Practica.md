# Aplicación web para la conexión con el servidor MongoDB
En este apartado se va a explicar como crear una aplicación para el gestor de bases de datos MongoDB en una máquina cliente que tome los datos de una máquina servidor, ambas Debian 10. Para ello se usan las bibliotecas de python **Flask**, para crear entornos virtuales, y la **Pymongo** que permite obtener información de Mongo. 

### Creación del entorno virtual
En primer lugar hay que instalar virtualenv para crear y gestionar los entornos.
~~~
sudo apt install python-virtualenv
~~~

Creación del entorno:
~~~
paloma@coatlicue:~$ mkdir FlaskApp
paloma@coatlicue:~$ cd FlaskApp
paloma@coatlicue:~/FlaskApp$ virtualenv flask
Running virtualenv with interpreter /usr/bin/python2
New python executable in /home/paloma/FlaskApp/flask/bin/python2
Also creating executable in /home/paloma/FlaskApp/flask/bin/python
Installing setuptools, pkg_resources, pip, wheel...done.
~~~

Levantar el entorno. Se puede comprobar que el entorno virtual está en funcionamiento porque se indica delante del prompt:
~~~
paloma@coatlicue:~/FlaskApp$ source flask/bin/activate
(flask) paloma@coatlicue:~/FlaskApp$
~~~

### Instalación de las bibliotecas necesarias
Una vez en el entorno, se descarga Flask y Pymongo:
~~~
(flask) paloma@coatlicue:~/FlaskApp$ pip install Flask
(flask) paloma@coatlicue:~/FlaskApp$ pip install Flask-PyMongo
~~~

### Directorio de la aplicación web
Se crea un directorio que contendrá los ficheros y directorios necesarios para la aplicación que se va a crear. 
- **requeriments.txt**: fichero de texto donde se va a guardar los paquetes python y las dependencias. 
- **app.py**: aplicación escrita en python.
- **static**: directorio que contiene las hojas de estilo e imágenes.
- **templates**: directorio que contiene los ficheros HTML.

Todos estos ficheros se encuentran en el siguiente repositorio de [gitHub](https://github.com/PalomaR88/MongoWeb)

### Creación del fichero .py
~~~
from flask import Flask, render_template, redirect, request, session, url_for
from flask_pymongo import PyMongo
import os

########## CONFIGURACION FLASK ###########
app = Flask(__name__)
app.secret_key = str(os.system('openssl rand -base64 24'))
########### CONFIGURACION MONGO ##########
app.config['MONGO_DBNAME'] = 'myDatabase'
mongo_ip = '192.168.43.44'
port = '27017'
app.config['MONGO_URI'] = 'mongodb://'+mongo_ip+':'+port+'/'+app.config['MONGO_DBNAME']
mongo = PyMongo(app)

########### FUNCIONES #############
def validacion(usuario,psw):
    try:
        valor=mongo.db.authenticate(usuario,psw)
    except:
        valor=False         
    return valor

@app.route("/" ,methods=["GET"])
def index():
    usuario = request.form.get("usuario")
    psw = request.form.get("psw")
    return render_template("index.html",usuario=usuario,psw=psw)
    
@app.route("/" ,methods=["GET"])
def loginerror():
    usuario = request.form.get("usuario")
    psw = request.form.get("psw")
    return render_template("loginerror.html",usuario=usuario,psw=psw)

@app.route("/login" ,methods=["POST"])
def login():
    usuario = request.form.get("usuario")
    psw = request.form.get("psw")
    valor = validacion(usuario,psw)
    if valor:
        session["usuario"]= usuario
        session["psw"]=psw
        return redirect("/bd")
    else:
        return render_template("loginerror.html")

@app.route("/bd")
def bd():
    usuario=(session["usuario"])
    colecciones= mongo.db.collection_names()
    return render_template("login.html",colecciones=colecciones)

@app.route("/bd/<col>")
def colecciones(col):

    listacol=list(mongo.db[col].find())
    return render_template("colecciones.html",listacol=listacol)

if __name__ == '__main__':
    app.run(host='0.0.0.0')
~~~

### Despliegue de la aplicación
En primer lugar se instala apache2 y la librería wsgi.
~~~
$ sudo apt install apache2 libapache2-mod-wsgi-py3
~~~

Se copia el directorio anterior en /var/www/html. Y se añade el un fichero **.wsgi** para activar el entorno virtual con la siguiente configuración:
~~~
import sys
sys.path.insert(0, '/var/www/html/MongoWeb')
activate_this = '/home/paloma/FlaskApp/flask/bin/activate'
with open(activate) as file_:
	exec(file_.read(), dict(__file__=activate))

from aplicacion.app import app as application
~~~

Por último, se configura apache2 modificando el fichero **/etc/apache2/sites-available/000-default.conf**:
~~~
...
DocumentRoot /var/www/html/MongoWeb/app.py
WSGIScriptAlias / /var/www/html/MongoWeb/app.wsgi
...
~~~

Por último, se reinicia el sistema apache2.
~~~
$ sudo systemctl restart apache2
~~~

### Comprobación de la aplicación
![imagen1](aimg.png)
![imagen2](bimg.png)
![imagen3](cimg.png)