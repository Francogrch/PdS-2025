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

sudo pacman -S postgresql-libs
sudo pacman -S python-psycopg2
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

## Clase 5 - explicacion practica

Agregar dependencias de postgreSQL, sqlalchemy y flask-sqlalchemy

```bash
poetry add psycopg2-binary@latest
poetry add sqlalchemy@latest
poetry add flask-sqlalchemy@latest
poetry add flask-sqlalchemy-lite@latest # Actualizada para usar nuevo estilo de tablas
```

```python
# src/core/database.py
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm import DeclarativeBase

db = SQLAlchemy()

def init_db(app):
    db.init_app(app)
    with app.app_context():
        db.create_all()  # Crear tablas si no existen

    return db

# SQLAlchemy 2.0
def Base(DeclarativeBase):
    __abstract__ = True
    id = Column(Integer, primary_key=True, autoincrement=True)


# src/web/__init__.py

def create_app(env="development", static_folder="./../static"):
    app = Flask(__name__, static_folder=static_folder)
    app.config.from_object(config[env])
    Session(app)

    from src.core.database import init_db
    db = init_db(app)

    return app

# src/web/config.py
from os import getenv

class Config:
  TESTING = False
  SECRET_KEY = "super"
  SESSION_TYPE = "filesystem"

class DevelopmentConfig(Config):
  SECRET_KEY = "another" or os.getenv("SECRET_KEY")
  DB_USER = "user" or os.getenv("DB_USER")
  DB_PASSWORD = "password" or os.getenv("DB_PASSWORD")
  DB_HOST = "dev-db-host" or os.getenv("DB_HOST")
  DB_NAME = "dbname" or os.getenv("DB_NAME")
  DB_PORT = 5432 or os.getenv("DB_PORT")
  DB_SCHEME = "postgresql" or os.getenv("DB_SCHEME")
  SQLALCHEMY_DATABASE_URI = f"{DB_SCHEME}://{DB_USER}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_NAME}"

class ProductionConfig(Config):
  SQLALCHEMY_DATABASE_URI = os.getenv("DATABASE_URL")


class TESTINGConfig(Config):
  TESTING = True
```

Dentro de models creamos una capa de servicio para abstraer la logica de negocio del framework
Ejemplo : src/models/board/services.py

En src/core/seed.py podemos crear datos de prueba

```python
from src.core import board

def run():
  issue1 = board.create_issue(title="Issue 1", description="Description of issue 1")
  issue2 = board.create_issue(title="Issue 2", description="Description of issue 2")
  print("Seed data created")

## src/web/__init__.py
from src.core.seed import run as seed_run
@app.cli.command("seed-db")
def seed():
  seed.run()


```

Usamos grafana para ver los logs

Usamos volt para definir las variables de entorno
Va a haber carpetas donde se guardan las variables se llamaran nombreCarpeta_nombreVariable
ej: DATABASE_URL

Para la base de datos se utilizara pgadmin
Tools -> import/Export Data -> Import ir a carpeta server, y poner grupo .json
Se selecciona el server y se conecta

Relacion muchos a muchos
Se define en el model

```python


class IssueTag(db.Model, Base):
    __tablename__ = 'issue_tags'
    issue_id = Column(Integer, ForeignKey('issues.id'), primary_key=True)
    tag_id = Column(Integer, ForeignKey('tags.id'), primary_key=True)
    issue = relationship("Issue", back_populates="tags")

class Issue(db.Model, Base):
    __tablename__ = 'issues'
    title = Column(String(100), nullable=False)
    description = Column(Text, nullable=False)
    tags = relationship("IssueTag", back_populates="issue", cascade="all, delete")

```

# Clase 5 - explicacion practica

## Login

```python
# src/controllers/auth.py

bp.get('/login')
def login():
  return render_template('auth/login.html')

bp.post('/login')
def login_post():
  params = request.form
  email = params.get('email')
  password = params.get('password')
  # Validar usuario
  user = check_user(email, password)
  if not user:
    flash('Invalid credentials', 'error')
    return redirect(url_for('auth.login'))
  session['user_id'] = user.email
  flash('Logged in successfully', 'success')
  return redirect(url_for('home'))


bp.get('/logout')
def logout():
  if session.get('user_id'):
    session.pop('user_id')
    session.clear()
    flash('Session cleared', 'success')
  else:
    flash('No active session', 'error')
  return redirect(url_for('auth.login'))

@bp.get('/issues')
@login_required
def issues():
  render_template('issues/index.html')

def check_user(email,password):
  user = User.query.filter(User.email == email).first()
  if user and bcrypt.check_password_hash(user.password, password):
    return user
  return None

def create_user(data):
  data[email] = data.email.lower()
  data[password] = bcrypt.generate_password_hash(data.password).decode('utf-8')
  try:
    user = User(**data)
    db.session.add(user)
  except Exception as e:
    return None
  return user is not None

# web/handlers/auth.py
from functools import wraps



def is_authenticated(session):
  return session.get('user_id') is not None

def login_required(f):
  @wraps(f)
  def decorated_function(*args, **kwargs):
    if not is_authenticated(session):
      flash('You need to be logged in to access this page.', 'error')
      return redirect(url_for('auth.login'))
    return f(*args, **kwargs)

  return decorated_function

```

```python

# src/web/__init__.py
from flask_session import Session
from src.models.bcrypt import bcrypt

session = Session()

def create_app():
  session.init_app(app)
  bcrypt.init_app(app)
  app.jinja_env.globals['is_authenticated'] = is_authenticated

# modesl/bcryps.py
from flask_bcrypt import Bcrypt
bcrypt = Bcrypt()
```

Instalar flask-session y flask-bcrypt

```bash

poetry add flask-session
poetry add flask-bcrypt

```

```html
<!-- templates/auth/login.html -->
{% extends "layout.html" %} {% block title %}Login - My App{% endblock %} {%
block content %}
<h2>Login</h2>
{% with messages = get_flashed_messages(with_categories=true) %} {% if messages
%}
<ul class="flashes">
  {% for category, message in messages %}
  <li class="{{ category }}">{{ message }}</li>
  {% endfor %}
</ul>
{% endif %} {% endwith %}
<form method="POST" action="{{ url_for('auth.login_post') }}">
  <label for="email">Email:</label>
  <input type="email" id="email" name="email" required />
  <label for="password">Password:</label>
  <input type="password" id="password" name="password" required />
  <button type="submit">Login</button>
</form>
<!--  layout.html -->

{% if is_authenticated(session) %}
<nav>
  <a href="{{ url_for('home') }}">Home</a>
  <a href="{{ url_for('auth.logout') }}">Logout</a>
</nav>
{% else %}
<nav>
  <a href="{{ url_for('home') }}">Home</a>
  <a href="{{ url_for('auth.login') }}">Login</a>
</nav>
{% endif %}
```

```css
flash.error {
  color: red;
}
flash.success {
  color: green;
}
```

En config hay que agregar

```python
class Config:
  SECRET_JEY = "super"
  SESSION_TYPE = "filesystem"
  SQLARCHME_ENGINE_OPTIONS = {
    "pool_pre_ping": True,
    "pool_size": 10,
    "pool_recycle": 60,  # 60 segundos
  }
```

