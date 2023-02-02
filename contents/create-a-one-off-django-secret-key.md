---
title: Create a One-Off Django Secret Key
slug: create-a-one-off-django-secret-key

publish_timestamp: Feb. 1, 2023
url: https://www.codingforentrepreneurs.com/blog/create-a-one-off-django-secret-key/

---


I have to create one-off Django secret keys all the time. To do this, I use any of the following methods built-in Django methods for doing so. This is not the only way to create a valid and secure secret key for Django but it is what Django uses specifically.

This entire post assumes you have Django installed into a Python virtual environment. Bookmark this page as it will become more and more useful for you over time.


## From the Command Line
```bash
python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'
```
This will print out a new Django secret key every time you call it.

## Using the Django Managed Shell
In a virtual environment that has a Django project run:
```bash
python manage.py shell
```
In a virtual enviroment that *does not* have a Django project run:

```
django-admin shell
```

Now run the following:
```python

from django.core.management.utils import get_random_secret_key

print(get_random_secret_key())
```


## Base-64 Encoded Django Secret Key
If you need a _Base64 encoded_ version like for Kubernetes or Knative, use the following commands:

```bash
export SECRET_KEY=$(python -c "from django.core.management.utils import get_random_secret_key;print(get_random_secret_key())")
echo "Secret key is: $SECRET_KEY"
export SECRET_KEY_ENCODED=$(echo "$SECRET_KEY" | base64)
echo "Base64 Encoded Secret key is: $SECRET_KEY_ENCODED"
```
