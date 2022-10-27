---
title: Prepare Django for Digital Ocean App Platform
slug: prepare-django-for-digital-ocean-app-platform

publish_timestamp: July 27, 2021
url: https://www.codingforentrepreneurs.com/blog/prepare-django-for-digital-ocean-app-platform/

---


This post is a reference guide for the [Try Django 3.2](https://www.codingforentrepreneurs.com/projects/try-django-3-2) series. 


## Step 1. Clone Project with Git
```
mkdir try-djangoP
cd try-django
git clone --depth 1 -b 31-start https://github.com/codingforentrepreneurs/Try-Django-3.2 .
```
> We use branch `31-start` (`-b 31-start`) for this guide to ensure we're all using the same code. Modify it as you need.

## Step 2. Update `requirements.txt`
Update the file `requirements.txt` contain at least:
```
django>=3.2,<3.3
gunicorn
psycopg2-binary
```

## Step 3. Create `runtime.txt`
Create a file named `runtime.txt` right next to `manage.py` with:
```
python-3.6.13
```

## Step 4. Update `settings.py`
In `trydjango/settings.py` let's update a few configuration items for production environment variables including:

- `DEBUG`
- `DJANGO_ALLOWED_HOST`
- `DJANGO_SECRET_KEY`
- `POSTGRES_DB`
- `POSTGRES_HOST`
- `POSTGRES_USERNAME`
- `POSTGRES_PASSWORD`
- `POSTGRES_PORT`

### 1. Import `os`
We'll be using the python `os` package for the environment variables from above.
```python
import os
import pathlib
```
> Django 3.2 does no longer uses `import os` on the settings module by default.

### 2. Update Debug
```python
DEBUG=str(os.environ.get('DEBUG'))=='1'
```

### 3. Update `ALLOWED_HOSTS`
```python
ENV_ALLOWED_HOST = os.environ.get('DJANGO_ALLOWED_HOST') or None
ALLOWED_HOSTS = []
if ENV_ALLOWED_HOST is not None:
    ALLOWED_HOSTS = [ ENV_ALLOWED_HOST ]
```

### 4. Update `DJANGO_SECRET_KEY`
```python
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY') or 'j8go-s#xppcsm%$p@%q5we7u)l^%*z34ulo!o0-h%mx8%@q1-o'
```
> Use `python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'` from [this post](https://www.codingforentrepreneurs.com/blog/create-a-one-off-django-secret-key/).



### 5. Add PostgreSQL Settings
Just after the default `DATABASE` configuration add this:

```python
POSTGRES_DB = os.environ.get("POSTGRES_DB")
POSTGRES_PASSWORD = os.environ.get("POSTGRES_PASSWORD")
POSTGRES_USER = os.environ.get("POSTGRES_USER")
POSTGRES_HOST = os.environ.get("POSTGRES_HOST")
POSTGRES_PORT = os.environ.get("POSTGRES_PORT")

POSTGRES_READY = (
    POSTGRES_DB is not None
    and POSTGRES_PASSWORD is not None
    and POSTGRES_USER is not None
    and POSTGRES_HOST is not None
    and POSTGRES_PORT is not None
)

if POSTGRES_READY:
    DATABASES = {
        "default": {
            "ENGINE": "django.db.backends.postgresql",
            "NAME": POSTGRES_DB,
            "USER": POSTGRES_USER,
            "PASSWORD": POSTGRES_PASSWORD,
            "HOST": POSTGRES_HOST,
            "PORT": POSTGRES_PORT,
        }
    }
```


## Next step: Deploy
The above configuration is already in the branch `production-1` on [https://github.com/codingforentrepreneurs/Try-Django-3.2](https://github.com/codingforentrepreneurs/Try-Django-3.2) so we'll actually use that branch to deploy. Go to the [Deploy Django on App Platform](https://www.codingforentrepreneurs.com/blog/deploy-django-to-digitalocean-app-platform) post for details on how to do this.
