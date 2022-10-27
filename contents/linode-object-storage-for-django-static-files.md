---
title: Linode Object Storage for Django Static Files
slug: linode-object-storage-for-django-static-files

publish_timestamp: Nov. 8, 2020
url: https://www.codingforentrepreneurs.com/blog/linode-object-storage-for-django-static-files/

---

Pushing Django into production takes many different steps to consider. Static files are among the most important steps.

Why is this?

Well, Django itself should not serve static files (ie images, css, javascript, videos, pdfs, etc) but rather should reference it from an production-grade static file server.

Technically, Django *can* serve static files but, just as the Django team recommends, it's not efficient and is probably insecure.

Luckily for us, Linode provides a service that makes it very easy to serve static files.

In this post, we'll be doing the essential setup for your Django project to use Object Storage on Linode.

#### Technical Stack
- Django (v2.2+)
- Django Storages
- Python (v3.6+)

#### Service Stack
- Production Django Server: Anywhere (linode, heroku, gcp, aws, digitalocean,)
- Static Files Server: Linode.com's [Object Storage](https://www.linode.com/products/object-storage/)

#### Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/oAn7isP5Osg" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Guide


### 1. Setup an account on [Linode](https://www.linode.com/cfe)
Go [here](https://www.linode.com/lp/youtube-viewers/?ifso=cfe) for a new user $100 credit (affiliate link).

### 2. Login to [https://cloud.linode.com/](https://cloud.linode.com/)

### 3. Navigate to [Object Storage](https://cloud.linode.com/object-storage)

Activate Object Storage on your account if you haven't already. It costs $5/mo for about 250GB of storage. This is more than enough for the vast majority of projects. Plus with that credit above you probably will not be paying anything for a while. Read more [here](https://www.linode.com/products/object-storage/)


### 4. Create a bucket
Inside of [object storage](https://cloud.linode.com/object-storage), click the link for `Add a bucket`. Next enter `Label` and select a `region`. I have entered the following:

```
label: cfe2
region: Newark, NJ
```
After the bucket is created locate the bucket url. Mine is:
```
cfe2.us-east-1.linodeobjects.com
```

Let's reference these new items as:
```
BUCKET_NAME = 'cfe2'
BUCKET_URL = 'https://cfe2.us-east-1.linodeobjects.com'
BUCKET_REGION = 'us-east-1'

```
**Keep note of your `BUCKET_NAME`, `BUCKET_URL`, and `BUCKET_REGION`, as we'll use them in our Django project**

> If you've used AWS S3, you'll notice identical naming patterns on purpose. Linode has made it very easy to switch.

### 5. Create a Object Storage `Access Key`
Navigate to [object storage](https://cloud.linode.com/object-storage) and click on `Access Keys` then `Create an Access Key`.

For the label, name it similar to your BUCKET. In my case I have the following:

*Access Key*
```
Label='cfe2-staticfiles-key'
Limited Access = True
Read/Write to BUCKET_NAME cfe2
```
Click `Create`.

Now you should have your required access keys:

```
LINODE_BUCKET_ACCESS_KEY='GBN3RIQWFB4EAI8Y3SF7'
LINODE_BUCKET_SECRET_KEY='S7zT9e5OSDeY4vwBBBm1BEdT8gFidE3qKkPNvwNB'
```
> Keep these keys safe, they will give anyone (or anything) that has them 100% access to your bucket.


### 6. Create/Activate your Virtual Environment

> Need a virtual environment? Learn how to create one for [windows here](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows) [mac here](https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux), or [linux here](https://www.codingforentrepreneurs.com/blog/install-django-on-linux-ubuntu/).

**Open up `terminal` or `PowerShell`**
```
cd path/to/your/django/project/
```

**Create with `venv`**

```
python3 -m venv
```
> feel free to use `pipenv`, `virtualenv`, `poetry` or any other virtual environment manager.

**Activate virtual enviroment**

*macOS / Linux*
```
source bin/activate
```

*Windows*
```
.\Scripts\Activate
```



#### 7. Install requirements

```
(venv) $ pip install django django-storages boto boto3 
```

#### 8. Setup base a Django Project

```
(venv) $ django-admin startproject cfehome .
(venv) $ python manage.py migrate
(venv) $ pip freeze > requirements.txt
```

Now verify files created include:
```
(venv) $ ls
cfehome     db.sqlite3      manage.py
pyvenv.cfg  requirements.txt
```


#### 9. Setup `django-storages`
Update your `settings.py` to the following: Notice my project name is `cfehome` so my settings are located in `cfehome/settings.py` 

We'll be using almost the same settings we would for AWS because `django-storages` does not currently have settings for Linode. 

Here are the settings:

```python
# cfehome/settings.py
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
STATICFILES_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'

LINODE_BUCKET=os.environ.get('LINODE_BUCKET', 'cfe')
LINODE_BUCKET_REGION=os.environ.get('LINODE_BUCKET_REGION',  'us-east-1')
LINODE_BUCKET_ACCESS_KEY=os.environ.get('LINODE_BUCKET_ACCESS_KEY', 'GBN3RIQWFB4EAI8Y3SF7') 
LINODE_BUCKET_SECRET_KEY=os.environ.get('LINODE_BUCKET_SECRET_KEY', 'S7zT9e5OSDeY4vwBBBm1BEdT8gFidE3qKkPNvwNB') 


AWS_S3_ENDPOINT_URL=f'https://{LINODE_BUCKET_REGION}.linodeobjects.com'
AWS_ACCESS_KEY_ID=LINODE_BUCKET_ACCESS_KEY
AWS_SECRET_ACCESS_KEY=LINODE_BUCKET_SECRET_KEY
AWS_S3_REGION_NAME=LINODE_BUCKET_REGION
AWS_S3_USE_SSL=True
AWS_STORAGE_BUCKET_NAME=LINODE_BUCKET
```


#### 10. Run `collectstatic`
Now run:

```
python manage.py collectstatic --noinput
```
From here on out, your Django project is using a production-grade static file server. Feel free to use `FileField`, `ImageField` or any other field that requires uploading as well.

Congrats! You are now setup with Linode Object storage. Easy enough huh? There are definitely more advanced options we can do here but `django-storages` and Linode make it easy to get up and running fast and efficiently.