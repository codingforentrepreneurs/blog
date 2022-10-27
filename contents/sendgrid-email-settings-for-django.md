---
title: Sendgrid Email settings for Django
slug: sendgrid-email-settings-for-django

publish_timestamp: Jan. 18, 2021
url: https://www.codingforentrepreneurs.com/blog/sendgrid-email-settings-for-django/

---


*[Sendgrid](https://sendgrid.com/)* is a **production-ready** transactional email service you can use in your business. With transactional email services you can:
- *White label*: your email address (hi@yourdomain.com)
- *Spam-less*: Send a LOT of email without being spammed automatically which is likely to happen if you don't use a transactional email service.
- *Reports & Clicks*: Get detailed reporting built-in to the emails. Track clicks, open rates, bounces.
- Read more on their FAQ [here](https://sendgrid.com/pricing/)

### Is transactional email the same as marketing email? No. Why? Read about it [here](https://kirr.co/4dsjzw).

Setup is super easy and is almost the same as [this guide](https://www.codingforentrepreneurs.com/blog/configure-email-in-django/) with one key difference: you use an API key as your password and `apiuser` as your username (not matter which account). 


### DJANGO_SETTINGS_MODULE (aka `settings.py`)
```
import os

EMAIL_HOST = 'smtp.sendgrid.net'
EMAIL_HOST_USER = 'apikey' # always use this
EMAIL_HOST_PASSWORD = os.environ.get('SENDGRID_API_KEY') # sendgrid.com/settings/api_keys
EMAIL_PORT = 587
EMAIL_USE_TLS = True
```

**Ensure the SMTP is activated in your account*


### To get your sendgrid API key:

1. Sign up for an account on sendgrid.com
2. Navigate to settings
3. Click create `API Keys`
4. Add your `api key` to your environment variables

> How do I load/use environment variables locally & in production in Django projects? Check out [the django-dotenv package](https://github.com/jpadilla/django-dotenv).

### Send a test email on Django:

Ensure in `settings.py` you have:
```
ADMINS = (
    ('You', 'you@email.com'),
)
MANAGERS = ADMINS
```

Now you can call the cli command:
```
python manage.py sendtestemail --admins
```
> Use `--managers` instead of `--admins` if you want to send to your list of `MANAGERS` instead of `ADMINS` in your `settings.py` configuration.
