---
title: Celery + Redis + Django
slug: celery-redis-django

publish_timestamp: Aug. 25, 2022
url: https://www.codingforentrepreneurs.com/blog/celery-redis-django/

---


> Updated for Django 4.0 (4.1 is not yet supported by some packages)

**Celery** unlocks a worker process for Django. This means you can offload tasks from the main request/response cycle within Django. Using Celery becomes critical when your app starts to scale or you need better performance out of Django.

**Django** is a batteries included web framework written in Python. Django is how you'll create a web application so users can leverage your software.

**Redis** is the datastore and message broker between Celery and Django. In other words, Django and Celery use Redis to communicate with each other (instead of a SQL database). Redis can also be used as a cache as well. An alternative for Django & Celery is RabbitMQ (not covered here).

All three work together to make some asynchronous magic. Here's a few great use cases for Django + Celery:

- Offloading any long-running tasks
- Sending emails and/or email notifications
- Generating reports that take 3+ seconds to create
- Running Machine Learning training and/or inference
- Run specific functions on a schedule
- Backup a database
- Powering up / powering down additional virtual machines for handling load
- Triggering workflows and/or sending webhook notifications

**Recommended course**: [Time & Tasks 2](https://www.codingforentrepreneurs.com/projects/time-tasks-2). In this course, you'll learn how to implement Celery with Redis and Django leveraging a lot of what this guide has to offer.

============

## Tutorial

### Tech Stack
- Redis
- Celery 5+
- Django 3.2+ (4.1 not currently supported)
- Python 3.8+


### Key Assumptions

- Python 3.8+ installed
- Python experience
- Django experience 

By Django experience, I assume you already know Django fundamentals (views, models, templates, forms, etc). If you do not, I highly recommend watching my courses [Your First Django Project](https://www.codingforentrepreneurs.com/courses/your-first-django-project/) and/or [Try Django 3.2](https://www.codingforentrepreneurs.com/projects/try-django-2-2) .



### Redis Installed
Redis is a critical piece to making this who system works. If you don't have it installed, please consider some of the following resources.

### Install Redis on Windows
- [Blog post](https://www.codingforentrepreneurs.com/blog/redis-on-windows/)
- [Install Redis via Memurai (video)](https://www.codingforentrepreneurs.com/projects/setup-redis/install-redis-windows-using-memurai)
- [Install via Docker (video)](https://www.codingforentrepreneurs.com/projects/setup-redis/install-redis-windows-using-memurai)
### Install Redis on macOS
- [Blog post](https://www.codingforentrepreneurs.com/blog/install-redis-mac-and-linux)
- [Install Redis via Homebrew (video)](https://www.codingforentrepreneurs.com/projects/setup-redis/install-redis-macos-using-homebrew)
- [Install Redis via Docker (video)](https://www.codingforentrepreneurs.com/projects/setup-redis/install-redis-macos-using-docker)

### Install Redis on Linux
- [Blog post](https://www.codingforentrepreneurs.com/blog/install-redis-mac-and-linux)
- Hello Linux [blog series](https://www.codingforentrepreneurs.com/blog/hello-linux/) or [video tutorial](https://www.codingforentrepreneurs.com/projects/hello-linux).

## Getting Started

### 1. Create and Activate your Virtual Environment


*Create with venv*
```bash
cd path/to/project/
```

**Mac/Linux**
```bash
python3 -m venv venv
```

**Windows**
```bash
c:\Python38\python.exe -m venv venv
```

*Activate your virtual environment*

**Mac/Linux**
```bash
$ source venv/bin/activate
(venv) $
```

**Windows**
```bash
.\venv\Scripts\activate
(venv)
```

### 2. Install Requirements

```bash
python -m pip install pip --upgrade
python -m pip install redis django-celery-beat django-celery-results python-decouple
```
At the time of this writing, Django 4.0 is the latest version of Django that supports `django-celery-results`.  

Create `requirements.txt`
```text
Celery
Django
django-celery-beat 
django-celery-results # currently requires Django 4.0 or older
python-decouple
redis
```

Notes on a few of the packages:
- `django-celery-beat` unlocks Django's database to store periodic tasks to be run by celery.
- `django-celery-results` enables to store the results of any given celery task in the Django database
- `python-decouple` a easy to use Environment variables loader
- `redis` (the python package) enables Python to communicate with a Redis server.

### 3. Create a Django Project
```bash
mkdir -p src
cd src
django-admin startproject cfehome .
```

### 4. Create `.env` for Environment variables

In the root of our Django project (where `manage.py` is), we'll add the `.env` file:


`path/to/your/project/src/.env`
```bash
CELERY_BROKER_REDIS_URL="redis://localhost:6380"
DEBUG=True
```
As a reminder, our `src` folder should now contain:

- `.env`
- `cfehome` (our project configuration folder)
- `manage.py` 


### 5. Update `settings.py`

Update `cfehome/settings.py` to the following:


```python
from decouple import config

# SECURITY WARNING: don't run with debug turned on in production!
# DEBUG = True
DEBUG = config("DEBUG", default=0)

...

# add our newly installed packages to INSTALLED_APPS
INSTALLED_APPS = [
    ...
    'django_celery_beat',
    'django_celery_results',
    ...
]

# save Celery task results in Django's database
CELERY_RESULT_BACKEND = "django-db"

# This configures Redis as the datastore between Django + Celery
CELERY_BROKER_URL = config('CELERY_BROKER_REDIS_URL', default='redis://localhost:6379')
# if you out to use os.environ the config is:
# CELERY_BROKER_URL = os.environ.get('CELERY_BROKER_REDIS_URL', 'redis://localhost:6379')


# this allows you to schedule items in the Django admin.
CELERY_BEAT_SCHEDULER = 'django_celery_beat.schedulers.DatabaseScheduler'
```


### 6. Run Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```


### 7. Configure Celery App
This is *almost* identical to the [Celery docs](https://docs.celeryq.dev/en/stable/django/first-steps-with-django.html).

My project is named `cfehome` so we will create `celery.py` in the root project configuration folder (where `settings.py`, `wsgi.py`, and `urls.py` live).

In `cfehome/celery.py`:
```python

# path/to/your/proj/src/cfehome/celery.py
import os
from celery import Celery
from decouple import config

# set the default Django settings module for the 'celery' program.
# this is also used in manage.py
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'cfehome.settings')

app = Celery('cfehome')

# Using a string here means the worker don't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()

# We used CELERY_BROKER_URL in settings.py instead of:
# app.conf.broker_url = ''

# We used CELERY_BEAT_SCHEDULER in settings.py instead of:
# app.conf.beat_scheduler = ''django_celery_beat.schedulers.DatabaseScheduler'
```

### 8. Update Root Config `__init__.py`

In `cfehome/__init__.py` add:

```python
# path/to/your/proj/cfehome/__init__.py
# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from .celery import app as celery_app  # noqa

__all__ = ('celery_app',)
```




### 9. Verify Celery Runs

Open up your command line:

```bash
celery -A cfehome worker --beat
```
You should see something like:


```bash
 -------------- celery@justins-mbp.lan v5.2.7 (dawn-chorus)
--- ***** ----- 
-- ******* ---- macOS-12.5.1-x86_64-i386-64bit 2022-08-26 02:14:43
- *** --- * --- 
- ** ---------- [config]
- ** ---------- .> app:         cfehome:0x110ad7070
- ** ---------- .> transport:   redis://localhost:6380//
- ** ---------- .> results:     
- *** --- * --- .> concurrency: 16 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** ----- 
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery
                
[2022-08-26 02:14:45,435: WARNING/MainProcess] /Users/jmitch/Dev/reco/venv/lib/python3.10/site-packages/celery/fixups/django.py:203: UserWarning: Using settings.DEBUG leads to a memory
            leak, never use this setting in production environments!
  warnings.warn('''Using settings.DEBUG leads to a memory
```

Again the command is:
```bash
celery -A cfehome worker --beat
```
Let's break down the command:
- `celery` is the CLI command
- `-A cfehome` references the django configuration folder `cfehome` where `celery.py` lives.
- `worker` means our task offloader is running
- `--beat` turns this running instance of Celery into a beat process as well (ie runs scheduled items not just offloaded tasks). 

If we just wanted to run a beat worker on it's own we can omit `--beat` above and, in another terminal instance, run:

```bash
celery -A cfehome beat
```
The beat command would yield:
```bash
celery beat v5.2.7 (dawn-chorus) is starting.
__    -    ... __   -        _
LocalTime -> 2022-08-26 02:18:52
Configuration ->
    . broker -> redis://localhost:6380//
    . loader -> celery.loaders.app.AppLoader
    . scheduler -> django_celery_beat.schedulers.DatabaseScheduler

    . logfile -> [stderr]@%WARNING
    . maxinterval -> 5.00 seconds (5s)
```

### 10. Create a working app

```bash
python manage.py startapp movies
```
Update `INSTALLED_APPS` in `cfehome/settings.py`:

```python
INSTALLED_APPS = [
    ...
    'movies',
    ...
]
```
Run migrations:
```bash
python manage.py makemigrations
python manage.py migrate
```

### 11. Add Celery-based Tasks to our Example App

In `movies/tasks.py` add:
```python
import random
from celery import shared_task

@shared_task
def add(x, y):
    # Celery recognizes this as the `movies.tasks.add` task
    # the name is purposefully omitted here.
    return x + y

@shared_task(name="multiply_two_numbers")
def mul(x, y):
    # Celery recognizes this as the `multiple_two_numbers` task
    total = x * (y * random.randint(3, 100))
    return total

@shared_task(name="sum_list_numbers")
def xsum(numbers):
    # Celery recognizes this as the `sum_list_numbers` task
    return sum(numbers)
```

Yes, these tasks are just simple math examples that have little to do with an app called `movies` but help us learn how simple Celery can be.

The reason `tasks.py` is recognized by Celery is because we added `app.autodiscover_tasks()` in the `cfehome/celery.py` configuration file.


After updating or creating a new task in `tasks.py` in any of your `INSTALLED_APPS` you will likely have to restart your worker process.



### 12. Execute our `shared_tasks`

Let's run our worker (`--beat` is optional):

```bash
celery -A cfehome worker -l info --beat
```
Using `-l info` will provide a list of all registered `tasks` as well as more verbose output on tasks run.

Open another terminal window and run:

```bash
cd path/to/your/project
source venv/bin/activate
```
if windows, run `.\venv\Scripts\activate`.

Now jump into the Django-managed python shell.

```bash
python manage.py shell
```

```python
from movies.tasks import add, mul, xsum
add(1,3)
```
Great we see `4`. The function works as expected. Now let's send it to Celery:

```python
add.delay(1,3)
```
This returns `<AsyncResult: 7bb03f9a-5702-4661-b737-2bc54ed9f558>`

`.delay()` is the key here and how we send this task to be run in `celery`. The `AsyncResult` shows us the *task id* for this task. We can now use the Django Admin to review this task's results. In [Time & Tasks 2](https://www.codingforentrepreneurs.com/projects/time-tasks-2), we go into much more depth on how this works.


If you look at your celery worker terminal, you should see something like:
```bash
[2016-11-08 22:05:22,686: INFO/PoolWorker-5] Task sum_two_numbers[7bb03f9a-5702-4661-b737-2bc54ed9f558] succeeded in 0.0004559689841698855s: 4
```
This shows us that the task `7bb03f9a-5702-4661-b737-2bc54ed9f558` succeded in `0.0004559689841698855s` with the printed statement of `4` which, of course, is the result of `add.delay(1, 3)`



### 13. Hard-code Scheduled Tasks
Periodic Task [Reference Docs](http://docs.celeryproject.org/en/latest/userguide/periodic-tasks.html)

Opening `cfehome/celery.py`, we can add a hard-coded scheduler for our `--beat` server.

```python
# path/to/your/proj/src/cfehome/celery.py

from celery.schedules import crontab

# Below is for illustration purposes. We 
# configured so we can adjust scheduling 
# in the Django admin to manage all 
# Periodic Tasks like below
app.conf.beat_schedule = {
    'multiply-task-crontab': {
        'task': 'multiply_two_numbers',
        'schedule': crontab(hour=7, minute=30, day_of_week=1),
        'args': (16, 16),
    },
    'multiply-every-5-seconds': {
        'task': 'multiply_two_numbers',
        'schedule': 5.0,
        'args': (16, 16)
    },
    'add-every-30-seconds': {
        'task': 'movies.tasks.add',
        'schedule': 30.0,
        'args': (16, 16)
    },
}
```

Let's break this down:

- `multiply-task-crontab`  and `multiply-every-5-seconds` are an examples of a schedule's name that I made up. You can do the same.
- `'task': 'multiply_two_numbers',` is an example of using a named task. In this case, it's `@shared_task(name="multiply_two_numbers")` from `movies/tasks.py`
- `crontab(hour=7, minute=30, day_of_week=1)` crontab is a cron-like syntax for celery
- `'schedule': 5.0,` defaults to seconds
- `args` are the arguments (if any) that you want to pass to the task's function
- Using `app.conf.beat_schedule ` along with the `CELERY_BEAT_SCHEDULER` set to the `DatabaseScheduler`, will automatically add this schedule into the Django database which you can manage in the Django admin.


### 14. Run Worker & Beat Server

```bash
celery -A cfehome woker --beat -l info
```


### 15. At least 3 Processes Running (at least)

**Django's Run Server**
```bash
python manage.py runserver
```
In production, this would more likely be gunicorn with `gunicorn cfehome.wsgi`.

**Celery Worker & Beat** (this can be 2 processes)
```bash
celery -A cfehome worker --beat -l info
```
In production, you might need to make the *celery beat scheduler* it's own running instance `celery -A cfehome beat -l info` and then `celery -A cfehome worker -l info`

**Redis server**
Naturally, we need a redis server running:
```bash
redis-server
```


### 16. Launching to Production

To deploy a Celery, Redis, and Django in production, I recommend you read the following **Hello Linux** blog series:

- [Git Setup for Push/Pulling Code From Local to Production](https://www.codingforentrepreneurs.com/blog/git-push-local-code-to-live-linux-server)
- [Working Directory with a Virtual Environment](https://www.codingforentrepreneurs.com/blog/hello-linux-virtual-environment-working-directory)
- [PostgreSQL on Live Linux Server](https://www.codingforentrepreneurs.com/blog/hello-linux-postgresql-on-live-linux-server)
- [Setup Gunicorn & Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-setup-gunicorn-and-supervisor/) 
- [Nginx & UFW Firewall](https://www.codingforentrepreneurs.com/blog/hello-linux-nginx-and-ufw-firewall)
- [Custom Domain + HTTPs with Let's Encrypt](https://www.codingforentrepreneurs.com/blog/custom-domain-and-https-with-lets-encrypt/)
- [Celery, Redis, & Supervisor](https://www.codingforentrepreneurs.com/blog/hello-linux-celery-supervisor/)

We also have a [Hello Linux](https://www.codingforentrepreneurs.com/courses/hello-linux/) course that covers the above blogs.
