---
title: Django Static Files in Production on DigitalOcean Spaces
slug: django-static-files-digitalocean-spaces

publish_timestamp: Jan. 21, 2022
url: https://www.codingforentrepreneurs.com/blog/django-static-files-digitalocean-spaces/

---

In this post, we'll use DigitalOcean Spaces for hosting our Django Static Files in production. DO Spaces is an S3-Compatible Cloud Object Storage that features a built-in Content Delivery Network (CDN). 

Spaces and the built-in CDN maximizes performance globally and allows for virtually unlimited amount of items stored.


Django should not serve Static Files in production. Static Files include JavaScript (JS), Cascading Style Sheets (CSS), PDFs, Images, Videos, and virtually anything else that's not serving direct HTTP Requests.

DigitalOcean Spaces are perfectly suited for a low-cost method to serve your Django static files in production as well as for development use.

Are you new to Django? Be sure to get some experience first like watching [this series](https://www.youtube.com/watch?v=SlHBNXW1rTk&list=PLEsfXFp6DpzRMby_cSoWTFw8zaMdTEXgL).

> This guide is a continuation/reference in the series [Try Django 3.2](/projects/try-django-3-2) but can be used for nearly any Django Project (version 3.0 & up). 


### 1. Sign up for DigitalOcean [here](https://do.co/cfe-sh) (includes a $100 promo).

### 2. Setup a DigitalOcean Space
Spaces are charged at $5/mo for 250/gb of storage + 1tb of outbound. Review more details [here](https://www.digitalocean.com/products/spaces/).

DO Spaces is an S3-Compatible cloud storage which means it's a drop-in replace for Amazon Web Services (AWS) S3. 


#### 1. Sign up for DigitalOcean [here](https://do.co/cfe-sh) (includes a $100 credit for 60 days).
#### 2. Navigate to [Spaces](https://cloud.digitalocean.com/spaces) in your console
#### 3. Click `Create` > `Spaces` or go to [this link](https://cloud.digitalocean.com/spaces/new)
#### 4. Choose a datacenter region near you (or your users). I am in Texas, so I chose `New York 3`
#### 5. *CDN*: Enable if you have a Custom Domain you can use. 
I also recommend picking Let's Encrypt for the certificate if you go this route (it's easier to manage).
#### 6. *Allow file listing*: Select `Restrict File Listing` as you likely do not need this open to the world.
#### 7. *Choose a unique name*: 
In my case, I'll use `cfe-dj`. (You should make note of *Your Space's origin URL* although you can get this later too)
#### 8. Select an existing project to attach this space to (the default one is fine)
#### 9. Select `Create a Space`

After a few seconds (maybe a few minutes), your space will be created. 

#### 10. Under your space, click on `Settings` and navigate to **Endpoint**. My endpoint is:

```
nyc3.digitaloceanspaces.com
```
Yours will be set based on what region you choose. We'll use this endpoint again shortly.


### 3. Create DigitalOcean API Keys


#### 1. Navigate to [Applications & API Token/keys](https://cloud.digitalocean.com/account/api/tokens)
#### 2. Scroll to **Spaces access keys** near the bottom
#### 3. Click `Generate New Key`
#### 4. Give it a name, I used: `TryDjangoDev`. Select the checkmark (or hit enter/return)
#### 5. Make note of the two keys that appear. 
The one on top is the `PUBLIC KEY` the one on the bottom and the longer one is the `PRIVATE KEY`. The `PRIVATE KEY` will be hidden you navigate away. My keys look like this:

```
PUBLIC_KEY=7DE4MPNH2TNLX6Z7HIL4
PRIVATE_KEY=6JCDRA+hZ8qBDdPk1P4qgrBqQTXTLF276pov1+EZ5go
```
You can always `Regenerate` your `PRIVATE_KEY` under `More`.

#### 6. Repeat steps 1-5 for another set of keys called **TryDjangoProd**.

#### 7. Save your keys:
- For the **TryDjangoDev** keys, I'll be storing these in my local `.env` file.
- For the **TryDjangoProd** keys, I'll be storing these in my production app environment variables.

The environment variables are as follows:

```
AWS_ACCESS_KEY_ID=public_key_id_from_digital_ocean
AWS_SECRET_ACCESS_KEY=private_key_id_from_digital_ocean
```


### 4. Install `django-storages` & `boto3`


```
pip install django-storages boto3
```
- `django-storages` [docs](https://django-storages.readthedocs.io/en/latest/backends/amazon-S3.html) does the heavy lifting to integrate Django with boto3 and DO Spaces
- `boto3` [docs](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) is a python library for managing s3-compatible storage.

> Make sure you're using a virtual environment when installing new packages. If you're new to virtual environments, check out our guide for [Windows](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows) or [macOS](https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux) or [Linux](https://www.codingforentrepreneurs.com/blog/install-django-on-linux-ubuntu/) to laern more.


Update requirements.txt
```
django>=3.2,<3.3
gunicorn
psycopg2
django-dotenv
django-htmx
pint
django-storages
boto3
```
> The only real requirements for production are `django`, `gunicorn`, `psycopg2` (or `psycopg2-binary`), `django-storages`, and `boto3` for this guide.

### 5. Configure Django for DO Spaces & `django-storages`.

`trydjango/cdn/__init__.py`
```python
# blank file
```

`trydjango/cdn/conf.py`
```python
import os
AWS_ACCESS_KEY_ID = os.environ.get("AWS_ACCESS_KEY_ID")
AWS_SECRET_ACCESS_KEY = os.environ.get("AWS_SECRET_ACCESS_KEY")
AWS_STORAGE_BUCKET_NAME = "trydjango"
AWS_S3_ENDPOINT_URL = "https://nyc3.digitaloceanspaces.com"
AWS_S3_OBJECT_PARAMETERS = {
    "CacheControl": "max-age=86400",
     "ACL": "public-read"
}
AWS_LOCATION = "https://trydjango.nyc3.digitaloceanspaces.com"
DEFAULT_FILE_STORAGE = "trydjango.cdn.backends.MediaRootS3BotoStorage"
STATICFILES_STORAGE = 'trydjango.cdn.backends.StaticRootS3BotoStorage'
```

`trydjango/cdn/backends.py`
```python
from storages.backends.s3boto3 import S3Boto3Storage

class StaticRootS3BotoStorage(S3Boto3Storage):
    location = "static"

class MediaRootS3BotoStorage(S3Boto3Storage):
    location = "media"
```

### 7. Update `settings.py`

Add `"storages"` to `INSTALLED_APPS` like:
```python
INSTALLED_APPS = [
    # ...
    "storages"
]
```

Import the `cdn/conf.py` file

```python
STATIC_ROOT = STATIC_ROOT = BASE_DIR / "staticfiles-cdn" # dev example

from .cdn.conf import *  # noqa
```

### 8. Run Collect Static

```
python manage.py collectstatic --noinput
```
Verify your files have been uploaded into DigitalOcean Spaces.

Using `--noinput` is so you don't have to confirm you *may* override the files that are already in DO Spaces. This command is also ideal so you can setup [Django for Github Actions](https://www.codingforentrepreneurs.com/blog/django-github-actions) as well.