## Mapas

hay que isntalar postgis

```bash
sudo pacman -S postgis
poetry add geoalchemy2@latest
poetry add shapely@latest # Probar sin shapely
```

```sql
CREATE EXTENSION postgis;
```

```python
# models/location.py
from geoalchemy2 import Geometry

class Location(db.Model, Base):
    __tablename__ = 'locations'
    name = Column(String(100), nullable=False)
    geom = Column(Geometry(geometry_type='POINT', srid=4326), nullable=False)

  location: Mapped[str] = mapped_column(Geometry(geometry_type='POINT', srid=4326), nullable=False)

  @property
  def lat(self) -> float:
    if self.geom:
      punto = to_shape(self.geom)
      return self.geom.x
    return None
  @property
  def lon(self) -> float:
    if self.geom:
      punto = to_shape(self.geom)
      return self.geom.y
    return None

# seed.py
# WTK (Well-Known Text)
issue3 = board.create_location(name="Location 1", WKTElement=('POINT(30 10)', srid=4326))])

# controllers/location.py

@bp.get('/')
def index():
  locations = board.list_locations()
  return render_template('locations/index.html', locations=locations)

@bp.post('/add')
def add_location():
  params = request.form
  name = params.get('name')
  lat = float(params.get('lat'))
  lon = float(params.get('lon'))
  board.create_location(name=name, WKTElement=f'POINT({lon} {lat})')
  flash('Location added successfully', 'success')
  return redirect(url_for('locations.index'))
```

HTML

```html
<link
  rel="stylesheet"
  href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css"
/>
<script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
<script>
  var map = L.map("map").setView([0, 0], 2); // Centro del mapa y nivel de zoom

  //capa base
  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution: "&copy; OpenStreetMap contributors",
  }).addTo(map);

  map.on("click", function (e) {
    var lat = e.latlng.lat.toFixed(6);
    var lon = e.latlng.lng.toFixed(6);
    document.getElementById("lat").value = lat;
    document.getElementById("lon").value = lon;
    L.marker([lat, lon]).addTo(map);
  });
  var markers = {{ locations|tojson }};
  markers.forEach(function (loc) {
    var coords = loc.geom.coordinates;
    L.marker([coords[1], coords[0]]).addTo(map).bindPopup(loc.name);
  });
</script>

<style>
  #map {
    height: 400px;
    width: 100%;
  }
</style>

<div id="map"></div>
<form method="POST" action="{{ url_for('locations.add_location') }}">
  <input type="text" id="lat" name="lat" placeholder="Latitude" required />
  <input type="text" id="lon" name="lon" placeholder="Longitude" required />
  <button type="submit">Add Location</button>
</form>

<!-- templates/locations/index.html  -->
<script>
  var map = L.map("map").setView([0, 0], 2); // Centro del mapa y nivel de zoom
  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution: "&copy; OpenStreetMap contributors",
  }).addTo(map);
  var issues = {{ issues|tojson }};
  issues.forEach(function (issue) {
    var coords = issue.location.coordinates;
    L.marker([coords[1], coords[0]])
      .addTo(map)
      .bindPopup("<b>" + issue.title + "</b><br>" + issue.description);
  });
</script>
```

## Clase JS

Los script estan dentro del HTML
Van dentro de la etiqueta script, o en un archivo externo

```html
<script>
  // Seleccionar elemento por ID
    const element = document.getElementById('myElement');
    // Seleccionar elementos por clase
    const elements = document.getElementsByClassName('myClass');
    // Seleccionar elementos por etiqueta
    const tags = document.getElementsByTagName('div');
    // Seleccionar elementos con query selector (CSS)
    const queryElement = document.querySelector('.myClass');
    const queryElements = document.querySelectorAll('.myClass');
    // Modificar contenido
    element.textContent = 'Nuevo texto';
    element.innerHTML = '<strong>Texto en negrita</strong>';
    // Modificar estilos
    element.style.color = 'red';
    element.style.fontSize = '20px';
    // Manejar eventos
    element.addEventListener('click', function() {
        alert('Elemento clickeado!');
    });
    // Crear y agregar nuevos elementos
    const newElement = document.createElement('div');
    newElement.textContent = 'Nuevo elemento';
    document.body.appendChild(newElement);
</script>
```

Hay diferentes formas de ejecucion

```html
<script>
  // Codigo que se ejecuta inmediatamente
  console.log('Script cargado');
</script>
<script async>
  // Codigo que se ejecuta de forma asincronica
  fetch('https://api.example.com/data')
    .then(response => response.json())
    .then(data => console.log(data));
</script>
<script defer>
  // Codigo que se ejecuta despues de que el DOM este completamente cargado
  document.addEventListener('DOMContentLoaded', function() {
    console.log('DOM completamente cargado');
  });
</script>
```
En la consola del navegador se pueden ver errores y mensajes de log. Tambien se pueden ejecutar comandos JS directamente.

```html
<script>
 // Definiciones basicas
    let nombre = 'Juan'; // Variable que puede cambiar
    const PI = 3.1416; // Constante que no cambia
    var edad = 30; // Variable con alcance global o de funcion (menos usada)
    // Tipos de datos
    let numero = 42; // Numero
    // Strings
    let texto = 'Hola, mundo!';

    // Metodos de string
    texto.length; // Longitud del string
    texto.toUpperCase(); // Convierte a mayusculas
    texto.toLowerCase(); // Convierte a minusculas
    texto.includes('mundo'); // Verifica si contiene 'mundo'
    texto.replace('mundo', 'JavaScript'); // Reemplaza 'mundo'
    texto.split(', '); // Divide el string en un array ['Hola', 'mundo!']

    let booleano = true; // Booleano
    let arreglo = [1, 2, 3]; // Arreglo

    // Se pueden agregar de forma dinamica
    arreglo[3] = 4;
    arreglo.push(4);
    var frutas = new Array(); // Otra forma de crear arreglos
    frutas["citricos"] = ["naranja", "limon", "mandarina"]; // Arreglo asociativo
    frutas["tropicales"] = ["mango", "piña", "papaya"];
    console.log(frutas["citricos"][0]); // Acceder a elementos
    let objeto = { nombre: 'Juan', edad: 30 }; // Objeto

    findel2021 = new Date(2021,11,31); // Año, mes (0-11), día
    console.log(findel2021.getFullYear()); // 2021
    console.log(findel2021.getMonth()); // 11 (Diciembre)

    //Funciones
    function saludar(nombre) {
        return `Hola, ${nombre}!`;
    }
    function pasoAMayusculas(bandas){
        bandas = bandas.forEach(banda => banda.toUpperCase());
    }
    function recorridaConA(banda){
        for(let i=0; i<banda.length; i++){
            console.log(banda[i]);
        }
    }
    function recorrerForIn(bandas){
        for(let index in bandas){
            console.log(bandas[index]);
        }
    }
    function aleatoria(bandas){
        return bandas[Math.floor(Math.random() * bandas.length)];
    }
</script>
```
DOM: Document Object Model
El DOM es una representación estructurada del documento HTML que permite a los lenguajes de programación interactuar con el contenido, la estructura y el estilo de una página web. El DOM representa el documento como una jerarquía de nodos, donde cada nodo corresponde a un elemento HTML, atributo o texto.
Hay metodos para interactuar con el DOM

