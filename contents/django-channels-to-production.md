---
title: Django Channels 2.0 to Production Environment
slug: django-channels-to-production

publish_timestamp: May 15, 2018
url: https://www.codingforentrepreneurs.com/blog/django-channels-to-production/

---


Learn how to deploy a Django Channels project into production. 

#### Entire Project [Code](https://github.com/codingforentrepreneurs/ChatXChannels)
#### Course: [Chat x Channels](https://www.codingforentrepreneurs.com/courses/chat-channels-react/)

**Requirements**
- Django 2.0.5 or Django 1.11
- Channels 2.1.1
- Python 3.5 (and up)
- Heroku.com Account (likely paid)

**Resources**

- Our code is [here](https://github.com/codingforentrepreneurs/ChatXChannels)

- Learn how to build this with the course [Chat x Channels](https://www.codingforentrepreneurs.com/courses/chat-channels-react/)

- Go Live on Heroku with a non-channels Django project with [this post](https://www.codingforentrepreneurs.com/blog/go-live-with-django-project-and-heroku/)

- How to Stage Django for [production](https://www.codingforentrepreneurs.com/blog/staging-django-production-development/)

- [Create a Blank Django Project](https://www.codingforentrepreneurs.com/blog/create-a-blank-django-project/)
- [Redis Setup](https://www.codingforentrepreneurs.com/blog/celery-redis-django/)

Let's get started
============
### 1. Clone our project
Below will take you to the commit this guide is in reference to (the deployment ready code from [this course](https://www.codingforentrepreneurs.com/courses/chat-channels-react/))
```
$ cd path/to/your/dev/folder
$ mkdir channels
$ cd channels
$ git clone https://github.com/codingforentrepreneurs/ChatXChannels .
$ git reset 4763738e0080c878ac5351dd92f39c774615af40 --hard
$ git remote remove origin
$ virtualenv -p python3 .
$ source bin/activate
(channels) $ pip install -r requirements.txt
```

### 2. Prepare for Heroku [in-depth guide](https://www.codingforentrepreneurs.com/blog/go-live-with-django-project-and-heroku/)
**Create App**
```
(channels) $ cd src
(channels) $ ls
cfehome     chat        db.sqlite3  manage.py   templates
(channels) $ git init
(channels) $ heroku create cfe-channels
```
**Provision Addons**
Create your PostgreSQL database and Redis datastore.
```
(channels) $ heroku addons:create heroku-postgresql:hobby-dev
(channels) $ heroku addons:create heroku-redis:hobby-dev
```

**Add Environment Variables**
```
(channels) $ heroku config:set SECRET_KEY=<your-django-secret-key>
```

- `<your-django-secret-key>`: This is the `SECRET_KEY` in your `settings.py` module


**Create Procfile** with contents:
```
web: daphne cfehome.asgi:application --port $PORT --bind 0.0.0.0
```

- `cfehome.asgi:channel_layer` we'll create this below

- `daphne` is an alternative to `gunicorn`


### 3. Create Production Settings
For me, I'll put it in `cfehome/production.py` next to `settings.py`. To learn how tostage for production, [go here](https://www.codingforentrepreneurs.com/blog/staging-django-production-development/)

```
import os
from urllib.parse import urlparse


DEBUG = False
ALLOWED_HOSTS = ['cfe-channels.herokuapp.com']
SECRET_KEY = os.environ.get('SECRET_KEY', 'SOME+RANDOM+BACKUPKEz9+3vnmjb0u@&w68t#5_e8s9-lbfhv-')  


DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
import dj_database_url
db_from_env = dj_database_url.config()
DATABASES['default'].update(db_from_env)
DATABASES['default']['CONN_MAX_AGE'] = 500
```

##### Include Channels Specific Settings
```

## Channels Specific
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [os.environ.get('REDIS_URL', 'redis://localhost:6379')],
        },
        "symmetric_encryption_keys": [SECRET_KEY],
    },
}
ASGI_APPLICATION = 'cfehome.routing.application'
```

- `ASGI_APPLICATION`: Location of where your Channel's root routing is. 
- `ALLOWED_HOSTS`: Change to include your Heroku project's name


### 4. Daphne
Daphne is by Django to handle running Channels. This is a requirement in order to make your project to run live. Daphne replaces gunicorn for Channels projects.

##### Create `asgi.py` next to `wsgi.py`

Current path: `cfehome/asgi.py`
```
"""
ASGI entrypoint. Configures Django and then runs the application
defined in the ASGI_APPLICATION setting.
"""

import os
import django
from channels.routing import get_default_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "cfehome.settings")
django.setup()
application = get_default_application()
```

##### Local test
Open up terminal
```
$ cd path/to/your/dev/folder/channels/
$ source bin/activate
(channels) $ cd src
(channels) $ daphne cfehome.asgi:application
```
If errors occur, there's something not setup correctly


### 5. Requirements, Runtime, `.gitignore`
```
(channels) $ ls
cfehome     chat        db.sqlite3  manage.py   templates Procile
(channels) $ pip freeze > requirements.txt
(channels) $ echo "python-3.6.5" > runtime.txt
```
Copy the contents of [this .gitignore](https://raw.githubusercontent.com/codingforentrepreneurs/Try-Django-1.11/master/.gitignore) and paste them into `.gitignore`
```
(channels) $ ls -a
.       .gitignore  chat        manage.py
..      cfehome     db.sqlite3  templates
```


### 6. Disable Collectstatic and use [S3 for static files](https://www.codingforentrepreneurs.com/blog/s3-static-media-files-for-django/)
```
(channels) $ heroku config:set DISABLE_COLLECTSTATIC=1
```

### 7. Commit Production-ready Code
```
(channels) $ git add --all
(channels) $ git commit -m "Prepare for production"
```

### 8. Push
```
(channels) $ git push heroku master && heroku run python manage.py migrate
```

### 9. Scale Dynos
```
(channels) $  heroku ps:scale web=1
```

### 10. Additional Options are [in this guide](https://www.codingforentrepreneurs.com/blog/go-live-with-django-project-and-heroku/)
