---
title: Use Django in Jupyter
slug: use-django-in-jupyter

publish_timestamp: Oct. 7, 2020
url: https://www.codingforentrepreneurs.com/blog/use-django-in-jupyter/

---


Below is a simple method to use a Django project inside Jupyter's standard running (ie `jupyter notebook`) instead of using a Django-managed jupyter server (ie `python manage.py shell_plus --notebook`).

It's true packages exist to make it "easy" to use Django inside of a Jupyter notebook. I seem to always run into issues successfully running these packages. I've found the below method useful although I cannot recall how I discovered how this works (aka attribution needed).

In the long run, use [this gist](https://gist.github.com/codingforentrepreneurs/76e570d759f83d690bf36a8a8fa4cfbe) as I'll be able the code there up to date.

## `django_for_jupyter.py`
```python
import os, sys
PWD = os.getenv('PWD')

PROJ_MISSING_MSG = """Set an enviroment variable:\n
`DJANGO_PROJECT=your_project_name`\n
or call:\n
`init_django(your_project_name)`
"""

def init_django(project_name=None):
    os.chdir(PWD)
    project_name = project_name or os.environ.get('DJANGO_PROJECT') or None
    if project_name == None:
        raise Exception(PROJ_MISSING_MSG)
    sys.path.insert(0, os.getenv('PWD'))
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', f'{project_name}.settings')
    os.environ["DJANGO_ALLOW_ASYNC_UNSAFE"] = "true"
    import django
    django.setup()
```


## Requirements
- Virtual Environment (`virtualenv`, `venv`, `pipenv`, etc)
- Django installed & project created (we'll use the project name `cfehome`)
- Jupyter installed at least in the virtual environment


## Setup:
It's simple. Just copy `django_for_jupyter.py` next to your Jupyter notebook files (`.ipynb`).


### `init_django` Via Project Name
*First cell*
```python
from django_for_jupyter import init_django
init_django("cfehome")
````
> Change `cfehome` to your Django project name.

### `init_django` Using Environment Variables
In, `.env`:
```
DJANGO_PROJECT="cfehome"
```
Or on the CLI:
```
DJANGO_PROJECT="cfehome" jupyter notebook
```

*First cell*
```python
from django_for_jupyter import init_django
init_django()
```
> Change `cfehome` to your Django project name.


## Using Django Models
Now that you've run `init_django` with no errors. You can import Django models:

*Second Cell*
```python
from myapp.models import MyModel

MyModel.objects.all()
```

Simple enough right?