```html
<script>
  // Seleccionar elementos
    const element = document.getElementById('myElement');
    const elements = document.getElementsByClassName('myClass');
    const tags = document.getElementsByTagName('div');

    // Nodos del arbol DOM
    const parent = element.parentNode; // Nodo padre
    const children = element.childNodes; // Nodos hijos
    const firstChild = element.firstChild; // Primer nodo hijo
    const lastChild = element.lastChild; // Ultimo nodo hijo
    const nextSibling = element.nextSibling; // Siguiente nodo hermano
    const previousSibling = element.previousSibling; // Nodo hermano anterior

    // Crear y agregar nuevos elementos
    const newElement = document.createElement('div');
    newElement.textContent = 'Nuevo elemento';
    element.appendChild(newElement); // Agregar como hijo

    // Seleccionar
    const queryElement = document.querySelector('.myClass');

    //Jquery
    const queryElements = document.querySelectorAll('.myClass');
    const document.getElementById = $('#myElement');
    const document.getElementsByClassName = $('.myClass');
    const document.getElementsByTagName = $('div');
    const id1 = $('#id1');
    const clase1 = $('.clase1');
    const divs = $('div');
    const li_impares = $('li:nth-child(odd)');

    // Modificar arbol
    document.createElement('div');
    element.appendChild(newElement);
    element.removeChild(element.firstChild); // Eliminar primer hijo

</script>
```
Tipos de nodos:
- Element: Representa un elemento HTML (etiqueta)
- Text: Representa el contenido de texto dentro de un elemento
- Attribute: Representa un atributo de un elemento (no es un nodo en el DOM)
- Comment: Representa un comentario en el HTML
- Document: Representa el documento HTML completo

Nodos de texto:
para acceder al contenido de texto dentro de un elemento

```html
<script>
    const textNode = element.firstChild; // Primer nodo hijo (puede ser texto)
        const textContent = textNode.nodeValue; // Contenido del nodo de texto
        textNode.nodeValue = 'Nuevo texto'; // Modificar contenido de texto
</script>
```

### DOM y eventos
Los eventos son acciones o sucesos que ocurren en el navegador, como clics del ratón, pulsaciones de teclas, movimientos del ratón, carga de la página, entre otros. Los eventos permiten a los desarrolladores web crear interactividad en sus páginas al responder a estas acciones del usuario o del sistema.

```html
<script>
    // Manejar eventos
    element.addEventListener('click', function() {
        alert('Elemento clickeado!');
    });
    element.addEventListener('mouseover', function() {
        element.style.backgroundColor = 'yellow';
    });
    element.addEventListener('mouseout', function() {
        element.style.backgroundColor = '';
    });
    document.addEventListener('DOMContentLoaded', function() {
        console.log('DOM completamente cargado');
    });
</script>
```
### Interce events
Los eventos de intersección (Intersection Events) son eventos que se disparan cuando un elemento entra o sale del área visible del navegador (viewport) o de un contenedor específico. Estos eventos son útiles para implementar funcionalidades como la carga diferida de imágenes, animaciones al hacer scroll, o la detección de cuándo un elemento es visible para el usuario.
El flujo de eventos pasa por varias fases:
1. Capturing Phase (Fase de captura): El evento se propaga desde el objeto raíz (documento) hacia el objetivo, pasando por los nodos padres.
2. Target Phase (Fase objetivo): El evento llega al nodo objetivo donde se originó.
3. Bubbling Phase (Fase de burbuja): El evento se propaga de vuelta desde el nodo objetivo hacia el objeto raíz, pasando nuevamente por los nodos padres.
```html
<script>
    // Ejemplo de Intersection Observer
    var elem= document.getElementById('myElement');
    elem.onclick = function() {
        console.log('Elemento clickeado!');
    };
    elem.onmouseover = function() {
        elem.style.backgroundColor = 'yellow';
    };
    elem.onmouseout = function() {
        elem.style.backgroundColor = '';
    };
</script>
```
Escuchadores de eventos comunes:
- click: Se dispara cuando se hace clic en un elemento.
- mouseover: Se dispara cuando el puntero del ratón entra en un elemento.
- mouseout: Se dispara cuando el puntero del ratón sale de un elemento.
- keydown: Se dispara cuando se presiona una tecla.
- keyup: Se dispara cuando se suelta una tecla.
- submit: Se dispara cuando se envía un formulario.
- change: Se dispara cuando el valor de un elemento de formulario cambia.
- load: Se dispara cuando una página o un recurso se ha cargado completamente.

```html
<script>
    // Ejemplo de Intersection Observer
    var element = document.getElementById('myElement');
    elem.addEventListener ("mouseover",funcion1, true) // Capturing phase
    elem.addEventListener ("mouseover",funcion2, false) // Bubbling phase
</script>
```

## Objetos en JS

Los objetos en JavaScript son colecciones de pares clave-valor que permiten almacenar y organizar datos relacionados. Cada clave (también llamada propiedad) es una cadena de texto, y cada valor puede ser de cualquier tipo de dato, incluyendo otros objetos, funciones, arreglos, etc. Los objetos son fundamentales en JavaScript y se utilizan ampliamente para representar entidades del mundo real, configurar opciones, manejar datos complejos, entre otros usos.
Prototipos y herencia prototípica
JavaScript utiliza un sistema de herencia basado en prototipos, lo que significa que los objetos pueden heredar propiedades y métodos de otros objetos. Cada objeto tiene un enlace interno a otro objeto llamado prototipo. Cuando se accede a una propiedad o método de un objeto, JavaScript primero busca en el objeto mismo; si no lo encuentra, busca en su prototipo, y así sucesivamente hasta llegar al final de la cadena de prototipos (generalmente Object.prototype).

