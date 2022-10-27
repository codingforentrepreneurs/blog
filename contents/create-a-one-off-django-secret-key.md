---
title: Create a One-Off Django Secret Key
slug: create-a-one-off-django-secret-key

publish_timestamp: July 16, 2020
url: https://www.codingforentrepreneurs.com/blog/create-a-one-off-django-secret-key/

---


When you bring Django into Production, it's smart to put your `SECRET_KEY` into environment variables. What's better, is rotating your secret key from time to time. Here's a simple way to create a new secret key using a buil-in django method:


```bash
python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'
```


```python

from django.core.management.utils import get_random_secret_key

print(get_random_secret_key())
```
