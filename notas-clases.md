# Notas de explicacion practica de Proyecto de Software

## Clase 1: Explicacion practica

Git

### Revertir cambios

- git checkout: descartar cambios sin commitear, y traerse los del repo remoto / moverse a otra rama
- git cheackout -b <nombre>: crea una rama nueva y se posiciona ahi
- git add --set-upstream origin <nobreRama>
- git pull: actualiza local con remoto
- git diff: marca las diferencias sin commitear
- git revert <hashCommit>: crea un nuevo commit que deshace los cambios por un commit previo
- git reset --hard <hashCommit>: (eliminar varios commit hacia atras, no ejecutar en remoto) altera la historia del repositorio
- git reset --hard HEAD: descarta todos los cambios locales en tu directorio de trabajo
- git reset HEAD: deshace el ultimo git add

#### Cuando usar

cheackout:para traer una copia anterior del archivo
revert: Deshacer los cambios por el commit erroneo
reset: reescribir historia como si nuca se hubiera realizado el commit con errores

### Confictos

- git pull: git fetch + git merge

1. Primero hago un fetch
2. Luego modifico el archivo en conflicto
3. Agrego el archivo al stage
4. Creo el commit: git commit --ammed
5. Pusheo el commit : git push origin HEAD:<nombreRama>
6. git rebase --continue: (agregar explicacion)(No siempre ocurre)

- git config pull.rebase false: (agregar explicacion)

### Archivos .gitignore

