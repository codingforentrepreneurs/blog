---
title: Celery + Redis + Django
slug: celery-redis-django

publish_timestamp: Nov. 8, 2020
url: https://www.codingforentrepreneurs.com/blog/celery-redis-django/

---

*Celery* is a task queue with focus on real-time processing, while also supporting task scheduling.

*Redis* is a message broker. This means it handles the queue of "messages" between Django and Celery. 

*Django* is a web framework made for perfectionists with deadlines. 

All three work together to make real-time magic. 

## Learn Celery x Redis x Django on [Time & Tasks 2](https://www.codingforentrepreneurs.com/projects/time-tasks-2).
============

#### Do you have a Django Project? If not, use [this](http://kirr.co/itznm2/).
[This post](http://kirr.co/itznm2/) shows you how to create a basic Django project.


#### Tech Stack
- Redis
- Celery (v4+)
- Django (v2.2+)
- Python (v3.6+)


#### Assumptions
This project assumes you have a [Django Project](/t/django) already created. If you do not, consider doing [Try Django 2.2](https://www.codingforentrepreneurs.com/projects/try-django-2-2).


#### Install Redis

1. Windows
    - [Blog post](https://www.codingforentrepreneurs.com/blog/redis-on-windows/)
    - [Install Redis via Memurai (video)](https://www.codingforentrepreneurs.com/projects/setup-redis/install-redis-windows-using-memurai)
    - [Install via Docker (video)](https://www.codingforentrepreneurs.com/projects/setup-redis/install-redis-windows-using-memurai)
2. Mac
   - [Blog post](https://www.codingforentrepreneurs.com/blog/install-redis-mac-and-linux)
   - [Install Redis via Homebrew (video)](https://www.codingforentrepreneurs.com/projects/setup-redis/install-redis-macos-using-homebrew)
   - [Install Redis via Docker (video)](https://www.codingforentrepreneurs.com/projects/setup-redis/install-redis-macos-using-docker)

3. Linux
    - [Blog post](https://www.codingforentrepreneurs.com/blog/install-redis-mac-and-linux)
    - Hello Linux Series [first post](https://www.codingforentrepreneurs.com/blog/hello-linux/) or [video tutorial](https://www.codingforentrepreneurs.com/projects/hello-linux)



#### Celery, Redis & Django

##### 1. Create Virtual Environment

```bash
cd path/to/project/
```

```bash
python3 -m venv .
```
> You can use any virtual environment manager you choose (`pipenv`, `virtualenv`, `venv`, `poetry`, etc). `venv` is built in to Python 3+ which is why we use it.

##### 2. Activate Virtual Environment.

**Mac/Linux**
```bash
$ source bin/activate
(yourproject) $
```

**Windows**
```bash
> .\Scripts\activate
(yourproject) >
```

##### 3. Install Celery & Redis Python Packages

```bash
pip install celery==4.4.7
pip install redis
pip install django-celery-beat
pip install django-celery-results
pip freeze > requirements.txt
```
> At the time of this writing, `celery==5.0.2` has been released. This version does not yet support `django-celery-results` so we stick with `celery==4.4.7`


##### 4. Update Django `settings.py`:
```python
INSTALLED_APPS += [
    'django_celery_beat',
    'django_celery_results',
]

CELERY_RESULT_BACKEND = "django-db"
```


##### 5. Run migrations: 
    
```bash
python manage.py makemigrations
python manage.py migrate
```

##### 6. Create `celery.py` to setup `Celery app`:
In this case, my django project is named `cfehome`.

- Navigate to root project config module (where `settings` and `urls` modules are)
- Create a `celery.py` file with the contents:

```python
# yourvenv/cfehome/celery.py
from __future__ import absolute_import, unicode_literals # for python2

import os
from celery import Celery

# set the default Django settings module for the 'celery' program.
# this is also used in manage.py
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'cfehome.settings')


## Get the base REDIS URL, default to redis' default
BASE_REDIS_URL = os.environ.get('REDIS_URL', 'redis://localhost:6379')

app = Celery('cfehome')

# Using a string here means the worker don't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()

app.conf.broker_url = BASE_REDIS_URL

# this allows you to schedule items in the Django admin.
app.conf.beat_scheduler = 'django_celery_beat.schedulers.DatabaseScheduler'
```

##### 7. Update project configuration folder's `__init__.py` file:
It's the `__init__.py` located in the same directory as `settings.py`

```python
# ourvenv/cfehome/__init__.py
# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from .celery import app as celery_app  # noqa
```
   
##### 8. Create `tasks.py` in any Django app (a valid app in `INSTALLED_APPS`):
The `app.autodiscover_tasks()` configuration from above will automatically find these tasks. 

> You may have to restart your worker (more on this below) to ensure these are registered


```python
import random
from celery import shared_task

@shared_task(name="sum_two_numbers")
def add(x, y):
    return x + y

@shared_task(name="multiply_two_numbers")
def mul(x, y):
    total = x * (y * random.randint(3, 100))
    return total

@shared_task(name="sum_list_numbers")
def xsum(numbers):
    return sum(numbers)
```


##### 9. Test tasks:
1. Open a terminal window, and run a **celery worker** with in your project root (where `manage.py` lives). Our project is `cfehome` so change your project name accordingly.

```bash
celery -A yourproject worker -l info
# like 
celery -A cfehome worker -l info
```

2. Open another terminal window, in your Django project `python manage.py shell`:

```python
>>> from yourapp.tasks import add, mul, xsum
>>> add(1,3)
4
>>> add.delay(1,3)
<AsyncResult: 7bb03f9a-5702-4661-b737-2bc54ed9f558>
```
> `delay` is the key here. It offloads this task to the worker process. We discuss this in depth in the [Time & Tasks 2](https://www.codingforentrepreneurs.com/projects/time-tasks-2) series.

If you look at your celery worker, you should see something like:
```bash
[2016-11-08 22:05:22,686: INFO/PoolWorker-5] Task sum_two_numbers[7bb03f9a-5702-4661-b737-2bc54ed9f558] succeeded in 0.0004559689841698855s: 21
```

##### 10. Setup Schedule to Run Tasks [Docs](http://docs.celeryproject.org/en/latest/userguide/periodic-tasks.html):
```python
# yourvirtualenv/cfehome/celery.py

from celery.schedules import crontab
app.conf.beat_schedule = {
    'add-every-minute-contrab': {
        'task': 'multiply_two_numbers',
        'schedule': crontab(hour=7, minute=30, day_of_week=1),
        'args': (16, 16),
    },
    'add-every-5-seconds': {
        'task': 'multiply_two_numbers',
        'schedule': 5.0,
        'args': (16, 16)
    },
    'add-every-30-seconds': {
        'task': 'tasks.add',
        'schedule': 30.0,
        'args': (16, 16)
    },
}
```

##### 11. Run Scheduled Tasks With Celery Beat:
Open another `Terminal` window to run scheduled tasks:
```bash
celery -A cfehome woker --beat -l info -S django
```
- `-S django` tells celery to use the Django database scheduler.

> You can also have the `beat` server run as it's own process with `celery -A cfehome beat -l info`
   

##### 12. 3 Processes running:

**Django**
```
$ python manage.py runserver
```

**Celery Worker & Beat** (this can be 2 processes)
```
# celery worker & beat 
$ celery -A cfehome worker --beat -S django -l info
```

**redis server**
```
$ redis-server
```

   
##### 13. Launching.

Below we'll use Heroku. To learn how to launch `celery`, `redis`, and `django` on virtually any linux machine, watch [this project](https://www.codingforentrepreneurs.com/projects/hello-linux).

> You most likely need a paid Heroku account to run this.

1. Setup Procfile:
```
web: gunicorn cfehome.wsgi --log-file -
worker: celery -A cfehome worker --beat -S django --l info
```

2. Production `settings.py`:
Assuming you follow standard best practices for production Django settings, the settings we set above should work for you.

3. Setup [heroku-redis](https://elements.heroku.com/addons/heroku-redis):

```console
heroku addons:create heroku-redis:hobby-dev
```

4. Upgrade Heroku Account to `Professional`
This is so you can have 2 dynos (processes) running at once.

5. Update `requirements.txt`

```bash
pip freeze > requirements.txt
```

6. Commit, Push & Migrate
```bash
git add --all
git commit -m "Add Celery + Redis to Django"
git push heroku master
heroku run python manage.py migrate
```

7. Scale Up Services:
```bash
heroku ps:scale web=1 worker=1
```

8. Restart Server:
```bash
heroku restart
```