```html
<script>
    // Crear un objeto
    const persona = {
        nombre: 'Juan',
        edad: 30,
        saludar: function() {
            console.log(`Hola, mi nombre es ${this.nombre} y tengo ${this.edad} años.`);
        }
    };
    // Acceder a propiedades y métodos
    console.log(persona.nombre); // 'Juan'
    persona.saludar(); // 'Hola, mi nombre es Juan y tengo 30 años.'
    // Agregar nuevas propiedades y métodos
    persona.apellido = 'Pérez';
    persona.despedir = function() {
        console.log(`Adiós, ${this.nombre}`);
    };
    persona.despedir(); // 'Adiós, Juan'
    // Eliminar propiedades
    delete persona.edad;
    console.log(persona.edad); // undefined
    // Crear un objeto usando una función constructora
    function Coche(marca, modelo) {
        this.marca = marca;
        this.modelo = modelo;
        this.detalles = function() {
            console.log(`Coche: ${this.marca} ${this.modelo}`);
        };
    }
    const miCoche = new Coche('Toyota', 'Corolla');
    miCoche.detalles(); // 'Coche: Toyota Corolla'
    // Tambien se puede uscar class, pero es unn template
    class Animal {
        constructor(nombre, especie) {
            this.nombre = nombre;
            this.especie = especie;
        }
        get hacerSonido() {
            console.log(`${this.nombre} hace un sonido.`);
        }
        set cambiarNombre(nuevoNombre) {
            nuevoNombre = nuevoNombre.toUpperCase();
            this.nombre = nuevoNombre;
        }
        get info() {
            return `${this.nombre} es un ${this.especie}.`;
        }
    }
    // se usa get y set para encapsular propiedades
    const perro = new Animal('Firulais', 'Perro');
    perro.hacerSonido(); // 'Firulais hace un sonido.'
    perro.cambiarNombre = 'Max';
    console.log(perro.info); // 'MAX es un Perro.'
</script>
```
Todos los objetos en JS heredan de Object.prototype, que proporciona métodos y propiedades comunes como toString(), hasOwnProperty(), etc. Se pueden crear objetos personalizados y establecer su prototipo utilizando Object.create() o mediante funciones constructoras y clases.
Object.prototype es el prototipo base del cual todos los objetos en JavaScript heredan propiedades y métodos. Proporciona funcionalidades comunes que están disponibles para todos los objetos, como toString(), hasOwnProperty(), isPrototypeOf(), entre otros. Al crear un objeto, este hereda automáticamente las propiedades y métodos definidos en Object.prototype a través de la cadena de prototipos.
```html
<script>
    class Jugador {
        constructor(nombre, posicion) {
            this.nombre = nombre;
            this.posicion = posicion;
        }
        get descripcion() {
            return `${this.nombre} juega como ${this.posicion}.`;
        }
    }

    class Futbolista extends Jugador {
        constructor(nombre, posicion, equipo) {
            super(nombre, posicion); // Llamar al constructor de la clase base
            this.equipo = equipo;
        }
        get descripcionCompleta() {
            super.descripcion; // Llamar al getter de la clase base
            return `${this.nombre} juega como ${this.posicion} en el equipo ${this.equipo}.`;
        }
    }
    // el prototipo de Futbolista es Jugador.prototype
</script>
```
### JSON
JSON (JavaScript Object Notation) es un formato ligero de intercambio de datos que es fácil de leer y escribir para los humanos, y fácil de parsear y generar para las máquinas. JSON se utiliza comúnmente para transmitir datos entre un servidor y una aplicación web como texto plano. Está basado en una sintaxis similar a la de los objetos en JavaScript, pero es independiente del lenguaje, lo que lo hace ampliamente compatible con muchos lenguajes de programación.
```html
<script>
    // Convertir objeto a JSON
    const persona = {
        nombre: 'Juan',
        edad: 30,
        direccion: {
            calle: 'Calle Falsa 123',
            ciudad: 'Springfield'
        },
        hobbies: ['futbol', 'lectura', 'viajar']
    };
    const personaJSON = JSON.stringify(persona);
    console.log(personaJSON);
    // Convertir JSON a objeto
    const jsonString = '{"nombre":"Juan","edad":30,"direccion":{"calle":"Calle Falsa 123","ciudad":"Springfield"},"hobbies":["futbol","lectura","viajar"]}';
    const personaObj = JSON.parse(jsonString);
    console.log(personaObj);
</script>
```

### VueJS
Vue.js es un framework progresivo de JavaScript utilizado para construir interfaces de usuario y aplicaciones web. Fue creado por Evan You y se lanzó por primera vez en 2014. Vue.js se destaca por su simplicidad, flexibilidad y facilidad de integración con otros proyectos o bibliotecas. Es especialmente popular para desarrollar aplicaciones de una sola página (SPA) debido a su capacidad para manejar el estado y la reactividad de manera eficiente.
```html
<script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
<div id="app">
  <h1>{{ mensaje }}</h1>
  <input v-model="mensaje" placeholder="Escribe algo" />
  <button @click="mostrarAlerta">Mostrar Alerta</button>
</div>
<script>
  new Vue({
    el: '#app',
    data: {
      mensaje: 'Hola, Vue.js!'
    },
    methods: {
      mostrarAlerta() {
        alert(this.mensaje);
      }
    }
  });
</script>
```



#### Excepciones
Las excepciones en JavaScript son eventos que ocurren durante la ejecución de un programa y que interrumpen el flujo normal del código. Cuando se produce una excepción, el motor de JavaScript detiene la ejecución del código en el punto donde ocurrió la excepción y busca un manejador de excepciones adecuado para manejar el error. Si no se encuentra un manejador, el programa puede terminar abruptamente o mostrar un mensaje de error en la consola.
```html
<script>
    // Manejo de excepciones
    try {
        // Código que puede lanzar una excepción
        let resultado = 10 / 0; // División por cero
        console.log(resultado);
    } catch (error) {
        // Código para manejar la excepción
        console.error('Ocurrió un error:', error.message);
    } finally {
        // Código que se ejecuta siempre, haya o no excepción
        console.log('Bloque finally ejecutado');
    }
    try {
        let data = JSON.parse('{"nombre": "Juan", "edad": 30}'); // JSON válido
        if (!data.nombre) {
            throw new Error('El campo nombre es obligatorio');
        }
    } catch (error) {
        console.error('Error al parsear JSON:', error.message);
    }
</script>
```

## Clase Seguridad SQLi - XSS
Un diseño seguro de aplicaciones web debe considerar múltiples aspectos para protegerse contra diversas amenazas y vulnerabilidades. Aquí hay algunos principios clave para un diseño seguro:
- Segmentacion: Separar las diferentes partes de la aplicación (frontend, backend, base de datos) para limitar el alcance de un ataque.
- Usabilidad: Asegurarse de que las medidas de seguridad no dificulten la experiencia del usuario.
- Principio de menor privilegio: Otorgar a los usuarios y componentes solo los permisos necesarios para realizar sus tareas.
- Defensa en profundidad: Implementar múltiples capas de seguridad para proteger la aplicación.
- KISS (Keep It Simple, Stupid): Mantener el diseño y la implementación lo más simple posible para reducir la superficie de ataque.
- Failsafe: Diseñar la aplicación para que falle de manera segura, minimizando el impacto de un fallo.
- Separacion de datos y control: Mantener separados los datos de usuario y la lógica de control para evitar inyecciones de código.
- Open Design: Utilizar componentes y bibliotecas de código abierto que hayan sido revisados por la comunidad.

