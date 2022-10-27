---
title: Create a Blank Django Project
slug: create-a-blank-django-project

publish_timestamp: Oct. 12, 2020
url: https://www.codingforentrepreneurs.com/blog/create-a-blank-django-project/

---


#### A how-to guide for creating a Django project for use in both Local & Production (deployment) environments.
----------
_Guide Updated for [pipenv](https://www.codingforentrepreneurs.com/blog/pipenv-virtual-environments-for-python) & [Django 3.1](https://www.djangoproject.com/download/)_

> This guide assumes your machine is [setup](/t/setup). If not, go to [Windows installation](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows/) or [Mac/Linux installation](https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux/)

> `cd ~/` is a mac/linux command, Windows users just use `cd \`

> Reference Code is [here](https://github.com/codingforentrepreneurs/CFE-Blank-Project).

## 1. Create Dev Directory for General Project Storage
```console
cd ~/
mkdir Dev
cd Dev
```

## 2. Create Project Directory
We'll call our project "cfehome"
```console
cd ~/Dev
mkdir cfehome
cd cfehome
mkdir src
```
`src` is where your django project will be stored.

## 3. Create Virtual Environment with [Pipenv](https://www.codingforentrepreneurs.com/blog/pipenv-virtual-environments-for-python)
```console
cd src
pipenv install django==2.2 --python 3.6
```
`Django 2.2` is the [current LTS version](https://www.djangoproject.com/download/) which is why we've chosen it here. LTS means Long Term Support release. It will be supported until 2022.


## 4. Activate Pipenv Shell
```console
cd ~/Dev/cfehome/src
pipenv shell
```

## 5. Create Django Project
```console
django-admin startproject cfehome .
```
The trailing `.` is very important here.

## 6. Create New Settings Module
The purpose of this is to work in both production and local environments. It's also called [Staging Django](https://www.codingforentrepreneurs.com/blog/staging-django-production-development).

##### 6.1. Create Settings Directory
```console
cd ~/Dev/cfehome/src/
pipenv shell
cd cfehome
mkdir settings && cd settings
```

##### 6.2. Create `__init__.py` in the new Settings folder to make it a module
```console
echo "from .base import *

from .production import *

try:
   from .local import *
except:
   pass
" > __init__.py
```
> For reference, this was done in `~/Dev/cfehome/src/cfehome/settings/ ` on mac/linux or  ` \Users\YourName\Dev\cfehome\src\cfehome\settings\` on windows.

##### 6.3.  Change `BASE_DIR` in `settings.py`:

**Django > 3**
```python
BASE_DIR = Path(__file__).resolve().parent.parent
# to
BASE_DIR = Path(__file__).resolve().parent.parent.parent
```

**Django< 3**
```python
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
# to
BASE_DIR = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
```

##### 6.4. Move default `settings.py` into new `settings` module and rename `settings.py` to `base.py`:

```console
# mac/linux
mv ~/Dev/cfehome/src/cfehome/settings.py ~/Dev/cfehome/src/cfehome/settings/base.py

# windows
move \Users\YourName\Dev\cfehome\src\cfehome\settings.py \Users\YourName\Dev\cfehome\src\settings\base.py
```

##### 6.5. Copy Local settings (local.py) to make new (base.py & production.py) file:
```console
# mac/linux
cp ~/Dev/cfehome/src/cfehome/settings/base.py ~/Dev/cfehome/src/cfehome/settings/local.py

cp ~/Dev/cfehome/src/cfehome/settings/base.py ~/Dev/cfehome/src/cfehome/settings/production.py

#windows
copy \Users\YourName\Dev\cfehome\src\cfehome\settings\base.py \Users\YourName\Dev\cfehome\src\cfehome\settings\local.py

copy \Users\YourName\Dev\cfehome\src\cfehome\settings\base.py \Users\YourName\Dev\cfehome\src\cfehome\settings\production.py
```

## 7. Some other common installations (optional):
```console
# for a postgresql database
pipenv install psycopg2

# for a mysql database
pipenv install mysqlclient #python3
pipenv install MySQL-python #python2

# for use on heroku
pipenv install gunicorn dj-database-url

pipenv install django-crispy-forms
pipenv install pillow
```

### 8. Run Migration & Createsuperuser
```console
python manage.py migrate
python manage.py createsuperuser
```


All done.


********


### Below is the version using the `virtualenv` package instead of `pipenv`, it's an older method but it still works.

#####  1. Create Dev Directory for General Project Storage
```
cd ~/
mkdir Dev && cd Dev
```


##### 2. Create Virtual Environment
```
cd ~/Dev
mkdir cfehome && cd cfehome
virtualenv -p python3 .

> Python 2 is no longer supported on Django versions 2.2 and up.

# activate on Mac/Linux
source bin/activate

# activate on Windows
.\Scripts\activate
```

##### 3. Install Django & Start Project
```
pip install django==2.2

mkdir src && cd src 

django-admin.py startproject cfehome . 

# Windows (optional)
.\Scripts\django-admin.py startproject cfehome .
```

##### 4. Create New Settings Module
The purpose of this is to create a settings module that works for both local and production use. This is not a requirement but is highly recommended.
 ```
# currently working in 
# ~/Dev/cfehome/src/ on mac/linux
# \Users\YourName\Dev\cfehome\src on windows

cd cfehome
mkdir settings && cd settings
```

Create `__init__.py` within new Settings folder to make it a module. Add the following:
```
# currently working in 
# ~/Dev/cfehome/src/cfehome/settings/ on mac/linux
# \Users\YourName\Dev\cfehome\src\cfehome\settings\ on windows

echo "from .base import *

from .production import *

try:
   from .local import *
except:
   pass
" > __init__.py
```

Change `BASE_DIR` in `settings.py`:


**Django > 3**
```python
BASE_DIR = Path(__file__).resolve().parent.parent
# to
BASE_DIR = Path(__file__).resolve().parent.parent.parent
```

**Django< 3**
```python
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
# to
BASE_DIR = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
```


Move default settings.py into new `settings` module and rename `settings.py` to `base.py`:

```
# mac/linux
mv ~/Dev/cfehome/src/cfehome/settings.py ~/Dev/cfehome/src/cfehome/settings/base.py

# windows
move \Users\YourName\Dev\cfehome\src\cfehome\settings.py \Users\YourName\Dev\cfehome\src\settings\base.py
```

Copy Local settings (local.py) to make new (base.py & production.py) file:
```
# mac/linux
cp ~/Dev/cfehome/src/cfehome/settings/base.py ~/Dev/cfehome/src/cfehome/settings/local.py

cp ~/Dev/cfehome/src/cfehome/settings/base.py ~/Dev/cfehome/src/cfehome/settings/production.py

# windows
copy \Users\YourName\Dev\cfehome\src\cfehome\settings\base.py \Users\YourName\Dev\cfehome\src\cfehome\settings\local.py

copy \Users\YourName\Dev\cfehome\src\cfehome\settings\base.py \Users\YourName\Dev\cfehome\src\cfehome\settings\production.py
```

##### 5. Some other common installations (optional):
```
# for a postgresql database
pip install psycopg2

# for a mysql database
pip install mysqlclient #python3
pip install MySQL-python #python2

# for use on heroku
pip install gunicorn dj-database-url

pip install django-crispy-forms
pip install pillow
```

##### 6. Create `requirements.txt` file
```
pip freeze > requirements.txt
```

##### 7. Run Migration & Createsuperuser
```
python manage.py migrate
python manage.py createsuperuser
```


That's it.

<iframe width="560" height="315" src="https://www.youtube.com/embed/pDQDvV4QY7I?rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
