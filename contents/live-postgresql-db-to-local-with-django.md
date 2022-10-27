---
title: Backup &amp; Load Live Heroku PostgreSQL Database to Local Django Project
slug: live-postgresql-db-to-local-with-django

publish_timestamp: March 21, 2019
url: https://www.codingforentrepreneurs.com/blog/live-postgresql-db-to-local-with-django/

---


I've used [Django fixtures](https://docs.djangoproject.com/en/2.1/howto/initial-data/) a good amount to ensure my local database had at least some of the same data as my live database.

The problem is, of course, fixtures are not the same as your actual database.  They are abstractions.

So what we want instead is a fairly automated way to:
1. Backup our database (good idea anyways)
2. Download the backup (even better)
3. Load in the backup

Personally, I want Django to handle the above three steps for me but you can definitely use this method on any Python application.


**This post assumes you are using**
- Django on Heroku and Local Development
- PostgreSQL on Heroku and Local Development


### 1. Backup and Download PostgreSQL Database via Heroku

**Trigger Backup**
This tells heroku to backup your project's database. Append `--app` if you're not in the current app's directory.
```
heroku pg:backups:capture
```


**Trigger Download**

This downloads your database. 
```
heroku pg:backups:download -o path/to/a/git/ignored/folder/latest.dump
```

`-o` means output path including a filename (ours is `latest.dump`)

**Update `.gitignore`**

As you might imagine, you don't want to push your database dump anywhere. Therefore, update your gitignore file to remove the dump.
```
*.dump
```


### 2. Load PostgreSQL Backup into your Local PostgreSQL Database.

Again, I'll assume you have PostgreSQL installed locally. I'll assume you have it setup in Django correctly.


My *development/local* Django settings module has the following settings:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'myprojectdb',
        'USER': 'myprojectuser',
        'PASSWORD': 'myprojectuserpasswrod',
        'HOST': 'localhost',
        'PORT': 5432,
    }
}
```

Cool. Now I can run the next command:

```
pg_restore --verbose --clean --no-acl --no-owner -h localhost -U myprojectuser -d myprojectdb path/to/a/git/ignored/folder/latest.dump
```

This will load in your database. But I want this a bit more automated.



### 3. Automated Commands For Backup, Download and Load Locally


**Default Imports**
In my Django configuration directory, I'll create a file called `backup_utils.py`. 

```python
import os
import subprocess
import sys
from django.conf import settings
```


**Run Shell Command within Python**
```python
def run_shell_command(command):
    proc_list = []
    proc = subprocess.Popen(command, 
        shell=True, 
        stdin=sys.stdin, 
        stdout=subprocess.PIPE, 
        stderr=sys.stderr) 
    stdout = proc.communicate()[0]
    proc_list.append(proc)
    proc.wait(timeout=500)
    return 
```

**Backup & Download**
```python
def backup_and_download_live_db():
    print("Tapping heroku for live db...")
    backups_dir = os.path.join(settings.BASE_DIR, "backups")
    os.makedirs(backups_dir, exist_ok=True)
    path = os.path.join(settings.BASE_DIR, "backups", 'latest.dump')
    proc_list = []
    # chained command for backup and downloading 
    command = f'heroku pg:backups:capture; heroku pg:backups:download -o {path}' 
    command_done = run_shell_command(command)
    return "Done"
```


**Backup & Download**
```python
def load_in_local_backup_db():
    print("Loading previous database.")
    path = os.path.join(settings.BASE_DIR, "backups", 'latest.dump')
    if not os.path.exists(path):
        backup_and_download_live_db()
        print("Backup didn't exist. Run again after it does.")
        return
    command = f'pg_restore --verbose --clean --no-acl --no-owner -h localhost -U myprojectuser -d myprojectdb {path}' 
    command_done = run_shell_command(command)
    return "Done"
```


Cool. Now we have a couple commands that we can re-use, let's add a Django management command.



### 4. Custom Django Management Command
Inside any of your `INSTALLED_APPS` we'll add a management command. I'll call it `load_live_db.py` so I can call `python manage.py load_live_db`

Your app should look something like this...
```
polls/
    __init__.py
    models.py
    management/
        __init__.py
        commands/
            __init__.py
            load_live_db.py
    tests.py
    views.py
```

```python
from django.core.management.base import BaseCommand

from myproject.backup_utils import (
    backup_and_download_live_db, 
    load_in_local_backup_db
)


class Command(BaseCommand):
    help = 'Grab ask fixture data from heroku'

    def add_arguments(self, parser):
        parser.add_argument(
            '--ignore-download',
            action='store_true',
            dest='ignore-download',
        )
        parser.add_argument(
            '--ignore-load-in',
            action='store_true',
            dest='ignore-load-in',
        )

    def handle(self, *args, **options):
        print("Backing up live db.")
        if not options.get('ignore-download'):
            backup_and_download_live_db()
        if not options.get('ignore-load-in'):
            load_in_local_backup_db()
```

Boom. Now you can load `python manage.py load_live_db` and your local database will be updated to what's in your live database.

I do not recommend doing this in reverse (ie pushing a local database to a live database) unless you absoultely know what you're doing. If the above is all 100% new to you, I'd argue you probably aren't ready to push a local database to a live one.


What do you think? I've found this very useful for my Heroku projects.

Cheers!