OWASP (Open Web Application Security Project) es una organización sin fines de lucro que se dedica a mejorar la seguridad del software. Proporciona recursos, herramientas y mejores prácticas para ayudar a los desarrolladores a crear aplicaciones web seguras. Uno de los proyectos más conocidos de OWASP es el OWASP Top Ten, que identifica las diez principales vulnerabilidades de seguridad en aplicaciones web.
(OWASP) [https://owasp.org/www-project-top-ten/]


### SQLi
La inyección SQL (SQL Injection o SQLi) es una vulnerabilidad de seguridad que ocurre cuando un atacante puede insertar o "inyectar" código SQL malicioso en una consulta SQL legítima. Esto puede permitir al atacante manipular la base de datos, acceder a datos sensibles, modificar o eliminar registros, e incluso tomar el control completo del servidor de la base de datos.
```python
# Ejemplo vulnerable a SQLi
def get_user_by_email(email):
    query = f"SELECT * FROM users WHERE email = '{email}'"
    result = db.session.execute(query)
    return result.fetchone()
# Un atacante podría pasar un email como: ' OR '1'='1
# La consulta resultante sería: SELECT * FROM users WHERE email = '' OR '1'='1'
# Esto devolvería todos los usuarios, ya que '1'='1' siempre es verdadero.
```
Para prevenir la inyección SQL, es importante utilizar consultas preparadas o declaraciones parametrizadas, que separan el código SQL de los datos proporcionados por el usuario. En SQLAlchemy, esto se puede lograr utilizando el ORM o pasando parámetros a las consultas.
```python
# Ejemplo seguro usando SQLAlchemy ORM
def get_user_by_email(email):
    user = User.query.filter(User.email == email).first()
    return user
# Ejemplo seguro usando consultas parametrizadas
from sqlalchemy import text
def get_user_by_email(email):
    query = text("SELECT * FROM users WHERE email = :email")
    result = db.session.execute(query, {'email': email})
    return result.fetchone()
```
Estructuras para sqli para obtener acceso:
- ' OR '1'='1 --
- ' UNION SELECT username, password FROM users --
- ' DROP TABLE users; --
Recordar que -- indica un comentario en SQL, por lo que todo lo que sigue después de -- es ignorado por el motor de la base de datos.

### XSS
La vulnerabilidad de Cross-Site Scripting (XSS) ocurre cuando un atacante logra inyectar código malicioso, generalmente en forma de scripts, en páginas web que son vistas por otros usuarios. Esto puede permitir al atacante robar información sensible, como cookies de sesión, redirigir a los usuarios a sitios maliciosos, o realizar acciones en nombre del usuario afectado.
```html
<!-- Ejemplo vulnerable a XSS -->
<div>
  <h1>Bienvenido, {{ username }}</h1>
</div>
<!-- Si username es: <script>alert('XSS')</script> -->
<!-- El navegador ejecutará el script, mostrando una alerta -->
```
Para prevenir XSS, es fundamental sanitizar y escapar cualquier entrada de usuario antes de renderizarla en la página web. En Flask, Jinja2 escapa automáticamente las variables al renderizar plantillas, pero es importante asegurarse de no desactivar esta funcionalidad.
```html
<!-- Ejemplo seguro -->
<div>
  <h1>Bienvenido, {{ username | e }}</h1>
</div>
<!-- El filtro | e escapa caracteres especiales, previniendo la ejecución de scripts -->
``````html
<!-- Ejemplo seguro usando Jinja2 -->
<div>
  <h1>Bienvenido, {{ username | e }}</h1>
</div>
<!-- El filtro | e escapa caracteres especiales, previniendo la ejecución de scripts -->
```
Hay diferentes tipos de XSS:
- Reflejado (Reflected XSS): El código malicioso se refleja en la respuesta del servidor, generalmente a través de parámetros en la URL o formularios.
- Almacenado (Stored XSS): El código malicioso se almacena en la base de datos o en el servidor y se sirve a los usuarios cuando acceden a la página afectada.
- DOM-based XSS: El código malicioso se ejecuta debido a la manipulación del DOM en el navegador, sin interacción directa con el servidor.
Para prevenir XSS, además de sanitizar y escapar entradas, se pueden implementar políticas de seguridad de contenido (Content Security Policy, CSP) que restringen las fuentes desde las cuales se pueden cargar scripts y otros recursos.


## Clase Autenticacion OAuth/JWT
OAuth (Open Authorization) es un protocolo abierto que permite a las aplicaciones obtener acceso limitado a cuentas de usuario en un servicio HTTP, como Facebook, GitHub, o Google, sin exponer las credenciales del usuario. OAuth funciona mediante la delegación de acceso a través de tokens, lo que permite a los usuarios autorizar a una aplicación para actuar en su nombre sin compartir su contraseña.
JWT (JSON Web Token) es un estándar abierto (RFC 7519) que define un formato compacto y autónomo para transmitir información segura entre partes como un objeto JSON. Los JWT son comúnmente utilizados para la autenticación y autorización en aplicaciones web, ya que permiten verificar la identidad del usuario y sus permisos de manera eficiente.

### OAuth 2.0
OAuth 2.0 es la versión más reciente del protocolo OAuth y es ampliamente utilizado para la autorización en aplicaciones web y móviles. OAuth 2.0 define varios flujos de autorización (grant types) que se adaptan a diferentes escenarios de uso, como aplicaciones web, aplicaciones móviles, y aplicaciones de servidor a servidor. Algunos de los flujos más comunes incluyen:
- Authorization Code Grant: Utilizado por aplicaciones web y móviles que pueden mantener la confidencialidad del cliente. Implica un intercambio de código de autorización por un token de acceso.
- Implicit Grant: Utilizado por aplicaciones web de una sola página (SPA) donde el token de acceso se entrega directamente al cliente sin un código de autorización.
- Resource Owner Password Credentials Grant: Utilizado en situaciones donde el usuario confía completamente en la aplicación cliente y proporciona sus credenciales directamente.
- Client Credentials Grant: Utilizado para la autorización de aplicaciones de servidor a servidor, donde no hay un usuario involucrado.



### OAuth
El flujo de OAuth generalmente implica los siguientes pasos:
1. El usuario inicia sesión en la aplicación cliente y solicita acceso a un recurso protegido.
2. La aplicación cliente redirige al usuario al servidor de autorización del proveedor de OAuth (por ejemplo, Google) para que inicie sesión y otorgue permisos.
3. El usuario concede permisos y el servidor de autorización redirige al usuario de vuelta a la aplicación cliente con un código de autorización.
4. La aplicación cliente intercambia el código de autorización por un token de acceso en el servidor de autorización.
5. La aplicación cliente utiliza el token de acceso para acceder a los recursos protegidos en nombre del usuario.

Es necesario registrar la aplicación cliente con el proveedor de OAuth para obtener un client_id y client_secret, que se utilizan para autenticar la aplicación durante el proceso de autorización.
Consola de Google: https://console.developers.google.com/api/credentials
De aqui se obtiene el client_id y client_secret. Se preciosa crear credenciales de OAuth 2.0, tipo de aplicacion web. Hay que configurar la URI de redireccion (redirect URI) que es la URL a la que Google redirigirá al usuario después de que haya autorizado la aplicación. Esta URL debe coincidir exactamente con la que se utiliza en el código de la aplicación.
Recordar que OAuth funciona con https, no con http.

Para tener con flask un certificado ssl hay que agregar en config.py
```python
class Config:
  # otras configuraciones...
  SSL_CERT_PATH = 'path/to/cert.pem'
  SSL_KEY_PATH = 'path/to/key.pem'

# Tambien se puede utilizar adhoc
app.run(ssl_context='adhoc')
```



```python
# Ejemplo de flujo OAuth con Flask y Authlib
from authlib.integrations.flask_client import OAuth
oauth = OAuth(app)
google = oauth.register(
    name='google',
    client_id='YOUR_CLIENT_ID',
    client_secret='YOUR_CLIENT_SECRET',
    access_token_url='https://accounts.google.com/o/oauth2/token',
    authorize_url='https://accounts.google.com/o/oauth2/auth',
    client_kwargs={'scope': 'openid profile email'},
)
@app.route('/login')
def login():
    redirect_uri = url_for('authorize', _external=True)
    return google.authorize_redirect(redirect_uri)
@app.route('/authorize')
def authorize():
    token = google.authorize_access_token()
    resp = google.get('userinfo')
    user_info = resp.json()
    # Aquí se puede crear o actualizar el usuario en la base de datos
    session['user'] = user_info
    return redirect('/')
```


### JWT
Un JWT consta de tres partes separadas por puntos: el encabezado (header), el cuerpo (payload) y la firma (signature). El encabezado contiene información sobre el tipo de token y el algoritmo de firma utilizado. El cuerpo contiene las declaraciones (claims) que representan la información del usuario y sus permisos. La firma se utiliza para verificar la integridad del token y asegurar que no ha sido alterado.
Estructura del toke: header.payload.signature
- Header: {"alg": "HS256", "typ": "JWT"}
- Payload: {"sub": "1234567890", "name": "John Doe", "iat": 1516239022}
- Signature: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)

