---
title: Use Gmail for Email in Django
slug: use-gmail-for-email-in-django

publish_timestamp: June 8, 2017
url: https://www.codingforentrepreneurs.com/blog/use-gmail-for-email-in-django/

---


This is a simple way to setup gmail as your primary email service in Django. A general [configure your email](https://www.codingforentrepreneurs.com/blog/configure-email-in-django/) overview is [here](https://www.codingforentrepreneurs.com/blog/configure-email-in-django/).

> **Gmail** *is not recommended* for a production project (live web application) because gmail is not a transactional email service; gmail is not made web application use and, if abused, could cause you to be banned from gmail. In any case, it's still very useful to test on gmail until you move to a production-ready email service like Sendgrid or Postmark. Full disclosure: we used gmail as our production email service for many months prior to switching to sendgrid. 

### DJANGO_SETTINGS_MODULE (aka `settings.py`)
```
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = 'youremail@gmail.com' 
EMAIL_HOST_PASSWORD = 'yourpassword'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
```
**Ensure the SMTP is activated in your account*

### Allow less secure apps
[https://myaccount.google.com/lesssecureapps?pli=1](https://myaccount.google.com/lesssecureapps?pli=1)

Gmail has a TON of security built in. Allowing less secure apps makes it easier for your server via Django to login to gmail.

### Disable Captcha
[https://accounts.google.com/displayunlockcaptcha](https://accounts.google.com/displayunlockcaptcha)

In some cases, you may have to disable captcha when using Gmail with Django. Another security feature of Gmail.
