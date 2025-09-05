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

### Actividad 1

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

### Actividad 2

Python

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

Bash

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

### Actividad 3

```html
# Archivo: web/templates/home.html
```

```html
# Archivo: web/templates/layout.html
```

```python
def create_app()
  return render_template('home.html') # En home()
```

### Actividad 4

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

### Actividad 5

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
# Archivo: templates/error.html {% extends "layout.html" %} {% block title
%}Error{% endblock %} {% block content %}
```

### Fuentes

[Poetry](https://python-poetry.org/docs/)
[Flask](https://flask.palletsprojects.com/en/2.3.x/)