```python
# Ejemplo de uso de JWT con Flask y PyJWT
import jwt
import datetime
SECRET_KEY = 'your_secret_key'
def create_token(user_id):
    payload = {
        'user_id': user_id,
        'exp': datetime.datetime.utcnow() + datetime.timedelta(hours=1)  # Expira en 1 hora
    }
    token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
    return token
def decode_token(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        return payload['user_id']
    except jwt.ExpiredSignatureError:
        return None  # Token expirado
    except jwt.InvalidTokenError:
        return None  # Token inválido
@app.route('/login', methods=['POST'])
def login():
    data = request.json
    user = authenticate(data['username'], data['password'])
    if user:
        token = create_token(user.id)
        return jsonify({'token': token})
    return jsonify({'message': 'Invalid credentials'}), 401
@app.route('/protected')
def protected():
    token = request.headers.get('Authorization').split()[1]  # Asumiendo formato 'Bearer token'
    user_id = decode_token(token)
    if user_id:
        return jsonify({'message': f'Access granted for user {user_id}'})
    return jsonify({'message': 'Invalid or expired token'}), 401
```

# Clase Intro VueJS
## Microservicios
Los microservicios son una arquitectura de software que descompone una aplicación en un conjunto de servicios pequeños, independientes y autónomos. Cada microservicio se enfoca en una funcionalidad específica y puede ser desarrollado, desplegado y escalado de manera independiente. Esta arquitectura contrasta con el enfoque monolítico, donde toda la aplicación se construye como una única unidad.
### Características
- Independencia: Cada microservicio puede ser desarrollado y desplegado de manera independiente, lo que facilita la actualización y el mantenimiento.
- Especialización: Cada microservicio se enfoca en una funcionalidad específica, lo que permite un desarrollo más rápido y especializado.
- Escalabilidad: Los microservicios pueden escalarse de manera independiente según la demanda, lo que optimiza el uso de recursos.
- Resiliencia: La falla de un microservicio no afecta necesariamente a toda la aplicación, lo que mejora la disponibilidad y la tolerancia a fallos.
- Comunicación: Los microservicios se comunican entre sí a través de APIs, generalmente utilizando protocolos ligeros como HTTP/REST o mensajería asíncrona.

En esta arquitectura se suele utilizar la misma UI para todos los microservicios, que se comunica con cada uno de ellos a través de APIs RESTful o GraphQL. La UI puede estar construida con frameworks modernos como React, Vue.js o Angular, y se encarga de orquestar las interacciones entre los diferentes microservicios. Incluso se puede centrar la comunicacion utilizando una API Gateway, que actúa como un punto de entrada único para todas las solicitudes de la UI hacia los microservicios.
Los message brokers como RabbitMQ, Apache Kafka o AWS SQS se utilizan para facilitar la comunicación asíncrona entre microservicios. Estos sistemas permiten que los microservicios envíen y reciban mensajes de manera eficiente, lo que mejora la escalabilidad y la resiliencia de la arquitectura.


## VueJS
Vue.js es un framework progresivo de JavaScript utilizado para construir interfaces de usuario y aplicaciones web. Fue creado por Evan You y se lanzó por primera vez en 2014. Vue.js se destaca por su simplicidad, flexibilidad y facilidad de integración con otros proyectos o bibliotecas. Es especialmente popular para desarrollar aplicaciones de una sola página (SPA) debido a su capacidad para manejar el estado y la reactividad de manera eficiente.
Vue.js utiliza un enfoque basado en componentes, donde cada componente encapsula su propia lógica, plantilla y estilos. Esto facilita la reutilización de código y la organización de la aplicación. Vue.js también ofrece un sistema de enlace de datos bidireccional (two-way data binding), lo que significa que los cambios en el modelo de datos se reflejan automáticamente en la vista y viceversa.
Vue.js es conocido por su curva de aprendizaje suave, lo que lo hace accesible tanto para desarrolladores principiantes como para expertos. Además, cuenta con una comunidad activa y una amplia gama de plugins y herramientas que amplían sus capacidades.

### Caracteristicas
- Reactividad: Vue.js utiliza un sistema de reactividad que permite que la interfaz de usuario se actualice automáticamente cuando los datos cambian."Two-way data binding": cuando los datos cambian, la vista se actualiza automáticamente, y viceversa.
- Componentes: Vue.js se basa en componentes reutilizables que encapsulan su propia lógica, plantilla y estilos.
- Directivas: Vue.js proporciona directivas especiales (como v-if, v-for, v-model) que facilitan la manipulación del DOM y la gestión de eventos.
- Patron MVVM (Model-View-ViewModel): Vue.js sigue el patrón MVVM, que separa la lógica de la aplicación (modelo) de la interfaz de usuario (vista) a través de un intermediario (ViewModel).
- Ecosistema: Vue.js cuenta con un ecosistema rico que incluye herramientas como Vue Router (para el enrutamiento), Vuex (para la gestión del estado) y Vue CLI (para la creación y gestión de proyectos).
- Integración: Vue.js se puede integrar fácilmente con otras bibliotecas o proyectos existentes, lo que lo hace adecuado para una amplia variedad de casos de uso.
- Curva de aprendizaje suave: Vue.js es conocido por su facilidad de uso y su curva de aprendizaje suave, lo que lo hace accesible tanto para desarrolladores principiantes como para expertos.
- Comunidad activa: Vue.js tiene una comunidad vibrante y en crecimiento, con una amplia gama de recursos,tutoriales y plugins disponibles.

### Instalacion