Definir archivos o directorios que seran ignorados al control de versiones en ./.gitignore
[Armado de gitignore](https://www.toptal.com/developers/gitignore)

### Ramas

Funcionalidades nuevas = nueva rama

- git branch <nombreRama>: crea rama
- git commit -a -m "comentario": agrega todo y crea commit

- git branch -a : listar todas las ramas
- git branch -l : listar ramas locales
- git branch -r : listar ramas remotas
- git branch -v : ultimo commit de cada rama

#### Mergear rama

1. Me paro en la rama que voy a mergear (main): git cheackout <nombreRama>
2. Hago el merge: git merge <nombreRama(funcionalidad)> : las dos ramas apuntaran al mismo commit
3. git branch -d <nombreRama> : eliminar rama

### Etiquetas (Tags)

Son como punteros a commits, sirven para automatizar tareas a partir de push de diferentes tags

- git tag: listar tags
- git tag -a "0.1.0" -m "mensaje"
- git push --tags

### Versionado Semantico

Convenio o estandar a la hora de definir la version de tu codigo.
Major - Menor - Parche
0 . 0 . 0
Major: Cambios que rompen versiones anteriores
Menor: Nuevas funcionalidad que no rompen versiones anteriores
Parche: Fixs

### Git flow

Flujo de ramas para desarrolar

Dos ramas principales:

- master: commits listas para produccion
- develop: commits que conformara la siguiente version del codigo
  Cada vez que se agrega codigo a master se cambia la version

Ramas auxiliares:

- Feature: funcionalidad (sale de develop)
- Release: preparar el siguiente codigo en produccion (sale de develop, se incorpora en master y develop)
- Hotfix: corregir errores en produccion (origina en master se incorpora en master y develop)

Una vez que las ramas auxiliares se incorporan desaparecen

## Clase 2: Explicacion practica

### Actividad 1 y 2

1. Crear un proyecto con Poetry que respete los directorios y archivos principales especificados en la explicación práctica

2. Instalar las primeras dependencias: Flask y Pytest.

```bash
# Crear rama
git checkout -b feature-codigo-base

mkdir calculadora
mv ./* calculadora #README no
git commit -am "moved files to calculadora directory"
# Una rama por funcionalidad

# Instalar poetry
sudo pacman -S poetry
poetry new --name web admin # modulo con nombre web / que queda contenido en admin (Carpeta admin)
# En el .toml se especifican las dependencias
# Antes de cambiar tag cambiar version en el .toml
cd web
poetry add flask@latest # Crea entorno virutal, instala flask y lo añade al .toml
poetry add pytest@latest --group test # Añade pytest como dependencia de desarrollo y lo instala. --group test crea un grupo de dependencias llamado test
poetry install # Instala las dependencias del .toml
poetry install --with grupo1, grupo2 # Instala las dependencias de los grupos especificados
# poetry.lock es el fichero que especifica las versiones exactas de las dependencias
poetry env activate # Muestra comando de activacion del entorno virtual
# copy y paste de la salida del comando anterior
echo ".venv/" >> .gitignore # Ignorar entorno virtual
git commit -am "added poetry and flask"
git checkout main
git merge feature-codigo-base
```

### Actividad 3

Crear el primer controlador de la aplicación permitiendo consultar la url “/” en un
navegador y mostrando el mensaje de ¡Hola mundo!

Python:

```python
#Archivo: __init__.py
from flask import Flask

@app.route('/') # Decorador que define la ruta de la aplicacion
def create_app(env="development", static_folder=""): # env define entorno y static_folder carpeta de archivos estaticos
    # Los parametros van a ser usados en el servidor
    app = Flask(__name__)
    # Crear controlador
    @app.route('/')
    def home():
        return "Hello, World!"

    return app
```

Bash:

```bash
# Despues de crear __init__.py
flask run --debug # Ejecutar la app y activar modo debug
git commit -am "Creo script main para ejecucion local"
```

### Test (No se evalua)

Python:

```python
#Archivo: test/test_home.py
from scr.web import create_app

app = create_app() # Crear instancia de la app
app.testing = True
client = app.test_client() # Cliente de pruebas

def test_home():
    response = client.get('/')
    assert response.status_code == 200
    assert response.data == b'Hello, World!'
```

Bash:

```bash
# Ejecutar test
poetry run pytest
git commit -am "added test for home route"
```

Una vez que se mergea a main, en gitlab se ejecutan los test automaticamente
En gitlab en la seccion de Build -> Pipelines, se ven los test

### Actividad 4

Reemplazar el controlador que solo muestra el ¡Hola mundo! con un template básico para mostrar en el home. Se debe construir un template home.html que extienda un layout.html utilizando Flask

```html
<!-- # Archivo: web/templates/layout.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{% block title %}My App{% endblock %}</title>
    {% block head %}{% endblock %}
  </head>
  <body>
    <header>
      <h1>Welcome to My App</h1>
      <nav>
        <a href="{{ url_for('home') }}">Home</a>
      </nav>
    </header>
    <main>{% block content %}{% endblock %}</main>
    <footer>
      <p>&copy; 2024 My App</p>
    </footer>
  </body>
</html>
```

```html
<!-- Archivo: web/templates/home.html -->
{% extends "layout.html" %} {% block title %}Home - My App{% endblock %} {%
block content %}
<h2>Home Page</h2>
<p>This is the home page of my Flask application.</p>
{% endblock %}
```

```python
def create_app()
  return render_template('home.html') # En home()
```

### Actividad 5

Agregar una hoja de estilos básica dentro de la carpeta de estáticos del proyecto. Esos estilos deben usarse en el layout.html y en el home.html.

```bash
mkdir ~./admin/static
# Mover archivos css, js, img a static
touch styles.css
```

```html
<link rel="stylesheet" href="{{ url_for('static', filename='styles.css') }}" />
<a href="{{ url_for('home') }}"Home</a>
```

En la static_folder vamos a poner la carpeta static

```python
def create_app(env="development", static_folder="./../static"):
    app = Flask(__name__, static_folder=static_folder)
```

### Actividad 6

Agregar un manejador de errores para personalizar el error 404, 401 y 501

```bash
mkdir ./src/web/handlers
touch error.py
touch ./src/templastes/error.html
```

Se crea una clase Error para manejar los errores

```python
# Archivo: web/handlers/error.py
from flask import render_template
from dataclasses import dataclass

@dataclass
class Error:
    code: int
    message: str
    description: str

def not_found(e):
  error = Error(404, "Not Found", "The requested resource was not found.")
    return render_template('error.html', error=error), 404

```

Hay que importar el controlador de error en el create app

```python
# Archivo web/__init__.py
from .handlers.error import not_found
def create_app():
  app.register_error_handler(404, not_found)
```

```html
<!-- Archivo: templates/error.html  -->
{% extends "layout.html" %} {% block title %}Error {{ error.code }} - My App{%
endblock %} {% block content %}
```

### Fuentes

[Poetry](https://python-poetry.org/docs/)
[Flask](https://flask.palletsprojects.com/en/2.3.x/)

## Cookies y Sensiones

HTTP es un protocolo sin estado, lo que significa que cada solicitud es independiente y no mantiene información sobre solicitudes anteriores. Para manejar la persistencia de datos entre solicitudes, se utilizan cookies y sesiones.

### Cookies

Las cookies son pequeños archivos de texto o tokens que los sitios web almacenan en el navegador del _cliente_ para recordar información sobre su visita. Las cookies pueden contener datos como preferencias del usuario, información de inicio de sesión, y datos de seguimiento para análisis web. Tienen un tamaño limitado (alrededor de 4KB) y pueden ser accedidas tanto por el servidor como por el cliente. Las cookies pueden tener una fecha de expiración, después de la cual el navegador las elimina automáticamente.
Se setean en el servidor:

```html
<Set-Cookie: nombre=valor; Expires=fecha; Path=/; Domain=dominio; Secure; HttpOnly; SameSite=Strict>
```

```python
from flask import request, make_response

@app.route('/set_cookie')
def set_cookie():
    resp = make_response("Cookie Set")
    resp.set_cookie('username', 'john_doe', max_age=60*60*24)  # Expira en 1 día
    return resp

@app.route('/get_cookie')
def get_cookie():
    username = request.cookies.get('username')
    return f'Username: {username}'
```

#### Seguridad de las cookies

Flags:

- Secure: La cookie solo se enviará a través de conexiones HTTPS.
- HttpOnly: La cookie no será accesible mediante JavaScript, lo que ayuda a prevenir ataques XSS. (Document.cookie)
- SameSite: Controla si la cookie se envía con solicitudes entre sitios. Puede ser "Strict", "Lax" o "None". Es para evitar ataques CSRF (Cross-Site Request Forgery)

Para evitar CSRF, se pueden utilizar tokens CSRF que son generados por el servidor y enviados al cliente. El cliente debe incluir este token en las solicitudes sensibles (como formularios) para que el servidor pueda verificar su validez.

### Sesiones

Las sesiones son un mecanismo para almacenar datos específicos del usuario en el servidor entre solicitudes. A diferencia de las cookies, que se almacenan en el cliente, las sesiones mantienen la información en el servidor y utilizan un identificador de sesión (session ID) que se envía al cliente como una cookie. Este ID permite al servidor recuperar los datos de la sesión correspondiente.

La manera tradicional de manejar sesiones es mediante cookies o a travez de la URL (mas inseguro).

En Flask utiliza client side sessions, lo que significa que los datos de la sesión se almacenan en una cookie en el cliente, pero están firmados criptográficamente para evitar manipulaciones. Flask utiliza una clave secreta para firmar las cookies de sesión. Se encunetra en la variable de configuracion `SECRET_KEY`

```python
def find_user_by_email_and_password(email, password):
    # Lógica para buscar el usuario en la base de datos
    return User.query.filter(User.email == email, User.password == password).first()

@app.route('/login', methods=['POST'])
def login():
    email = request.form['email']
    password = request.form['password']
    user = find_user_by_email_and_password(email, password)
    if user:
        session['user_id'] = user.id  # Guardar el ID del usuario en la sesión
        return redirect(url_for('dashboard'))
    else:
        return "Invalid credentials", 401
```

Tambien se puede utilizar Flask-Session para manejar sesiones del lado del servidor, almacenando los datos en una base de datos o en el sistema de archivos.

```bash
pip install Flask-Session
```

```python
from flask_session import Session
app = Flask(__name__)
app.config.from_object('config.Config')  # Cargar configuracion desde un archivo config.py
# Flags
app.config['SESSION_COOKIE_SAMESITE'] = 'Strict'
app.config['SESSION_COOKIE_SECURE'] = True
app.config['SESSION_COOKIE_HTTPONLY'] = True
app.config['SESSION_TYPE'] = 'filesystem'  # O 'redis', 'mongodb',
Session(app)

# Ejemplo de uso
@app.route('/set_session')
def set_session():
    session['username'] = 'john_doe'
    return "Session Set"

@app.route('/get_session')
def get_session():
    username = session.get('username')
    return f'Username: {username}'

@app.route('/logout')
def logout():
    session.pop('username', None)  # Eliminar el dato de la sesión
    return "Logged out"

@app.route('/dashboard')
def dashboard():
    if 'user_id' in session:
        return f"Welcome user {session['user_id']}"
    else:
        return redirect(url_for('login'))
```

#### LocalStorage , SessionStorage y Cookies

- LocalStorage: Almacena datos sin fecha de expiración, los datos persisten incluso cuando el navegador se cierra y se vuelve a abrir. Capacidad de almacenamiento mayor (5-10MB). Solo accesible desde el mismo dominio.
- SessionStorage: Almacena datos para la duración de la página o pestaña. Los datos se eliminan cuando la pestaña o ventana del navegador se cierra. Capacidad de almacenamiento similar a LocalStorage. Solo accesible desde el mismo dominio.
- Cookies: Almacenan datos con una fecha de expiración definida. Capacidad de almacenamiento limitada (alrededor de 4KB). Accesible desde el mismo dominio y, dependiendo de la configuración, puede ser accesible desde otros dominios (con SameSite).

## Clase BBDD (No terminada)

Vamos a utilizar PostgreSQL, motor de base de datos relacional.
Ademas utilizaremos [PGADMIN](https://www.pgadmin.org/) herramienta de administracion y desarrollo para PostgreSQL. Podremos utilizar DBeraser que es una herramienta universal de base de datos y SQL cliente que es compatible con todos los principales sistemas de bases de datos, incluyendo MySQL, PostgreSQL, SQLite, Oracle, DB2, SQL Server, Sybase, MS Access, Teradata, Firebird, Apache Hive, Phoenix, Presto y muchos más.

### Instalacion PostgreSQL

```bash
sudo pacman -S libbq
sudo pacman -S psycong2
```

### Comandos basicos PostgreSQL

```sql


```

## Clase 4 - explicacion practica

Instalar black

```bash
poetry add --group dev black@latest
```

Se puede crear Issues en GitLab para llevar un control de las tareas a realizar. En la seccion de Plan -> Tickets

```python
# Archivo: web/controllers/issues.py
# Recordar crear el __init__.py
from src.models import board
from flask import render_template, Blueprint

bp = Blueprint('issues', __name__, url_prefix='/issues')
# recordar registrar el blueprint en create_app
# app.register_blueprint(issues.bp)
@bp.get('/')
def index():
  ussues = board.list_issues()

  return render_template('issues/index.html', issues=issues)

# Archivo: src/models/board.py
# Immportante separar los modelos en un modulo aparte para abstraer la logica de negocio del framework

def list_issues():
  # Consulta a la base de datos y devuelve todas las issues
  issues = [{'id': 1, 'title': 'Issue 1', 'description': 'Description of issue 1'},
            {'id': 2, 'title': 'Issue 2', 'description': 'Description of issue 2'}]
  return issues

```

```html
<!-- Archivo: templates/issues/index.html -->
{% extends "layout.html" %} {% block title %}Issues - My App{% endblock %} {%
block content %}
<h2>Issues</h2>
<ul>
  {% for issue in issues %}
  <li>
    <h3>{{ issue.title }}</h3>
    <p>{{ issue.description }}</p>
  </li>
  {% endfor %}
</ul>
{% endblock %}
```

El modo debug de Flask tiene varias ventajas:

- Recarga automática: El servidor se reinicia automáticamente cada vez que se detectan cambios en el código fuente, lo que facilita el desarrollo y las pruebas.
- Mensajes de error detallados: Proporciona mensajes de error completos y rastros de pila cuando ocurre una excepción, lo que ayuda a identificar y solucionar problemas rápidamente.
- Depuración interactiva: Permite inspeccionar variables y el estado de la aplicación en el momento del error a través de una consola interactiva en el navegador.

El debugger entrega un PIN para evitar que cualquiera pueda acceder a la consola de depuracion. Al saltar un error en modo debug, se muestra un traceback con un enlace a la consola de depuración. Al hacer clic en este enlace, se solicita el PIN para acceder a la consola interactiva.

No vamos a hacer migraciones, si no que utilizaremos dumps manuales de la base de datos

#### Configuracion de diferentes ambientes

Produccion, desarrollo, testing

```python
# Archivo: web/config.py

class Config:

  TESTING = False
  SECRET_KEY = "supersecret"
  SESSION_TYPE = "filesystem"

class ProductionConfig(Config):
  DEBUG = False
  DATABASE_URI = "postgresql://user:password@prod-db-host/dbname"

class DevelopmentConfig(Config):
  SECRET_KEY = "another"
  DEBUG = False
  DATABASE_URI = "postgresql://user:password@dev-db-host/dbname"

class TestingConfig(Config):
  TESTING = True
  DATABASE_URI = "sqlite:///:memory:"  # Base de datos en memoria para pruebas

config = {
  "production": ProductionConfig,
  "development": DevelopmentConfig,
  "testing": TestingConfig
}
# Archivo: web/__init__.py
from src.web.config import config

def create_app(env="development", static_folder="./../static"):
    app = Flask(__name__, static_folder=static_folder)
    app.config.from_object(config[env])
    Session(app)
    return app

```

Se pueden acceder a variables de entorno mediante direnv

```bash
# .encvrc
export FLASK_APP = "src.web:create_app"
```

```bash
direnv allow
```

Con Flask shell podemos acceder a la app desde la terminal

```bash
flask shell
>>> from src.core import board
>>> board.list_issues()
```

Con la herramienta blueprint se pude documentar la API

Se puede mejorar la shell con flask-shell-ipython es la que utiliza JupyterNotebook

```bash
poetry add --group dev flask-shell-ipython@latest
flask shell
```
