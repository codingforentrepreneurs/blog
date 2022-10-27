---
title: Staging Django for Production &amp; Local Development
slug: staging-django-production-development

publish_timestamp: May 6, 2018
url: https://www.codingforentrepreneurs.com/blog/staging-django-production-development/

---

## How do you have different settings files for production and local development?

This post is an inspiration of two sources: Jan Hesters (a member of the CFE community) and an awesome [presentation](http://lanyrd.com/sgbwt/) by Jacob Kaplan-Moss (a core contributor to Django).

There are a few ways it can be done, and thanks to Jacob, I'll show you the best practice.

Most of my Django projects are [created](https://www.codingforentrepreneurs.com/blog/create-a-blank-django-project/) to look something like this:

```
src/
    accounts/
    cfehome/
        wsgi.py
        urls.py
        settings/
            __init__.py
            base.py
            production.py
            local.py
    manage.py
    requirements.txt
    runtime.txt
```


- `cfehome/` is the name of my Django project in this example.
- `production.py` is the settings configuration for a production server.
- `local.py` is the settings configuration for a development server.
- `base.py` is a duplicate of `local.py` so other devs can use it on their machines. I tend to not import from `base.py` in my other settings modules as I've found this leads to messiness. 


In `src/cfehome/settings/__init__.py` you'll have:

```
from .base import *
from .production import *
```

This means, that the default `python manage.py runserver` will run with your `production.py` configuration by default. `base.py` is imported to include any accidental missing settings but `production.py` will overwrite any production-needed ones.


#### Running Django with a Different Settings Module
This is surprisingly very simple and super effective. Using my above project archticture, you can:

```
python manage.py runserver --settings:cfehome.settings.local
```
Boom! Now you can work off your local settings model. 


```
python manage.py test --settings:cfehome.settings.local
```

This absolutely can be an easier way to run your settings & configuration... and is probably the *best* way for many cases.

#### In Production
You'll likely have something like [gunicorn](http://gunicorn.org/) running your wsgi process. So you'll do something like:
```
gunicorn cfehome.wsgi
```
You can also update `manage.py` to use:
```
s.environ.setdefault("DJANGO_SETTINGS_MODULE", "cfehome.settings.production")
```
But that's not needed because the default `DJANGO_SETTINGS_MODULE` will use `production.py` as your default based on the imports in `__init__.py`.

#### An alternative

This is an alternative way to run a local settings module. This is how I've done it for a long time for a few reasons I'll get into below.

In `src/cfehome/settings/__init__.py` you'll have:

```
from .base import *
from .production import *

try:
    from .local import *
except:
    pass
```

Now, what this does is overwrite `production.py` if `local.py` is present. That means you'll have to update your `.gitignore` file to have `local.py` included so you're not pushing that to a live server ever.

Why do I do it this way?

Simple. I have custom deployment [commands](https://docs.djangoproject.com/en/2.0/howto/custom-management-commands/#management-commands-and-locales):
```
python manage.py push
python manage.py deploy
```

Those two commands do several things for me: 
    - push to storage repo (on bitbucket) 
    - deploy to production repo (on heroku)
    - check git for needed commits and prevent pushing if needed
    - run all [unit tests](https://www.codingforentrepreneurs.com/projects/django-tests-unleashed/) prior to pushing and **stopping** the deploy if tests fail (this can be a huggge pain saver)
    - Update, add, and commit all locally added requirements. (small but another huge pain saver)

That's why I choose to use this method. It makes things a little more complicated to setup when you're working with a team of developers but the time it takes to automate the above tasks is well worth it.