Hay varias formas de instalar Vue.js en un proyecto, dependiendo de las necesidades y el entorno de desarrollo. Aquí hay algunas de las formas más comunes de instalar Vue.js:
1. CDN (Content Delivery Network): La forma más sencilla de comenzar a usar Vue.js es incluirlo directamente desde un CDN en el archivo HTML. Esto es útil para proyectos pequeños o para probar Vue.js rápidamente.

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
```
2. NPM (Node Package Manager): Para proyectos más grandes o cuando se utiliza un sistema de construcción como Webpack o Vue CLI, es común instalar Vue.js a través de NPM.
```bash
npm install vue
```
3. Yarn: Similar a NPM, Yarn es otro gestor de paquetes que se puede utilizar para instalar Vue.js.
```bash
yarn add vue
```
4. Vue CLI: Vue CLI es una herramienta oficial para crear y gestionar proyectos Vue.js. Proporciona una configuración predefinida y facilita la creación de aplicaciones Vue.js con características avanzadas.
```bash
npm install -g @vue/cli
vue create my-project
cd my-project
npm run serve
```
5. Vue.js con Vite: Vite es una herramienta de construcción moderna que se utiliza para desarrollar aplicaciones Vue.js de manera rápida y eficiente. Puedes crear un nuevo proyecto Vue.js con Vite utilizando el siguiente comando:
```bash
npm create vite@latest my-vue-app -- --template vue
cd my-vue-app
npm install
npm run dev
```
5. Vue.js con frameworks de construcción: Si estás utilizando frameworks como Nuxt.js (para aplicaciones universales) o Quasar (para aplicaciones móviles y de escritorio), Vue.js ya está integrado y no es necesario instalarlo por separado.

### Extension para navegadores
- Vue.js devtools: Una extensión para navegadores como Chrome y Firefox que permite inspeccionar y depurar aplicaciones Vue.js de manera más eficiente. Proporciona una interfaz visual para ver la estructura de componentes, el estado y las propiedades reactivas.

### Estructura de un proyecto Vue.js
Un proyecto Vue.js típico tiene una estructura de archivos organizada que facilita el desarrollo y la gestión del código. Aquí hay una estructura de proyecto comúnmente utilizada:
```my-vue-project/
├── node_modules/          # Dependencias instaladas por NPM
├── public/                # Archivos estáticos (index.html, favicon, etc.)
│   └── index.html         # Archivo HTML principal
├── src/                   # Código fuente de la aplicación
│   ├── assets/            # Recursos estáticos (imágenes, estilos, etc.)
│   ├── components/        # Componentes Vue reutilizables
│   │   └── HelloWorld.vue # Ejemplo de componente Vue
│   ├── views/             # Vistas o páginas de la aplicación
│   │   └── Home.vue       # Ejemplo de vista
│   ├── router/            # Configuración del enrutador (si se usa Vue Router)
│   │   └── index.js       # Archivo de configuración del enrutador
│   ├── store/             # Gestión del estado (si se usa Vuex)
│   │   └── index.js       # Archivo de configuración de Vuex
│   ├── App.vue            # Componente raíz de la aplicación
│   └── main.js            # Punto de entrada de la aplicación
├── .gitignore             # Archivos y carpetas a ignorar por Git
├── package.json           # Información del proyecto y dependencias
├── package-lock.json      # Archivo de bloqueo de dependencias (NPM)
└── README.md              # Documentación del proyecto
```

### Componentes en Vue.js
Los componentes son bloques de construcción reutilizables en Vue.js que encapsulan su propia lógica, plantilla y estilos. Cada componente puede tener su propio estado, propiedades (props) y métodos, lo que facilita la organización y el mantenimiento del código. Los componentes pueden ser anidados, lo que permite crear interfaces de usuario complejas a partir de piezas más pequeñas y manejables.
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue.js Example</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
</head>
<body>
    <div id="app">
        <h1>{{ message }}</h1>
        <input v-model="message" placeholder="Type something">
        <button @click="reverseMessage">Reverse Message</button>
    </div>
    <script>
        const HelloVueApp = {
            data() {
                return {
                    message: 'Hello, Vue.js!'
                }
            },
            methods: {
                reverseMessage() {
                    this.message = this.message.split('').reverse().join('');
                }
            }
        }
        Vue.createApp(HelloVueApp).mount('#app');
    </script>
</body>
</html>
```

# Clase VueJS Directivas - Eventos
## Directivas en Vue.js
Las directivas en Vue.js son atributos especiales que se utilizan para enlazar datos y manipular el DOM de manera declarativa. Las directivas comienzan con el prefijo "v-" y proporcionan funcionalidades específicas para interactuar con el modelo de datos y la vista. Algunas de las directivas más comunes en Vue.js incluyen:
- v-if: Renderiza un elemento condicionalmente basado en una expresión booleana.
- v-for: Renderiza una lista de elementos basados en una colección.
- v-bind: Enlaza atributos HTML a datos del modelo.
- v-model: Crea un enlace bidireccional entre un elemento de formulario y una propiedad del modelo.
- v-on: Escucha eventos del DOM y ejecuta métodos en respuesta.
- v-show: Muestra u oculta un elemento basado en una expresión booleana, pero no lo elimina del DOM.
- v-pre: Salta la compilación de Vue para el elemento y sus hijos.
- v-cloak: Mantiene el elemento oculto hasta que Vue.js haya compilado la plantilla.
- v-once: Renderiza el elemento y sus hijos solo una vez, evitando futuras actualizaciones.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue.js Directives Example</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
</head>
<body>
    <div id="app">
        <laberl v-bind:title="message">Hover over me to see the message</laberl>
        <h1 v-if="isVisible">Hello, Vue.js!</h1>
        <h1 v-else>Goodbye, Vue.js!</h1>
        <ul>
            <li v-for="item in items" :key="item.id">
                {{ item.name }}
            </li>
        </ul>
        <input v-model="newItem" placeholder="Add new item">
        <h3>{{ newItem }}</h3>
        <button @click="addItem">Add Item</button>
        <button v-on:click="toggleVisibility">Toggle Visibility</button>
    </div>
    <script>
        const app = {
            data() {
                return {
                    isVisible: true,
                    items: [
                        { id: 1, name: 'Item 1' },
                        { id: 2, name: 'Item 2' },
                        { id: 3, name: 'Item 3' }
                    ],
                    newItem: '',
                    message: 'Hello, Vue.js!'
                }
            },
            methods: {
                addItem() {
                    if (this.newItem.trim()) {
                        this.items.push({ id: this.items.length + 1, name: this.newItem });
                        this.newItem = '';
                    }
                },
                toggleVisibility() {
                    this.isVisible = !this.isVisible;
                }
            }
        }
        Vue.createApp(app).mount('#app');
    </script>
</body>
</html>
```
v-on:
- click: Se dispara cuando un elemento es clickeado.
- dblclick: Se dispara cuando un elemento es doble clickeado.
- mouseover: Se dispara cuando el puntero del ratón entra en el área de un elemento.
- mouseout: Se dispara cuando el puntero del ratón sale del área de un elemento.
- keyup: Se dispara cuando una tecla es liberada.
- keydown: Se dispara cuando una tecla es presionada.
- submit: Se dispara cuando un formulario es enviado.
- change: Se dispara cuando el valor de un elemento de formulario cambia.
- focus: Se dispara cuando un elemento recibe el foco.
- blur: Se dispara cuando un elemento pierde el foco.
- input: Se dispara cuando el valor de un elemento de entrada cambia.
- contextmenu: Se dispara cuando se hace clic derecho en un elemento, abriendo el menú contextual.
- scroll: Se dispara cuando un elemento es desplazado (scrolled).

v-model.lazy: Actualiza el modelo solo cuando el elemento pierde el foco (blur).
v-model.number: Convierte automáticamente el valor de entrada a un número.
v-model.trim: Elimina los espacios en blanco al inicio y al final del valor de entrada.

#### Propiedades computadas
Las propiedades computadas en Vue.js son propiedades que se definen en la sección "computed" de un componente y se utilizan para calcular valores basados en otras propiedades del modelo. A diferencia de los métodos, las propiedades computadas son reactivas y se almacenan en caché, lo que significa que solo se recalculan cuando sus dependencias cambian. Esto las hace ideales para realizar cálculos complejos o formatear datos sin necesidad de llamar a un método cada vez que se accede a la propiedad.
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue.js Computed Properties Example</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
</head>
<body>
    <div id="app">
        <h1>Full Name: {{ fullName }}</h1>
        <input v-model="firstName" placeholder="First Name">
        <input v-model="lastName" placeholder="Last Name">
    </div>
    <script>
        const app = {
            data() {
                return {
                    firstName: '',
                    lastName: ''
                }
            },
            computed: {
                fullName() {
                    return `${this.firstName} ${this.lastName}`.trim();
                }
            }
        }
        Vue.createApp(app).mount('#app');
    </script>
</body>
</html>
```

