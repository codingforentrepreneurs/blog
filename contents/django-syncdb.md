---
title: Syncdb is ... gone?
slug: django-syncdb

publish_timestamp: Oct. 11, 2016
url: https://www.codingforentrepreneurs.com/blog/django-syncdb/

---

Yup `python manage.py syncdb` is no more. Here's the simple updates:

### Old commands:

`python manage.py syncdb`

`python manage.py createsuperuser` This creates a super user like `syncdb` does if the database is empty


### New commands:

`python manage.py makemigrations`

`python manage.py migrate`

`python manage.py createsuperuser` 


Read more about these commands on the [Django 1.10 docs](https://docs.djangoproject.com/en/1.10/ref/django-admin/#django-admin-and-manage-py)

We recommend watching our latest [Try Django](http://joincfe.com/youtube) series to see it in action.