#### Watchers
Los watchers en Vue.js son funciones que se utilizan para observar cambios en propiedades específicas del modelo de datos. A diferencia de las propiedades computadas, que se recalculan automáticamente cuando sus dependencias cambian, los watchers permiten ejecutar código personalizado en respuesta a cambios en una propiedad. Esto es útil para realizar acciones secundarias, como llamadas a APIs, validaciones o actualizaciones de otras propiedades, cuando una propiedad cambia.
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue.js Watchers Example</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
</head>
<body>
    <div id="app">
        <h1>Counter: {{ counter }}</h1>
        <button @click="incrementCounter">Increment Counter</button>
    </div>
    <script>
        const app = {
            data() {
                return {
                    counter: 0
                }
            },
            methods: {
                incrementCounter() {
                    this.counter++;
                }
            },
            watch: {
                counter: {
                  handler(newValue, oldValue) {
                      console.log(`Counter changed from ${oldValue} to ${newValue}`);
                      if (newValue >= 5) {
                          alert('Counter reached 5!');
                      }
                  },
                  deep: true,
                }
            }
        }
        Vue.createApp(app).mount('#app');
    </script>
</body>
</html>
```

### Consumir APIs
Se puede utilizar fetch o axios para consumir APIs en Vue.js. Aquí hay un ejemplo utilizando fetch:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue.js Fetch API Example</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
</head>
<body>
    <div id="app">
        <h1>Users</h1>
        <ul>
            <li v-for="user in users" :key="user.id">
                {{ user.name }} ({{ user.email }})
            </li>
        </ul>
        <button @click="fetchUsers">Fetch Users</button>
    </div>
    <script>
        const app = {
            data() {
                return {
                    users: []
                }
            },
            created() {
                this.fetchUsers();
                async fetchUsers() {
                    try {
                        const response = await fetch('https://jsonplaceholder.typicode.com/users');
                        const data = await response.json();
                        this.users = data;
                    } catch (error) {
                        console.error('Error fetching users:', error);
                    }
                }
            }
        }
        Vue.createApp(app).mount('#app');
    </script>
</body>
</html>

```
## Consumir api con Axios
Axios es una biblioteca de JavaScript que facilita la realización de solicitudes HTTP desde el navegador o Node.js. Es una alternativa popular a la función fetch nativa de JavaScript, ya que ofrece una sintaxis más sencilla y características adicionales, como la interceptación de solicitudes y respuestas, la cancelación de solicitudes y la transformación automática de datos JSON.
Para utilizar Axios en un proyecto Vue.js, primero debes instalarlo a través de NPM o incluirlo desde un CDN. Aquí hay un ejemplo de cómo consumir una API utilizando Axios en Vue.js:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue.js Axios Example</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
</head>
<body>
    <div id="app">
        <h1>Users</h1>
        <ul>
            <li v-for="user in users" :key="user.id">
                {{ user.name }} ({{ user.email }})
            </li>
        </ul>
        <button @click="fetchUsers">Fetch Users</button>
    </div>
    <script>
        const app = {
            data() {
                return {
                    users: []
                }
            },
            created() {
                this.fetchUsers();
            },
            methods: {
                async fetchUsers() {
                    try {
                        const response = await axios.get('https://jsonplaceholder.typicode.com/users');
                        this.users = response.data;
                    } catch (error) {
                        this.users = ["Error fetching users"];
                        console.error('Error fetching users:', error);
                    }
                }
            }
        }
        Vue.createApp(app).mount('#app');
    </script>
</body>
</html>
```

## Ciclo de vida de un componente
El ciclo de vida de un componente en Vue.js se refiere a las diferentes etapas por las que pasa un componente desde su creación hasta su destrucción. Vue.js proporciona varios hooks (ganchos) que permiten a los desarrolladores ejecutar código en momentos específicos del ciclo de vida del componente. Aquí hay una descripción de las principales etapas del ciclo de vida de un componente en Vue.js:
1. Creación (Creation):
   - beforeCreate: Se llama antes de que el componente sea creado. En este punto, las propiedades reactivas y los eventos aún no están disponibles.
   - created: Se llama después de que el componente ha sido creado. En este punto, las propiedades reactivas y los eventos están disponibles, pero el DOM aún no ha sido montado.
2. Montaje (Mounting):
   - beforeMount: Se llama justo antes de que el componente sea montado en el DOM.
   - mounted: Se llama después de que el componente ha sido montado en el DOM. En este punto, puedes acceder al DOM y realizar operaciones relacionadas con él.
3. Actualización (Updating):
   - beforeUpdate: Se llama antes de que el componente se actualice debido a cambios en las propiedades reactivas.
   - updated: Se llama después de que el componente ha sido actualizado en el DOM.
4. Destrucción (Destruction):
   - beforeUnmount: Se llama justo antes de que el componente sea destruido.
   - unmounted: Se llama después de que el componente ha sido destruido. En este punto, puedes realizar tareas de limpieza, como eliminar event listeners o cancelar solicitudes pendientes.

Aquí hay un ejemplo de cómo utilizar los hooks del ciclo de vida en un componente Vue.js:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vue.js Lifecycle Hooks Example</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
</head>
<body>
    <div id="app">
        <h1>Counter: {{ counter }}</h1>
        <button @click="incrementCounter">Increment Counter</button>
    </div>
    <script>
        const app = {
            data() {
                return {
                    counter: 0
                }
            },
            methods: {
                incrementCounter() {
                    this.counter++;
                }
            },
            beforeCreate() {
                console.log('beforeCreate: Component is being created');
            },
            created() {
                console.log('created: Component has been created');
            },
            beforeMount() {
                console.log('beforeMount: Component is about to be mounted');
            },
            mounted() {
                console.log('mounted: Component has been mounted');
            },
            beforeUpdate() {
                console.log('beforeUpdate: Component is about to be updated');
            },
            updated() {
                console.log('updated: Component has been updated');
            },
            beforeUnmount() {
                console.log('beforeUnmount: Component is about to be destroyed');
            },
            unmounted() {
                console.log('unmounted: Component has been destroyed');
            }
        }
        Vue.createApp(app).mount('#app');
    </script>
</body>
</html>
```

[Vue playground](https://vuejs.org/guide/extras/playground.html#introduction)
